# Using Script Args

In an earlier lesson, we created the Data10 type script, which only allows cells to be created with a maximum of 10 bytes of data. In the script, we hard-coded the limit to 10 bytes. This works, but we have to create a new script every time we want to change the size limit. We can avoid this limitation by placing the limit in the script's args field so it can be changed easily.

Lock scripts and type scripts both take args similarly to how a program would take an argument on the command line. This allows configuration data to be passed to the script without having to modify the script. You've already used this several times with the default lock, which takes a hash of a public key as in the lock script args field to indicate who owns the cell and has permission to unlock it.

Next, we will look at a type script that uses the args to specify the data limit. We will call this the "DataCap" type script.

![](../.gitbook/assets/valid-invalid-transaction%20%283%29.png)

On the top left of the image is a transaction where the DataCap type script is used. In the type script args field is the value `10`, indicating a data limit of 10 bytes within the data field. The cell contains a string that is 10 bytes long. If this cell were created in a transaction, meaning it was added as an output, the type script would execute without error and the transaction would process successfully.

On the bottom left is a similar transaction using the DataCap type script, with the type script args set to a larger value of 20. The data area contains a string much larger than 20 bytes. If this cell were put into a transaction as an output, the type script would execute and return an error. This transaction would be rejected.

### Script Logic

Next, we will look at the logic and code that would be used to create this type script.

Let's take a look at it in pseudo-code first to understand the logic.

```javascript
function main()
{
    max_data_size = integer_from_binary(args);

    outputGroup = load_output_group();
    for cell in outputGroup
    {
        if(cell.data.length() > max_data_size)
        {
            return 1;
        }
    }

    return 0;
}
```

On line 3, we load the max data size limit from the args.

On line 5, we load the output group. When this is called from a type script, the output group only includes only the output cells that have the same type script. The outputs could contain many different cells, but this script is only concerned with those using the DataCap type script, which is why we use the output group instead of just the outputs.

On lines 6 to 12, we cycle through every cell in the output group, checking the data field of each one. If any of them have data larger than `max_data_size` bytes, an error is returned. We only check the outputs, because that is when the cell is created. When the cell is used as an input, we don't need to check again. This is because we already checked when the cell was created, and cells are immutable once created.

On line 14, we return successfully after no errors are found.

Now let's look at the real version of the type script, written in Rust. This is located in the `entry.rs` file within the directory `developer-training-course-script-examples/contracts/datacap/src`.

```rust
// Import from `core` instead of from `std` since we are in no-std mode.
use core::result::Result;

// Import CKB syscalls and structures.
// https://nervosnetwork.github.io/ckb-std/riscv64imac-unknown-none-elf/doc/ckb_std/index.html
use ckb_std::ckb_constants::Source;
use ckb_std::ckb_types::{bytes::Bytes, prelude::*};
use ckb_std::high_level::{load_cell_data, load_script, QueryIter};

// Import our local error codes.
use crate::error::Error;

// Main entry point.
pub fn main() -> Result<(), Error>
{
    // Load arguments from the current script.
    let script = load_script()?;
    let args: Bytes = script.args().unpack();
    
    // Verify that correct length of arguments was given.
    if args.len() != 8
    {
        return Err(Error::ArgsLen);
    }
    
    // Load the cell_data_limit from the script args.
    let mut buffer = [0u8; 8];
    buffer.copy_from_slice(&args[0..8]);
    let cell_data_limit = u64::from_le_bytes(buffer);
    
    // Load the cell data from each cell.
    for data in QueryIter::new(load_cell_data, Source::GroupOutput)
    {
        if data.len() as u64 > cell_data_limit
        {
            return Err(Error::DataLimitExceeded);
        }
    }
    
    Ok(())
}
```

Lines 1 to 11 are all imports of dependencies.

* The `core` library is an alternative to the Rust standard library that has some basic structures and types that work in `no_std` mode.
* The `ckb_std` library is the standard library used for developing Nervos scripts in Rust.
* Line 11 imports the custom error codes we have created for our script.

Lines 13 to 41 contain the main logic for our type script. The Rust syntax is a little more complex than our pseudo-code since we have to do more validation, but the code flow is very similar.

On line 17 and 18 we load the raw bytes from the script's `args`. On lines 21 to 24 verify that the data in the `args` is exactly 8 bytes. Our script expects a u64 value, which is always 8 bytes. If the `args` length is not the expected size, we return an `ArgsLen` error. The data we retrieved from the `args` is still raw data at this point.

On lines 27 to 29 we convert the data from the `args` to a u64 value. We don't need to do any kind of validation here because we know that the data is exactly 8 bytes, and this can always convert into an u64 successfully.

On line 28, look at how the bytes to copy are specified using the `0..8` range. We can add multiple values to the args field by packing them next to each other, then specifying the ranges to read them out again.

 On line 32, we use the `load_cell_data()` function to load cell data from the `GroupOutput` source. The `load_cell_data()` function can be used to load individual cells, but when combined with `QueryIter()` it can be used as a Rust `Iterator`, allowing us to cycle through all cells more easily.

