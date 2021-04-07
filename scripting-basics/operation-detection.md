# Operation Detection

At this point, you should be familiar enough with the Cell Model to see that writing scripts on Nervos is a distinctly unique process. The rules are inherently more abstract and conceptualizing the development of a dapp can be very challenging for those who are new.

When we build a transaction in the Cell Model, we describe the start and end state. The _operation_, meaning the intent of what the transaction is supposed to do, is not expressed directly. There is no `transfer()` function in the Cell Model that clearly indicates the intent. However, by analyzing the cell configuration, we can often determine what the intent was, and this can help make script construction more intuitive.

In the Cell Model, the four most common basic operations are Create, Transfer, Update, and Burn.

* A **Create** operation occurs when a new cell is created for an asset, like a token.
* A **Transfer** operation occurs when a user sends an asset to another user.
* An **Update** operation occurs when a user updates some value within a cell.
* A **Burn** operation occurs when a user destroys the cell containing an asset.

Your script can incorporate many other operations or custom operations for your dapp, but we will focus on these basic four for this lesson.

### Script Logic

Next, we will look at an example of a counter that uses operation detection. We will call this the "ODCounter" for short. We're going to skip the pseudo-code example this time around, and instead jump straight into the real Rust code.

We're going to review the code one section at a time to make it easier to understand, but you can review the full source code at any time by opening the `entry.rs` file in the directory`developer-training-course-script-examples/contracts/odcounter/src`.

```rust
// Import from `core` instead of from `std` since we are in no-std mode.
use core::result::Result;

// Import CKB syscalls and structures
// https://nervosnetwork.github.io/ckb-std/riscv64imac-unknown-none-elf/doc/ckb_std/index.html
use ckb_std::ckb_constants::Source;
use ckb_std::high_level::{load_cell, load_cell_data, QueryIter};

// Import local modules.
use crate::error::Error;
```

This first block of code is the usual imports. There is nothing special about this, but we're going to quickly peek into `errors.rs` to see the possible values of `Error` from line 10.

```rust
/// Error
#[repr(i8)]
pub enum Error
{
    IndexOutOfBound = 1,
    ItemMissing,
    LengthNotEnough,
    Encoding,
    // Add customized errors here...
    CounterValueOverflow,
    InvalidTransactionStructure,
    InvalidInputCellData,
    InvalidOutputCellData,
    InvalidCounterValue,
}
```

On lines 10 to 14, you see we have five distinct custom errors. This is more than any other script we went through in the past. The more complex the application, the more potential there is for errors to occur.

Trying to debug a transaction can be a taxing experience. Having descriptive and distinct error codes can greatly help with this, so it is always recommended that they be included.

```rust
// The modes of operation for the script. 
enum Mode
{
    Burn, // Consume an existing counter cell.
    Create, // Create a new counter cell.
    Transfer, // Transfer (update) a counter cell and increase its value.
}
```

Here we are defining the possible modes \(operations\) that our script will use. You may be wondering why there are only three instead of the four we said we would cover. There are only three here because, for the purposes of our counter, Transfer and Update are exactly the same.

```rust




// Determines the mode of operation for the currently executing script.
fn determine_mode() -> Result<Mode, Error>
{
    // Gather counts on the number of group input and groupt output cells.
    let group_input_count = QueryIter::new(load_cell, Source::GroupInput).count();
    let group_output_count = QueryIter::new(load_cell, Source::GroupOutput).count();
    
    // Detect the operation based on the cell count.
    if group_input_count == 1 && group_output_count == 0
    {
        return Ok(Mode::Burn);
    }
    if group_input_count == 0 && group_output_count == 1
    {
        return Ok(Mode::Create);
    }
    if group_input_count == 1 && group_output_count == 1
    {
        return Ok(Mode::Transfer);
    }
    
    // If no known code structure was used, return an error.
    Err(Error::InvalidTransactionStructure)
}

// Validate a transaction to create a counter cell.
fn validate_create() -> Result<(), Error>
{
    // Load the output cell data and verify that the value is 0u64.
    let cell_data = load_cell_data(0, Source::GroupOutput)?;
    if cell_data != 0u64.to_le_bytes().to_vec()
    {
        return Err(Error::InvalidOutputCellData);	
    }
    
    Ok(())
}

// Validate a transaction to transfer (update) a counter cell and increase its value.
fn validate_transfer() -> Result<(), Error>
{
    // Load the input cell data and verify that the length is exactly 8, which is the length of a u64.
    let input_data = load_cell_data(0, Source::GroupInput)?;
    if input_data.len() != 8
    {
        return Err(Error::InvalidInputCellData);
    }
    
    // Load the output cell data and verify that the length is exactly 8, which is the length of a u64.
    let output_data = load_cell_data(0, Source::GroupOutput)?;
    if output_data.len() != 8
    {
        return Err(Error::InvalidOutputCellData);
    }
    
    // Create a buffer to use for parsing cell data into integers.
    let mut buffer = [0u8; 8];
    
    // Convert the input cell data to a u64 value.
    buffer.copy_from_slice(&input_data[0..8]);
    let input_value = u64::from_le_bytes(buffer);
    
    // Convert the output cell data to a u64 value.
    buffer.copy_from_slice(&output_data[0..8]);
    let output_value = u64::from_le_bytes(buffer);
    
    // Check for an overflow scenario.
    if input_value == u64::MAX
    {
        return Err(Error::CounterValueOverflow);
    }
    
    // Ensure that the output value is always exactly one more that in the input value.
    if input_value + 1 != output_value
    {
        return Err(Error::InvalidCounterValue);
    }
    
    Ok(())
}

// Main entry point.
pub fn main() -> Result<(), Error>
{
    // Determine the mode and validate as needed.
    match determine_mode()
    {
        Ok(Mode::Burn) => return Ok(()),
        Ok(Mode::Create) => validate_create()?,
        Ok(Mode::Transfer) => validate_transfer()?,
        Err(e) => return Err(e),
    }

    Ok(())
}
```



