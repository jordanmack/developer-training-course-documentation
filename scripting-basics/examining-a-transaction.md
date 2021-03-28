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



