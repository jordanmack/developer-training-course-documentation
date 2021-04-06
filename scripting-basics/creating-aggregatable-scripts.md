# Creating Aggregatable Scripts

One of the unique features of the Cell Model is that multiple cells can be combined into a single large transaction instead of creating separate small transactions. This process is called "aggregation". However, aggregation is only possible when scripts are programmed in a way flexible enough to allow it. In this lesson, we will demonstrate some of the basic principles of creating aggregatable scripts.  

### Minimal Concern Pattern

One of the most common ways to create an aggregatable script is the follow the minimal concern pattern.  Following this creates scripts that have composable cell logic that allows them to be combined in a single transaction safely without affecting other cells.

To incorporate the minimal concern pattern a script should:

1. Process all cells in the script group.
2. Allow any number of cells in a script group.
3. Ignore all cells that are not in the same script group.

In essence, a script should only check the minimal amount of information needed to ensure the validity of the cells it is concerned with.

### Script Logic

Next, we will show an example of an Aggregatable Counter type script. But first, we will start by reviewing the pseudo-code for the original Counter type script as a quick refresher.

```javascript
function main()
{
    group_input_count = load_input_group().length();
    group_output_count = load_output_group().length();

    if(group_input_count == 0)
        return 0;
    
    if(group_input_count != 1 || group_output_count != 1)
        return 1;

    input_value = integer_from_binary(load_input_group().get_first_cell().data);
    output_value = integer_from_binary(load_output_group().get_first_cell().data);

    if(input_value + 1 != output_value)
        return 1;

    return 0;
}
```

Now, let's take a look at what a transaction would look like if it contained multiple Counter cells.

![](../.gitbook/assets/counter-transaction-structure-2.png)

This transaction would not execute successfully because the Counter type script would give an error. The offending lines in the pseudo-code would be 9 and 10. The Counter type script was only designed to process exactly one input and one output.

Here is the pseudo-code for the Aggregatable Counter.

```javascript
function main()
{
    group_input_count = load_input_group().length();
    group_output_count = load_output_group().length();

    if(group_input_count == 0)
        return 0;
    
    if(group_input_count != group_output_count)
        return 1;

    for(i = 0; i < group_input_count; i++)
    {
        input_value = integer_from_binary(load_input_group_data(i));
        output_value = integer_from_binary(load_output_group_data(i));

        if(input_value + 1 == output_value)
            return 1;
    }

    return 0;
}
```

The code starts out the same as the regular counter. On lines 3 and 4, we count the number of group input cells and group output cells. On lines 6 and 7, we immediately succeed if there are no input cells which allows for the creation of new Counter cells.

On lines 9 and 10, we check that the counts match 1:1. This is necessary in order to match up the inputs with the outputs and locate matches, but we no longer restrict the transaction to having one input and one output.

On line 12, we loop through each cell index.

On lines 14 and 15, we load the cell data from the inputs and outputs at the current index, then we convert the data from binary to integer values.

On lines 17 and 18, we compare the input value and the output value to ensure the output value is exactly one higher. If this is not a match, we return an error.

On line 21, we return success if no errors have occurred.

This code can take any number of Counter cells. The requirements are that while updating there must be one input for every output and that the updated output value for an input value must be at the same index.

Here is the real version in Rust.

```rust
// Import from `core` instead of from `std` since we are in no-std mode.
use core::result::Result;

// Import CKB syscalls and structures
// https://nervosnetwork.github.io/ckb-std/riscv64imac-unknown-none-elf/doc/ckb_std/index.html
use ckb_std::ckb_constants::Source;
use ckb_std::high_level::{load_cell, load_cell_data, QueryIter};

// Import local modules.
use crate::error::Error;

// Main entry point.
pub fn main() -> Result<(), Error>
{
    // Count on the number of group input and groupt output cells.
    let group_input_count = QueryIter::new(load_cell, Source::GroupInput).count();
    let group_output_count = QueryIter::new(load_cell, Source::GroupOutput).count();
    
    // If there are no inputs, skip validation.
    if group_input_count == 0
    {
        return Ok(());
    }
    
    // If there isn't an exact 1:1 ratio, give an error.
    if group_input_count != group_output_count
    {
        return Err(Error::InvalidTransactionStructure);
    }
    
    // Loop through all the group input cell data.
    for i in 0..group_input_count
    {
        // Load the input and output data at the current index.
        let input_data = load_cell_data(i, Source::GroupInput)?;
        let output_data = load_cell_data(i, Source::GroupOutput)?;
        
        // Convert the input cell data into a u64 value.
        let mut buffer = [0u8; 8];
        buffer.copy_from_slice(&input_data[0..8]);
        let input_value = u64::from_le_bytes(buffer);
        
        // Convert the output cell data into a u64 value.
        let mut buffer = [0u8; 8];
        buffer.copy_from_slice(&output_data[0..8]);
        let output_value = u64::from_le_bytes(buffer);
        
        // Check if the output is one more than the input.
        if input_value + 1 != output_value
        {
            // If no match was found return an error.
            return Err(Error::InvalidCounterValue);
        }
    }
    
    // Return success if all group input and output cells have been checked and no errors were found.
    Ok(())
}
```

The dependencies and boilerplate code are the same in this example as in the previous lessons, so we won't go over them. We'll only go over the main logic of the code.

On lines 15 to 29, we count the group input and group output cells, then perform some basic validation. The Aggregatable Counter type script uses the `GroupInput` and `GroupOutput`. This limits the scope of concern to only other Aggregatable Counter cells. It then checks the number of input cells, which is logically checking if the operation is creating cells, or updating cells. It then checks that the number of input and output cells is equal. This ensures that we have the proper structure to validate an update operation.

On line 32, we loop through all the cell indexes. On lines 35 and 36, we retrieve the data for the group input cell and group output cell at the current cell index.

On lines 38 to 46, we convert the input and output data to u64 values.

On lines 48 to 53, we check that the values are as expected, and return an error if it is not as expected.

On line 57, we return successfully if no errors were found.



