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

Next, we will skip to the `main()` function and work through the application in the order it would execute.

```rust
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

The flow of the `main()` function is fairly simple:

1. Determine the mode of operation.
2. Validate according to the detected mode.
3. Return success or failure.

Starting with step 1, let's look at how `determine_mode()` works.

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
```

On lines 4 to 6, we count the group inputs and outputs.

On lines 8 to 20, we detect which mode of operation is in use based on the counts we just collected.

On lines 9 to 12, if the input count is 1, and the output count is 0, a burn operation is detected. Input cells are always consumed, and if a new cell is not created, the asset is effectively destroyed. Our previous counter scripts did not include burn functionality because we were trying to keep our script as simple as possible. However, burn functionality is highly recommended, because it allows the user to recover the CKBytes in the cell and remove it from the state once it is no longer needed.

On lines 13 to 16, if the input count is 0, and the output count is 1, a create operation is detected. This is also known as a minting operation. Most asset types will need to have some kind of special conditions for a create operation. It's common for the mining operation to be restricted in some way for store of value assets like tokens. We will demonstrate this in the next lesson.

On lines 17 to 20, if the input count is 1 and the output count is 1, a transfer operation is detected. A transfer is defined as the input cell being consumed, and the output cell is created with the same values, but having a different lock script which indicates a different owner.

As we mentioned earlier, we are combining the transfer operation with the update operation. An update is defined as the input cell being consumed, and the output cell is created with different values, but the same lock script. If the output cell was created with different values AND a different lock script, then both a transfer and update operation are occurring at the same time. For the purposes of our counter, we do not want to treat any of these operations differently, so we are combining these into a single mode.

On line 23, if no supported count pattern is recognized, we return an `InvalidTransactionStructure` error.

Now that we have detected the mode, let's look at the `main()` function once again.

```rust
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

If a burn operation is detected \(line 7\), we can immediately return success. If an error is detected \(line 10\), we immediately return the error. For create and transfer operations, we need to go through another layer of validation. Let's take a look at `validate_create()` first, and then the `validate_transfer()` function.

```rust
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
```

This code validates a create operation and should be fairly straightforward. It loads the cell data, and it ensures that the data is a zero u64 value. In the previous counter examples, we didn't check the value that a cell was created with. We skipped it to make the logic simpler, but it is generally a good idea to do so. This ensures that cells cannot be created with an invalid value that could cause undesired behavior.

```rust
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
```

On lines 4 to 9, we load the data from the first group input cell, then check that its data is exactly 8 bytes; the size of a u64.

On lines 11 to 16, we load the data from the first group output cell, and perform the same validation as we did on the input.

The check on the input data is a bit redundant. Since we validate the data of the output in `validate_create()` and we also validate the data of the output in `validate_transfer()`, there is no way for a cell to be created with malformed data. However, as we stated earlier, we do not consider it bad practice to check anyway.

On lines 21 to 27, we convert the raw input and output data to u64 values.

On lines 29 to 33, we check for an overflow scenario. The counter can only go so high before the value overflows and goes back to zero. It is extremely unlikely we would hit this scenario using a u64 value, but it is still good practice to check and deliver a proper error message.

On lines 35 to 39, we check that the output value is exactly one more than the input value, and deliver an `InvalidCounterValue` error if it doesn't match.

The `validate_transfer()` function is the longest chunk of code in our script, but most of it is data validation and conversion. Some of our previous counters skipped some of the data validation to keep things more simple in our examples, but it is always recommended that you never cut corners on a script intended for production. If you look closely, line 36 is the only piece of real counter logic! 

Operation detection can be a helpful approach to better understand how a complex program operates. However, it may not always be the best solution. Operation detection can sometimes lead to unnecessary complications or functionality restrictions.

Our example had three discrete modes of operation, create, transfer/update, and burn. It is not aggregatable and is only capable of processing one mode at a time. However, it is possible to create an aggregatable counter that can handle all modes simultaneously without using operation detection.

What you should use will depend on the specifics of your project, and the architecture of your dapp.

