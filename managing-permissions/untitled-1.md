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

This lock has some similarity to the previous "CKB Lock", but there are a few changes. This code takes both an amount of `ckb_required` and a `count_required` as arguments, and will only unlock if there are at least `count_required` output cells that have a capacity of exactly `ckb_required`. We will call this the CKB Output Lock going forward.

On lines 3 to 5, we load the `args` data, then we extract the two values. We are using 64-bit integer values, so we know the length of the data we need is always 8 bytes for each value. These values will not vary in size, so they can safely be packed next to each other in the `args` field.

On lines 7 to 22, the code loads every output cell and checks the capacity. If it finds an input cell with a capacity matching `ckb_required`, then it increases the `matches_found` value. If match count is equal or greater to the `count_required` value then it unlocks. If the match count is below the `count_required` threshold, then an error will be returned.

### Usage in Lumos

Open the `index.js` file from the `Using-Multiple-Lock-Args-Example` directory and scroll down to the `main()` function. Our code has the usual four sections.

![](../.gitbook/assets/example-flow.png)

### Deploying the CKB Lock Binary

The process begins with deploying the lock script binary that contains our new lock code. This code is contained in `deployCkbOutputLockBinary()`. Feel free to go over it, but we're not going to go through it here since it is nearly identical to the previous examples.

### Creating the CKB Lock Cells

Next, we will look at the relevant parts of the `createCellsWithCkbOutputLock()` function.

```javascript
// Create cells using the CKB Lock lock.
const outputCapacity1 = ckbytesToShannons(500n);
const ckbLockAmount1 = intToU64LeHexBytes(ckbytesToShannons(500n));
const lockScript1 =
{
	code_hash: dataFileHash1,
	hash_type: "data",
	args: ckbLockAmount1
};
const output1 = {cell_output: {capacity: intToHex(outputCapacity1), lock: lockScript1, type: null}, data: "0x"};
transaction = transaction.update("outputs", (i)=>i.concat([output1, output1]));
```

This is the code that creates the cells using the CKB Lock. On line 2, you will see that we are creating cells with a capacity of exactly 500 CKBytes.

On line 3, we specify the number of CKBytes that must be present on any input to unlock the cell. We are specifying 500 CKBytes as a 64-bit little-endian value as hex bytes. This specific binary format is used because it is what is expected by CKB Lock. This is inserted into the lock script on line 8.

On line 6, we specify the data hash of the CKB Lock for the `code_hash`. On line 8, we put the value created on line 3.

If you look closely at line 11, you will notice that we are adding `output1` to the transaction two times, therefore creating two cells with 500 CKBytes each.

Our resulting transaction should look similar to this.

![](../.gitbook/assets/create-transaction-structure%20%281%29.png)

### Consuming the CKB Lock Cells

Next, we will look at the relevant parts of the `consumeCellsWithCkbOutputLock()` function.

```javascript
// Add the CKB Lock cells to the transaction. 
const capacityRequired = ckbytesToShannons(1_000n);
const ckbLockAmount1 = intToU64LeHexBytes(ckbytesToShannons(500n));
const lockScript1 =
{
	code_hash: dataFileHash1,
	hash_type: "data",
	args: ckbLockAmount1
};
const collectedCells = await collectCapacity(indexer, lockScript1, capacityRequired);
transaction = transaction.update("inputs", (i)=>i.concat(collectedCells.inputCells));
```

Here we add the CKB Lock cells to the transaction. We are using the `collectCapacity()` library function to locate the cells, and specifying the `capacityRequired` as 1,000 CKBytes. This should pull in the two cells we just created.

Normally when cell collection is performed you cannot be assured of the exact capacity value of the cells that are returned. In this example, we specifically created two cells each with 500 CKBytes each. We knew ahead of time that these would be the only two cells that exist with the CKB Lock so we can skip some of the extra code to verify. 

```javascript
	// Add in the witness placeholders.
	// transaction = addDefaultWitnessPlaceholders(transaction);

	// Sign the transaction.
	// const signedTx = signTransaction(transaction, privateKey1);
	const signedTx = sealTransaction(transaction, []);
```

Just like our previous example, we are skipping the placeholders and signing because only the CKB Lock cells were used as inputs and the CKB Lock does not check signatures.

Our resulting transaction will look like this.

![](../.gitbook/assets/consume-transaction-structure%20%281%29.png)

In the lock script `args` of our CKB Lock cells we specified 500 CKBytes and each of our cells were also created with exactly 500 CKBytes. This makes it very easy to form this transaction since they will always unlock.

What if we had created our cells and specified 250 CKBytes in the lock script `args`? Then the transaction would not go through because the CKB Lock cells would require at least one input cell with exactly 250 CKBytes. To complete the transaction, another input cell would have to be inserted.

![](../.gitbook/assets/consume-transaction-structure-2.png)

This transaction would complete because `address1` provided a cell with the correct capacity to unlock the CKB Lock cells.

