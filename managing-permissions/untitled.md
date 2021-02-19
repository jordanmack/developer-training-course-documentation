# Using Lock Args

We mentioned before that a lock script's `args` field can contain any data in any format. However, it is up to the developer to decide what data should be put into the `args` field, and how it is encoded. Let's take a look at a common lock script example.

```text
{
    "code_hash": "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",
    "hash_type": "type",
    "args": "0x988a9c3e74c09dab76c8e41d481a71f4d36d772f"
}
```

The `code_hash` and `hash_type` values define what code will execute. These values match those for the default lock on a local development chain, so we know the default lock is being used. This mean that the data in the`args` field needs to conform to the exact requirements of the default lock in order to function correctly.

The `args` field should always contain whatever data is required by the lock that is in use. The code above specifies the default lock, so the `args` field contains a hashed public key that indicates who owns it. We will cover all the details of the default lock later on, but first, let's explore a more simple lock that uses the `args` described in the pseudo-code below.

```javascript
function lockScript(args, input_cells)
{
    capacity_required = integer_from_binary(args);

    for(cell in input_cells)
    {
        if(cell.capacity == capacity_required)
        {
            return 0;
        }
    }
    
    return 1;
}
```

This lock is similar to the "CKB 500" example from the previous topic, but there are a few changes. This code will unlock only if there is at least one input cell in the transaction that has a capacity equal to a specific amount. That specific amount is contained within the lock's `args` field. We will call this the "Input Capacity Check Lock" going forward, or "ICC Lock" for short.

On line 3, the `args` data is converted from binary to an integer, and it contains the capacity amount required. The `args` data that we are reading is that which was added to the cell's lock script when it was created. We'll come back to this subject in a moment.

On lines 5 to 12, the code loads every input cell and checks the capacity. If it finds an input cell with a matching capacity, then it immediately unlocks, otherwise, it will return an error on line 14.

Let's look at the lifecycle of a cell with the ICC Lock is used in the image below.

![](../.gitbook/assets/lifecycle-explainer.png)

The transaction on the left creates a cell with the ICC Lock. In the lock script, the `args` field has a value of 500. We are just setting the value here, but it is not being enforced because lock scripts only execute on inputs, not outputs. Once this transaction confirms, the cell will be added to the blockchain and will be immutable. The `args` value is stored in the cell, and it cannot be changed until the cell is unlocked when it's used in a transaction.

The transaction on the right consumes the cell with the ICC Lock. The cell is being used as an input, so the ICC Lock would execute and read the `args` value just as our pseudo-code indicated above. The ICC Lock sees that the `args` value is 500, then it checks the inputs and finds a cell with a matching 500 CKByte capacity. The unlock is successful, and the transaction completes successfully.

### Usage in Lumos

Next, we will use the ICC Lock in a Lumos transaction example. Our code will deploy the lock, create some cells using the ICC Lock, then consume those cells that we just created.

Open the `index.js` file from the `Using-Lock-Args-Example` directory and scroll down to the `main()` function. Our code has the usual four sections.

![](../.gitbook/assets/example-flow.png)

### Deploying the CKB Lock Binary

The process begins with deploying the lock script binary that contains our new lock code. This code is contained in `deployCkbLockBinary()`. Feel free to go over it, but we're not going to go through it here since it is nearly identical to the previous examples.

### Creating the CKB Lock Cells

Next, we will look at the relevant parts of the `createCellsWithCkbLock()` function.

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

![](../.gitbook/assets/create-transaction-structure%20%282%29.png)

### Consuming the CKB Lock Cells

Next, we will look at the relevant parts of the `consumeCellsWithCkbLock()` function.

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

![](../.gitbook/assets/consume-transaction-structure-2%20%281%29.png)

This transaction would complete because `address1` provided a cell with the correct capacity to unlock the CKB Lock cells.



