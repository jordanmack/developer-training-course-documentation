# Conditional Unlocking

When the lock script executes it has access to any values that are provided in the transaction. This includes the inputs, outputs, and cell deps. The lock script can use any of these resources as part of its criteria to determine if the cell should unlock or not.

Let's look at another example lock script in pseudo-code. This one will examine the transaction, and only unlock if the total capacity in all of the inputs is exactly 500 CKBytes.

```rust
fn main() -> i8
{
    let mut total_capacity = 0;
    
    let input_cells = load_input_cells();
    for cell in cells
    {
        total_capacity += cell.capacity;
    }

    if total_capacity == 50_000_000_000 // 500 CKBytes
    {
        return 0;
    }
    else
    {
        return 1;
    }
}
```

This code should be reasonably easy to understand. The input cells are loaded from the transaction, and then the capacity of each input cell is tallied. If the total input capacity is exactly 500 CKBytes, then the lock script will return with 0, indicating success.

This lock script is conceptually different than the default lock script because it shows how a script can expand its scope of concern. The code is examining all the input cells unconditionally. It doesn't matter if the input cells have a matching lock script, or are even owned by different people. All the code cares about is the input capacity amount.

This code is insecure and unsafe to use outside of a test environment, but it is a good example to demonstrate how funds can be unlocked with smart contract-like conditions instead of signatures.

### Usage in Lumos

Open the `index.js` file from the `Conditional-Unlocking-Example` directory and scroll down to the `main()` function. Just like the code from the previous topic, this code has four main sections.

![](../.gitbook/assets/example-flow.png)

This should look familiar because it is the same basic process. All that is changing is the lock script in use and a few details in the transaction structure which we'll explain.

### Deploying the CKB 500 Binary

The process begins with deploying the lock script binary that contains our conditional code which only unlocks when the input capacity is exactly 500 CKBytes. We will call this the "CKB 500" lock going forward. This code is contained in `deployCkb500Binary()`. Feel free to go over it if you need to, but we're not going to go through it here since it is nearly identical to the previous examples.

### Creating the CKB 500 Cells

Next, we will look at the relevant parts of the `createCellsWithCkb500Lock()` function.

```javascript
	// Create cells using the CKB 500 lock.
	const outputCapacity1 = ckbytesToShannons(250n);
	const lockScript1 =
	{
		code_hash: dataFileHash1,
		hash_type: "data",
		args: "0x"
	}
	const output1 = {cell_output: {capacity: intToHex(outputCapacity1), lock: lockScript1, type: null}, data: "0x"};
	transaction = transaction.update("outputs", (i)=>i.concat([output1, output1]));
```

This is the code that creates the cells using the CKB 500 lock. On line 2 you will see that we are creating cells with a capacity of exactly 250 CKBytes. We chose that number because it will be easy to create a transaction with exactly 500 CKBytes of input capacity.

On lines 5-7 we specify the data hash of the CKB 500 lock for the `code_hash` and once again `args` is empty because it isn't used. The CKB 500 lock only cares about input capacity, so it doesn't read the `args` value at all. 

If you look closely at line 10, you will notice that we are adding `output1` to the transaction two times, therefore creating two cells with 250 CKBytes each.

### Consuming the CKB 500 Cells

Next, we will look at the relevant parts of the `consumeCellsWithCkb500Lock()` function.

```javascript
	// Add the cell dep for the lock script.
	const cellDep = {dep_type: "code", out_point: ckb500CodeOutPoint};
	transaction = transaction.update("cellDeps", (cellDeps)=>cellDeps.push(cellDep));
```

Just like the previous example, we have to add our CKB 500 code as a cell dep to the transaction. It is only added to the consume function because this is the only place where the CKB 500 lock script is executing.

```javascript
	// Add the CKB 500 cells to the transaction. 
	const capacityRequired = ckbytesToShannons(500n);
	const lockScript1 =
	{
		code_hash: dataFileHash1,
		hash_type: "data",
		args: "0x"
	};
	const collectedCells = await collectCapacity(indexer, lockScript1, capacityRequired);
	transaction = transaction.update("inputs", (i)=>i.concat(collectedCells.inputCells));
```

Next, we add the CKB 500 cells to the transaction. However, unlike the previous example, we are not directly adding the outpoint. Instead, we are using the `collectCapacity()` library function to locate the cells. By specifying the lock script we need, we can search only for cells that were created with the CKB 500 lock.

