# Using the Witness

In Nervos, the witness is a data structure that is the part of the transaction that is designated to hold signature data \(also known as witness data\). This is similar to the [witness data structure in Bitcoin](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki). Nervos extends the usage of the witness beyond signatures, to include any data which is needed by the transaction to confirm.

The witness is similar to the `args` field of a script, but it is distinctly different in several ways.

* The scope of the witness is a transaction instead of a script on a single cell.
* The data contained within the witness is still part of the blockchain, but it is not part of the state, and therefore doesn't require state rent.
* The witness is the part of the transaction that specifically takes into account the design considerations required for signatures.

The most common usage for the witness is to hold the signatures required by the transaction, but can also be used for more advanced functionality which we will cover in later lessons. In the most simple sense, the witness is like an `args` field for the transaction.

### The Structure of the Witness

The witness is a top-level part of a transaction, just like input, outputs, and cell deps. Similarly, the witness is also segmented the same way, in an array-like structure. This allows the data in the witness to be matched with inputs and outputs more easily.

Below is a line of code that has been used frequently in our examples. 

```javascript
// Add in the witness placeholders.
transaction = addDefaultWitnessPlaceholders(transaction);
```

This code adds the witness placeholders for the default lock to the transaction. We've used this code many times before, but we never went into detail about what it actually does.

The witness is an array structure that can be populated with any data, but there is one general convention that should be followed for compatibility with the default lock script and with scripts in general: For every unique lock script, the witness should contain the data required by the lock, at the same index. The image below will help illustrate this.

![](../.gitbook/assets/witness-indexes.png)

In this transaction, Alice and Bob are sending CKBytes to Charlie. Alice has provided input cells at indexes 0 and 1. Since index 0 is the first occurrence of Alice's lock in the inputs, she must add her signature at index 0 of the witness. Index 1 also uses Alice's lock, but it's not the first occurrence, so no data is needed. Bob has input cells at indexes 2 and 3. Index 2 is the first occurrence of Bob's lock in the inputs, so he must add his signature at index 2 of the witness. However, Bob cannot add his signature at index 2 unless there is something at index 1. To align the indexes properly, an empty value is added to the witness at index 1. An empty value could also be added to the witness at index 3, but this is optional.

Now that we understand the basic witness structure conventions, let's take a look at what `addDefaultWitnessPlaceholders()` is doing under the hood.

```javascript
/**
 * Adds witness placeholders to the transaction for the default lock.
 * 
 * This function adds zero-filled placeholders for all cells using the default
 * lock and empty placeholders for all other cells. If a cell is not using
 * the default lock, the placeholder may need to be altered after this function
 * is run. This function can only be used on an empty witnesses structure.
 * 
 * @param {Object} transaction An instance of a Lumos `TransactionSkelton`.
 * 
 * @return {Object} An instance of the transaction skeleton with the placeholders added.
 */
function addDefaultWitnessPlaceholders(transaction)
{
	if(transaction.witnesses.size !== 0)
		throw new Error("This function can only be used on an empty witnesses structure.");

	// Cycle through all inputs adding placeholders for unique locks, and empty witnesses in all other places.
	let uniqueLocks = new Set();
	for(const input of transaction.inputs)
	{
		let witness = "0x";

		const lockHash = computeScriptHash(input.cell_output.lock);
		if(!uniqueLocks.has(lockHash))
		{
			uniqueLocks.add(lockHash);

			if(input.cell_output.lock.code_hash === DEFAULT_LOCK_HASH && input.cell_output.lock.hash_type === "type")
				witness = "0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000";
		}

		witness = new Reader(core.SerializeWitnessArgs(normalizers.NormalizeWitnessArgs({lock: witness}))).serializeJson();
		transaction = transaction.update("witnesses", (w)=>w.push(witness));
	}

	return transaction;
}
```

This library function adds placeholder values to the witness specifically for the default lock script. At the first occurrence of each unique lock script, it adds a zero-filled placeholder. At every other index, it adds an empty value.

On line 21 you see the empty value, which is just a plain hex string. On line 29 you see the zero-filled placeholder. It is exactly 65 bytes long, which is the length required by a Secp256k1 signature.

On lines 30 and 31, the `witness` value is added to another structure called `WitnessArgs`. This is a structure that divides the witness entry into three components: `lock`, `input_type`, and `output_type`. The `lock` is reserved for lock scripts, and that is what is relevant to us since we're dealing with a lock script. The other two are reserved for the inputs and outputs on a type script, which will be covered in a later lesson.

