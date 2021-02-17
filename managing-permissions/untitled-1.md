# Using Multiple Lock Args

Each lock script has a single `args` field. In the last example, we populated that field with a single value, but it is very common for scripts to require multiple values. The lock that is used determines the data and formatting of the `args` field, and this includes how multiple values are included. 

An array or table of values can be serialized using the Molecule library. This works well, but the serialization can add overhead to both the computational cycles required and the state storage required. For that reason, it is more common to encode values directly in binary and pack them next to each other without any serialization library.

Let's take a look at a lock that requires multiple `args` values in pseudo-code.

```javascript
function main()
{
    lock_args = load_lock_args();
    ckb_required = int_from_le_bytes(lock_args[0..8]);
    count_required = int_from_le_bytes(lock_args[8..16]);

    matches_found = 0;
    output_cells = load_output_cells();
    for(cell in output_cells)
    {
        if(cell.capacity == ckb_required)
        {
            matches_found = matches_found + 1;
            
            if(matches_found >= count_required)
            {
                return 0;
            }
        }
    }
    
    return 1;
}
```

This lock has some similarities to the previous "CKB Lock", but there are a few changes. This code takes both an amount of `ckb_required` and a `count_required` as arguments, and will only unlock if there are at least `count_required` **output cells** that have a capacity of exactly `ckb_required`. We will call this the "CKB Output Lock" going forward.

On lines 3 to 5, we load the `args` data, then we extract the two values. We are using 64-bit integer values, so we know the length of the data we need is always 8 bytes for each value. These values will not vary in size, so they can safely be packed next to each other in the `args` field.

On lines 7 to 22, the code loads every output cell and checks the capacity. If it finds an input cell with a capacity matching `ckb_required`, then it increases the `matches_found` value. If match count is equal or greater to the `count_required` value then it unlocks. If the match count is below the `count_required` threshold, then an error will be returned.

### Usage in Lumos

Open the `index.js` file from the `Using-Multiple-Lock-Args-Example` directory and scroll down to the `main()` function. Our code has the usual four sections.

![](../.gitbook/assets/example-flow.png)

### Deploying the CKB Output Lock Binary

The process begins with deploying the lock script binary that contains our new lock code. This code is contained in `deployCkbOutputLockBinary()`. Feel free to go over it, but we're not going to go through it here since it is nearly identical to the previous examples.

### Creating the CKB Output Lock Cells

Next, we will look at the relevant parts of the `createCellsWithCkbOutputLock()` function.

```javascript
// Create cells using the CKB Output Lock lock.
const outputCapacity1 = ckbytesToShannons(500n);
const ckbOutputLockAmount1 = intToU64LeHexBytes(ckbytesToShannons(1_000n));
const ckbOutputLockCount1 = intToU64LeHexBytes(3);
const lockScript1 =
{
	code_hash: dataFileHash1,
	hash_type: "data",
	args: ckbOutputLockAmount1 + ckbOutputLockCount1.substr(2)
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

![](../.gitbook/assets/create-transaction-structure%20%281%29.png)

### Consuming the CKB Output Lock Cells

Next, we will look at the relevant parts of the `consumeCellsWithCkbOutputLock()` function.

```javascript
// Add the CKB Output Lock cells to the transaction. 
const capacityRequired1 = ckbytesToShannons(1_000n);
const ckbOutputLockAmount1 = intToU64LeHexBytes(ckbytesToShannons(1_000n));
const ckbOutputLockCount1 = intToU64LeHexBytes(3);
const lockScript1 =
{
	code_hash: dataFileHash1,
	hash_type: "data",
	args: ckbOutputLockAmount1 + ckbOutputLockCount1.substr(2)
};
const collectedCells = await collectCapacity(indexer, lockScript1, capacityRequired1);
transaction = transaction.update("inputs", (i)=>i.concat(collectedCells.inputCells));
```

Here we add the CKB Output Lock cells to the transaction. We are using the `collectCapacity()` library function to locate the cells. The lock script we specify is exactly the same as when we created the cells in the previous function, which ensures we get the exact same kind of cells, and specifying the `capacityRequired` as 1,000 CKBytes should pull in the two cells we just created which were each 500 CKBytes. 

```javascript
// Add an output cell with the specific amount required by the CKB Output Lock.
const outputCell = {cell_output: {capacity: intToHex(ckbytesToShannons(1_000n)), lock: addressToScript(address1), type: null}, data: "0x"};
transaction = transaction.update("outputs", (i)=>i.concat([outputCell, outputCell, outputCell]));
```

Here we add three output cells to the transaction, each with a capacity of 1,000 CKBytes. These cells are being added to the transaction to fulfill the requirements of the CKB Output Lock to unlock.

Our resulting transaction thus far looks like the image below. Do you see the problem? 

![](../.gitbook/assets/consume-transaction-structure%20%282%29.png)

The input capacity is 1,000 CKBytes, but the output capacity is 3,000 CKBytes. This transaction is invalid and would not confirm. To fix this we need to add more capacity.

```javascript
// Determine the capacity from input and output cells.
let inputCapacity = transaction.inputs.toArray().reduce((a, c)=>a+hexToInt(c.cell_output.capacity), 0n);
let outputCapacity = transaction.outputs.toArray().reduce((a, c)=>a+hexToInt(c.cell_output.capacity), 0n);

// Add capacity to the transaction.
const capacityRequired2 = outputCapacity - inputCapacity + ckbytesToShannons(61n) + txFee;
const {inputCells} = await collectCapacity(indexer, addressToScript(address1), capacityRequired2);
transaction = transaction.update("inputs", (i)=>i.concat(inputCells));
```

This code calculates the current input and output capacities, then performs cell collection to locate more cells for this. This code is nearly identical to some of the previous examples, so it should look familiar.

```javascript
// Recalculate new input capacity.
inputCapacity = transaction.inputs.toArray().reduce((a, c)=>a+hexToInt(c.cell_output.capacity), 0n);

// Create a change Cell for the remaining CKBytes.
const changeCapacity = intToHex(inputCapacity - outputCapacity - txFee);
const change = {cell_output: {capacity: changeCapacity, lock: addressToScript(address1), type: null}, data: "0x"};
transaction = transaction.update("outputs", (i)=>i.push(change));
```

This code recalculates the input capacity with the new input cells we just added, then it adds a change cell. Our resulting transaction will look like this.

![](../.gitbook/assets/consume-transaction-structure-2.png)

This transaction would complete because `address1` provided additional cells with the required capacity to complete the transaction.

```javascript
// Add in the witness placeholders.
transaction = addDefaultWitnessPlaceholders(transaction);

// Sign the transaction.
const signedTx = signTransaction(transaction, privateKey1);
```

Lastly, we add the necessary witness placeholders and sign the transaction. In the last topic, we didn't need to add signatures because the CKB Lock does not check them. The CKB Output Lock also does not check signatures. Do you know why we need to add signatures to this transaction?

The reason is that we added more input cells owned by `address1` because we need additional capacity to complete the transaction. These cells use the default lock script, and therefore always require a signature.