On line 34, we check the length of the data. If the data is longer than `cell_data_limit`, we return the error `DataLimitExceeded`.

On line 40, if no errors were detected, we return success.

### Usage in Lumos

Next, we will use the DataCap type script in a Lumos example. Our code will deploy the lock, create some cells using the DataCap script, then consume those cells that we just created to reclaim that capacity.

The code we will be covering here is located in the `index.js` file in the `Using-Script-Args-Example` directory. Feel free to open the `index.js` file and follow along. This code example is fully functional, and you should feel free to modify and experiment with it. You can execute this code in a console by entering the directory and executing `node index.js`.

Starting with the `main()` function, you will see our code has the usual four sections.

![](https://gblobscdn.gitbook.com/assets%2F-MLuiCvogNfxQTk5TWAq%2F-MWXyed_PZWmr5dt-R_b%2F-MRhtF2hvu67w4rFgsc_%2FExample-Flow.png?alt=media&token=0e93ab2c-178c-4bd9-a758-2ad39ea92d54)

The initialization and deployment code is nearly identical to the previous examples, so we're not going to go over it here. Feel free to review that code on your own if you need a refresher.

### Creating Cells

Next, we will look at the relevant parts of the `createCells()` function. This function generates and executes a transaction that will create cells using the DataCap type script.

```javascript
// Create cells using the DataCap type script.
const messages = ["Hello World!", "Foo Bar", "1234567890"];
for(let message of messages)
{
    const outputCapacity1 = ckbytesToShannons(500n);
    const lockScript1 = addressToScript(address1);
    const dataCapSize1 = intToU64LeHexBytes(20);
    const typeScript1 =
    {
        code_hash: dataFileHash1,
        hash_type: "data",
        args: dataCapSize1
    };
    const data1 = stringToHex(message);
    const output1 = {cell_output: {capacity: intToHex(outputCapacity1), lock: lockScript1, type: typeScript1}, data: data1};
    transaction = transaction.update("outputs", (i)=>i.push(output1));
}
```

This is the code logic that creates the cells that use the DataCap type script. It uses the `messages` provided on line 2, then loops through them creating three cells with the different data.

On line 7 we convert our limit of 20 bytes as a 64-bit little-endian value as hex bytes. This specific binary format is used because it is what is expected by DataCap type script. This value is added to the type script on line 12. We could set different limits here for each cell, but that would make it more difficult to collect the cells later. We'll explain more later when we cover cell collection.

On lines 9 to 13, we define the type script for the cell. The syntax for this is the same as when we created lock scripts in the past, but it is added as the `type` instead of the `lock` when we generate the cell structure on line 15.

On line 14 we convert our message to a hex string, and then add it to the structure on line 15. 

The resulting transaction will look similar to this. We are creating three cells using the DataCap type script, and all are the same except for the data contained within. 

![](../.gitbook/assets/create-transaction-structure%20%2811%29.png)

### Consuming Cells

Next, we will look at the relevant parts of the `consumeCells()` function. This function generates and executes a transaction that will consume the cells we just created that use the DataCap type script.

```javascript
// Add the DataCap cells to the transaction. 
const lockScript1 = addressToScript(address1);
const dataCapSize1 = intToU64LeHexBytes(20);
const typeScript1 =
{
    code_hash: dataFileHash1,
    hash_type: "data",
    args: dataCapSize1
};
const query = {lock: lockScript1, type: typeScript1};
const cellCollector = new CellCollector(indexer, query);
for await (const cell of cellCollector.collect())
    transaction = transaction.update("inputs", (i)=>i.push(cell));
```

Here we add the cells with the DataCap type script to the transaction using the `CellCollector()` library function. To form our query to locate the cells we must specify the same lock script and type script that they were create with.

On line 3, we specify 20 bytes as the DataCap limit. This will locate all available DataCap cells because they were all created with a 20 byte value. As we mentioned earlier, we could have specified different values for each cell. However, it would then require multiple queries to find them.

On line 10, we specify the `lock` and `type`. We could also specify `data` here, but then we would have to use three different queries to locate our cells. We're more interested in using a single query to find all the cells.

On lines 12 and 13, we add all the cells found to the transaction. This will add the three cells we created, but if there were more cells it would continue to loop, adding them all.

The resulting transaction will look similar to this.

![](../.gitbook/assets/consume-transaction-structure%20%2810%29.png)

In the image, the three input DataCap cells are the same, except for the data contained within. Only one type script is pictured due to space considerations, but all three are the same.

A type script executes on both inputs and outputs, so the DataCap type script will execute here. It will query the output group, but it won't find anything. There is one output, but that doesn't have the same type script, so it will not be included when the output group is queried. Since there are no DataCap cells to validate, it will exit with success, allowing the cells to be consumed.