The placeholder length of 65 bytes and the usage of the `WitnessArgs` structure to encapsulate our witness entries are specific requirements of the default lock. As we mentioned earlier, the witness allows any kind of data to be placed in it, but the default lock requires specific formatting.

Placeholders are put into the witness instead of signatures because this is required to generate the signing message for the transaction in a predictable way, and the `WitnessArgs` structure is used to further organize our data in a more compatible way. The usage of placeholders and `WitnessArgs` will be more apparent once we look at how the default lock script works later on. For now, we will use the witness as a basic array.

### Using the Witness to Provide Proof Data

The witness is most commonly used to provide signatures, but it can be used to provide any form of data to the transaction. We could use the witness to provide proof data to a transaction that isn't a signature.

Let's take a look at a lock that uses a Blake2b hash to secure a cell, and unlocks when the preimage is provided. We will call this a "hash lock" going forward. The pseudo-code below describes this.

```javascript
function main()
{
    lock_args = load_lock_args();
    hash = lock_args[0..32];

    witnessGroup = load_witness_group();
    preimage = witnessGroup[0];

    if(hash == blake2b(preimage, 256, "ckb-default-hash")))
    {
        return 0;
    }
    
    return 1;
}
```

On lines 3 and 4, we load the hash from the lock script args. This is a 256-bit hash, which is 32 bytes of data.

On lines 6 and 7, we load the preimage from the witness. Earlier we said, "For every unique lock script, the witness should contain the data required by the lock, at the same index." This is fulfilling that requirement and loading the data we need from the expected witness index. We're using something called a "witness group" here, which we will describe below. 

A witness group is the corresponding witnesses for inputs that have the same lock script as the one that is currently executing. Let's look at the transaction image again to better understand what this means.

![](../.gitbook/assets/witness-indexes.png)

When the lock script for Alice executes, the input group will include input cell index 0 and 1. Input cells 2 and 3 would not be included because the details of their lock script are different. The witness group for Alice would then be witnesses 0 and 1 because this matches the indexes for the input group. When the lock script for Charlie executes, the input group would include the cells at index 2 and 3. The witness group for Charlie would include only the witness at index 2 because no extra empty data was provided.

On line 9 to 14, we hash our preimage as a Blake2b 256-bit hash with the personalization phrase `ckb-default-hash`, then check if it matches the hash in the `args`. If it is a match, we unlock the cell, otherwise, we return an error.

### Usage in Lumos

Open the `index.js` file from the `Using-the-Witness-Example` directory and scroll down to the `main()` function. Our code has the usual four sections, and we will skip over the initialization and code deploying sections since they are redundant from previous examples.

![](../.gitbook/assets/example-flow.png)

### Creating the Hash Lock Cells

Near the top of `index.js` you will see this code.

```javascript
// This is the preimage which will be used to lock our cells.  
const preimage = "0x4f70656e20536573616d65"; // "Open Sesame"
```

This is the preimage data that we will use to generate our hash. This must remain a secret since anyone who knows the preimage would be able to unlock the cell. 

Next, we will look at the relevant parts of the `createCellsWithHashLock()` function.

```javascript
// Create cells using the Hash Lock.
const outputCapacity1 = ckbytesToShannons(500n);
const lockScript1 =
{
	code_hash: dataFileHash1,
	hash_type: "data",
	args: ckbHash(preimage)
};
const output1 = {cell_output: {capacity: intToHex(outputCapacity1), lock: lockScript1, type: null}, data: "0x"};
transaction = transaction.update("outputs", (i)=>i.concat([output1, output1]));
```

This is the code that creates the cells using the CKB Output Lock. On line 2, you will see that we are creating cells with a capacity of exactly 500 CKBytes.

On line 3, we specify the capacity that must be present on any output to unlock the cell. We are specifying 500 CKBytes as a 64-bit little-endian value as hex bytes. This specific binary format is used because it is what is expected by CKB Output Lock.

On line 4, we specify the minimum number of output cells that must match the capacity on line 3. The value is three, which means the outputs must have three cells with exactly 1,000 CKBytes in order for this input cell to unlock. 

On line 7, we specify the data hash of the CKB Output Lock for the `code_hash`.

On line 8, we add our amount and count values as `args`. They are packed together side by side as a single value as a hex string. The reason the second value uses `.substr(2)` is to remove the `0x` prefix when the hex string is concatenated.

If you look closely at line 12, you will notice that we are adding `output1` to the transaction two times, therefore creating two cells with 500 CKBytes each.

Our resulting transaction should look similar to this.

