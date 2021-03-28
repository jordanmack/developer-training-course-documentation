# Examining a Transaction

Every lock script and type script has the ability to evaluate the transaction that it is included in. One of the most common ways to do this is to inspect the cells that were included in the transaction. This is done by using syscalls \(system calls\) to load cells from the transaction. 

Let's create a type script that counts the number of input cells available, and succeeds only when that count is 3. We will call this the "IC3Type" for short. Here is the psuedo-code for this type script.

```javascript
function main()
{
    cells_required = 3;
    cell_count = 0;

    input_cells = load_input_cells();
    for(cell in input_cells)
        cell_count += 1;

    if(cell_count == cells_required)
        return 0;
    else
        return 1;
}
```

This should be fairly straightforward, but there are a few small details to point out.

On lines 6 to 8, we load the input cells and cycle through them, but we don't actually look at the content of the cells since we are just counting them. It may seem like it would be more efficient to just use `input_cells.lenth()` to get the number of cells that were returned, but we wanted this code to better represent the real process. In actuality, cells can only be loaded one at a time in a syscall, and we have to load them one by one to count them.

Here is the real code that uses the Capsule framework, and is written in Rust. This code is available in the Developer Training Course Scripts repo in the file `contracts/ic3type/src/entry.rs`.

```rust
// Import from `core` instead of from `std` since we are in no-std mode
use core::result::Result;

// Import CKB syscalls and structures.
// https://nervosnetwork.github.io/ckb-std/riscv64imac-unknown-none-elf/doc/ckb_std/index.html
use ckb_std::ckb_constants::Source;
use ckb_std::high_level::load_cell;
use ckb_std::error::SysError;

// Import local modules.
use crate::error::Error;

// Constants.
const CELLS_REQUIRED: usize = 3;

// Main entry point.
pub fn main() -> Result<(), Error>
{
    // Track the number of input cells that exist.
    let mut cell_count = 0;
    
    // Cycle through all the input cells to count them.
    let mut i = 0;
    loop
    {
        match load_cell(i, Source::Input)
        {
            Ok(_) => cell_count += 1,
            Err(SysError::IndexOutOfBound) => break,
            Err(e) => return Err(e.into())
        }
        i += 1;
    }
    
    // If our cell count matches the requirement then exit successfully.
    if cell_count == CELLS_REQUIRED
    {
        return Ok(());
    }
    
    // Return an error if the cell count is incorrect.
    Err(Error::Unauthorized)
}

```

The beginning of the file is all imports, so we will skip straight to line 14. This is the number of input cells required to unlock the cell. We hard-coded this to keep the code simple.

