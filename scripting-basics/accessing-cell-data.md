# Accessing Cell Data

Inspecting the data within a cell is another common operation for evaluating the validity of a transaction. Next, we will create a cell that only allows a limited amount of data to be stored within it by using a type script.

The type script we use will read the data that the cell is being created with and validate the size of data as being 10 bytes or less. Trying to create a cell containing more than 10 bytes of data will result in the transaction being rejected. We will call this the "Data10" type script.

![](../.gitbook/assets/valid-invalid-transaction.png)

On the top left of the image is a transaction where the Data10 type script is used. The data area of the cell contains a string that is 10 bytes or less. If this cell were created in a transaction, meaning it was added as an output, the type script would execute without error and the transaction process successfully.

On the bottom left is a similar transaction using the Data10 type script, but the data area contains a string much larger than 10 bytes. If this cell were put into a transaction as an output, the type script would execute and return an error. This transaction would be rejected.

### Script Logic

Next, we will look at the logic and code that would be used to create this type script.

Let's take a look at it in pseudo-code first to understand the logic.

```javascript
function main()
{
    current_script = load_current_script();

    outputs = load_outputs();
    for cell in outputs
    {
        if(cell.type == current_script)
        {
            if(cell.data.length() > 10)
            {
                return 1;
            }
        }
    }

    return 0;
}
```

On line 3, we load the currently executing script. This is needed for comparison later on. 

On line 5, we load the outputs. This will return all of the outputs in the current transaction we are validating.

On line 6, we begin iterating through each output cell.

On line 8, we check if the output cell's type script matches the currently executing script. We do this because we are only concerned with cells that use the same type script, but there could be other cells in the transaction. This is part of what is known as the the minimal concern pattern, which we will cover in more depth later. What's important to know now is that our script is only validating cells that have the Data10 type script.

On lines 10 to 13, we check the length of the data field. If it has data larger than 10 bytes, an error is returned.

On line 17, we return successfully after no errors are found.

Our code only checks the outputs, because that is when the cell is created. When the cell is used as an input, we don't need to check again. This is because we already checked when the cell was created, and cells are immutable once created. However, it would also be acceptable to validate all the Data10 cells in the inputs in addition to the outputs. This would consume slightly more cycles, but it is acceptable as a security precaution.

Now let's look at the real version of the Data10 type script, written in Rust. This is located in the `entry.rs` file within the directory `developer-training-course-script-examples/contracts/data10/src`.

```rust
// Import from `core` instead of from `std` since we are in no-std mode.
use core::result::Result;

// Import CKB syscalls and structures.
// https://nervosnetwork.github.io/ckb-std/riscv64imac-unknown-none-elf/doc/ckb_std/index.html
use ckb_std::ckb_constants::Source;
use ckb_std::ckb_types::prelude::*;
use ckb_std::high_level::{load_cell, load_cell_data, load_script, QueryIter};

// Import our local error codes.
use crate::error::Error;

// Constants.
const MAX_DATA_SIZE: usize = 10;

// Main entry point.
pub fn main() -> Result<(), Error>
{
    // Load the current script.
    let script = load_script()?;
    
    // Load each cell from the outputs.
    let mut i = 0;
    for cell in QueryIter::new(load_cell, Source::Output)
    {
        // Check if there is a type script, and skip to the next cell if there is not.
        let cell_type_script = &cell.type_();
        if cell_type_script.is_none()
        {
            continue;
        }
        
        // Convert the scripts to bytes and check if they are the same.
        let cell_type_script = cell_type_script.to_opt().unwrap();
        if *script.as_bytes() == *cell_type_script.as_bytes()
        {
            // Load the cell's data.
            let data = load_cell_data(i, Source::Output)?;
            
            // If the data is larger than our limit.
            if data.len() > MAX_DATA_SIZE
            {
                // Return a limit exceeded error.
                return Err(Error::DataLimitExceeded);
            }
        }
        
        // Increment the index to process the next cell.
        i += 1;
    }
    
    Ok(())
}

```

Lines 1 to 11 are all imports of dependencies.

* The `core` library is an alternative to the Rust standard library that has some basic structures and types that work in `no_std` mode.
* The `ckb_std` library is the standard library used for developing Nervos scripts in Rust.
* Line 11 imports the custom error codes we have created for our script.

On line 14, we set a constant for the maximum amount of data, 10 bytes.

On line 20, we load the currently executing script. 

Lines 13 to 25 contain the main logic for our type script. The Rust syntax is a little more complex than our pseudo-code, but code flow is very similar, and the length of the code isn't much longer.

 On line 16, we use the `load_cell_data()` function to load cell data from the `GroupOutput` source. The `load_cell_data()` function can be used to load individual cells, but when combined with `QueryIter()` it can be used as a Rust `Iterator`, allowing us to cycle through all cells more easily.

On line 18, we check the length of the data. If the data is longer than 10, we return the error `DataLimitExceeded`.

On line 24, if no errors were detected, we return success.