On lines 23 to 33, we load and count all the input cells. The [load\_cell\(\)](https://nervosnetwork.github.io/ckb-std/riscv64imac-unknown-none-elf/doc/ckb_std/high_level/fn.load_cell.html) function is the syscall we use to query for a single cell using an index and a source as arguments. The index value is the index of a cell in the transaction, and the source is where to load the cell from. For example, `load_cell(0, Source::Input)` loads the first input cell.

The `load_cell()` function returns a cell structure that we can examine, but in this example, we are just interested in counting the input cells. Even though we are only counting the cells, we still have to go through the process of loading each cell one at a time. On line 28, we successfully load a cell and count it.

On line 29, we handle an `IndexOutOfBound` syscall error. This is a special error that is received when we try to use `load_cell()` on an index that is higher than the number of cells that exist. It is normal to receive this error when iterating through all the available cells, because this error lets you know you have reached the end of what is available.

On lines 36 to 39, we check if the cell count matches the cells required, and return successfully if it matches.

On line 42, we return an error if the cell count didn't match. The error `Unauthorized` is a custom error code that is defined in the `contracts/ic3type/src/error.rs` file. Error codes are defined by the individual script, meaning you can define as many error codes as you need here.

### Usage in Lumos

Next, we will use the IC3Type script in a transaction using Lumos. We'll use a precompiled binary for this example to make things easy.

Open the `index.js` file from the `Using-Examining-a-Transaction-Example` folder. If you scroll down to the `main()` function, you will see that there four main sections. These are the same four sections as the previous Lumos example, and you will see this pattern often.

![](../.gitbook/assets/example-flow.png)

1. Initialize - In the first three lines of code in `main()`, we initialize the Lumos configuration, start the Lumos Indexer, and initialize the lab environment.
2. Deploy Code - The `deployCode()` function creates a cell with the contents of the RISC-V binary located in the file `./files/ic3type`. This is the ic3type script binary executable.
3. Create Cells - The `createCells()` function creates a cell that uses the ic3type script.
4. Consume Cell - The `consumeCells()` function consumes the cell with the ic3type script that we just created.

The initialization and deployment code is nearly identical to the previous examples, so we're not going to go over it again unless there is specifically something different we need to point out. Feel free to review that code on your own if you need a refresher.

### Creating a Cell with the Always Success Lock

Next, let's look at the `createCells()` function. This function generates and executes a transaction that will create a cell using the always success script code as a lock script. Once again, we'll skip straight to the relevant parts.

```javascript
// Create a cell using the always success lock.
const outputCapacity1 = ckbytesToShannons(41n);
const lockScript1 =
{
	code_hash: dataFileHash1,
	hash_type: "data",
	args: "0x"
};
const output1 = {cell_output: {capacity: intToHex(outputCapacity1), lock: lockScript1, type: null}, data: "0x"};
transaction = transaction.update("outputs", (i)=>i.push(output1));
```

There are a few interesting things about this code. Look at the value of the `outputCapacity1` variable. It's set to 41 CKBytes. You may be thinking, "isn't the minimum 61?" Yes, 61 CKBytes is the minimum for a standard cell using the default lock script, but we're not using the default lock script.

The `lockScript1` variable defines the lock script for the cell. The `code_hash` is being set to a Blake2b hash of the always success lock script binary. The `hash_type` is `data`, which means the `code_hash` value needs to match a Blake2b hash of the data in a cell containing the code that will be executed. Our `code_hash` value reflects this. You may have noticed that the `hash_type` of the default lock script is `type`. The meaning of this value is more complicated but usually means that the script is upgradable. We will cover that use case at a later time. Finally, we have the `args` value. Notice that it's empty. Let's compare it to the `args` of a live cell using the default lock script.

![](../.gitbook/assets/get-live-cell.png)

When you use the default lock script, the `args` field is always expected to have a hash of the public key. However, this specific requirement applies only to the default lock script. The `args` field can contain any data in any format. It is the script in use that dictates how the data in the `args` should be formatted, or if it is even needed at all.

The always success script code does not use `args` in any way, so it doesn't need to be included. This saves that 20 bytes of space, and is the reason our cell only needs 41 CKBytes instead of the normal 61 CKBytes.

Even though this saves a little bit of space, it isn't practical to use in a production environment. The always success lock is completely insecure, which is why we only use it for testing purposes.

The resulting generated transaction will look something like this.

![](../.gitbook/assets/create-transaction-structure%20%289%29.png)

### Consuming a Cell with the Always Success Lock

Now let's look at the relevant parts of the `consumeCells()` function. This function generates and executes a transaction that will consume the cells we just created that use the always success lock.

```javascript
// Add the cell dep for the lock script.
transaction = addDefaultCellDeps(transaction);
const cellDep = {dep_type: "code", out_point: alwaysSuccessCodeOutPoint};
transaction = transaction.update("cellDeps", (cellDeps)=>cellDeps.push(cellDep));
```

This code adds cell deps to our transaction skeleton. On line 2 you see the function `addDefaultCellDeps()`. If we look into the shared library, we will see this:

```javascript
function addDefaultCellDeps(transaction)
{
    return transaction.update("cellDeps", (cellDeps)=>cellDeps.push(locateCellDep({code_hash: DEFAULT_LOCK_HASH, hash_type: "type"})));
}
```

We can see that this function is adding a cell dep for the default lock hash, and it's getting it from the `locateCellDep()` function. The `locateCellDep()` function is part of Lumos, and it can be used to locate specific well-known cell deps for the default lock script, the multisig lock script, and the Nervos DAO. This function is getting this information from the `config.json` file in the working directory.

However, we will not be able to use the `locateCellDep` function with the always success binary we just loaded, because it is not well-known. Instead, we construct a cell dep object which we add to the cell deps in the transaction using this code:

```javascript
const cellDep = {dep_type: "code", out_point: alwaysSuccessCodeOutPoint};
transaction = transaction.update("cellDeps", (cellDeps)=>cellDeps.push(cellDep));
```

The `dep_type` can be either `code` or `dep_group`. The value of `code` indicates that the out point we specify is a code binary. The other possible value, `dep_group`, is used to specify multiple out points at once. We'll be covering how to use that in a future lesson.

If you look closely at the code in `createCells()` and `consumeCells()`, you will notice that we're only adding the always success lock as a cell dep in the consume function. The always success lock is referenced in the lock script of cells in both the create and consume functions, but we only need it to be referenced in the cell deps of the consume function is because that is the only time when it is executed.

A lock script executes when we need to check permissions to access a cell. We only need to do this when a cell is being used as an input, since this is the only time value is extracted from the cell. When creating an output, the value is coming from inputs that you have already proven you have permission to access. There is no reason you should have to prove ownership again, and therefore the lock script never executes on outputs, and we don't need to provide a cell dep to the always success binary since it isn't executing.

```javascript
// Add the always success cell to the transaction.
const input = await getLiveCell(nodeUrl, alwaysSuccessCellOutPoint);
transaction = transaction.update("inputs", (i)=>i.push(input));

// Add input capacity cells.
const capacityRequired = ckbytesToShannons(61n) + txFee;
const collectedCells = await collectCapacity(indexer, addressToScript(address1), capacityRequired);
transaction = transaction.update("inputs", (i)=>i.concat(collectedCells.inputCells));

// Determine the capacity from all input Cells.
const inputCapacity = transaction.inputs.toArray().reduce((a, c)=>a+hexToInt(c.cell_output.capacity), 0n);
const outputCapacity = transaction.outputs.toArray().reduce((a, c)=>a+hexToInt(c.cell_output.capacity), 0n);

// Create a change Cell for the remaining CKBytes.
const changeCapacity = intToHex(inputCapacity - outputCapacity - txFee);
let change = {cell_output: {capacity: changeCapacity, lock: addressToScript(address1), type: null}, data: "0x"};
transaction = transaction.update("outputs", (i)=>i.push(change));
```

This code is adding our always success cell to the transaction, adding more cells for capacity, then sending everything back to ourselves as a single change cell. Remember, the always success cell we created only has 41 CKBytes in it. This is below a 61 CKByte standard cell and doesn't account for the necessary transaction fee. 

Looking at the fourth block of code for the change cell, the `lock` is set to `addressToScript(address1)`. This means it is using the default lock script again, and that 61 CKBytes is the minimum required.

The reason that `capacityRequired` in the code above is set to 61 CKBytes + the tx fee is because we are anticipating an output of a single standard cell and a tx fee.

![](../.gitbook/assets/transaction-compare.png)

On the left is the transaction we are generating. We are consuming the always success cell and sending the value back to ourselves, so we only need one output.

On the right is what it would look like if we were sending to someone else. We would need more capacity since we would need an output to send to them, and a change cell. In that scenario, we would need at least 122 CKBytes \(+ tx fee\) since we are creating two output cells. We can reuse the 41 CKBytes on the consumed always success cell, meaning the absolute minimum capacity would need to collect is 81 CKBytes \(122 - 41\), plus the transaction fee.

```javascript
	// Sign the transaction.
	const signedTx = signTransaction(transaction, privateKey1);
```

This code looks standard and we've used it many times in the past, but it's important to point out why it's necessary for this transaction. The always success lock does not require any kind of signing in order to unlock. If it was the only input cell that existed, then we could skip this step. However, we had to add additional capacity from `address1`, and those cells use the default lock, which requires a standard signature in order to unlock.
