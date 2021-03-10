# Validating Cells in a Transaction

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

This should look very similar to lock script pseudo-code from past lessons. There are only a few small details to point out.

On lines 6 to 8, we load the input cells and cycle through them, but we don't actually look at the content of the cells since we are just counting them. It may seem like it would be more efficient to just use `input_cells.len()` to get the number of cells that were returned, but we wanted this code to better represent the real process. In actuality, cells can only be loaded one at a time in a syscall.

Here is the real code that uses the Capsule framework, and is written in Rust. This code is available in the Developer Training Course Scripts repo in the file `contracts/ic3type/src/entry.rs`.

```rust
// Import from `core` instead of from `std` since we are in no-std mode
use core::result::Result;

// Import CKB syscalls and structures.
// https://nervosnetwork.github.io/ckb-std/riscv64imac-unknown-none-elf/doc/ckb_std/index.html
use ckb_std::ckb_constants::Source;
use ckb_std::high_level::{load_cell, QueryIter};

// Import local modules.
use crate::error::Error;

// Constants.
const CELLS_REQUIRED: u64 = 3;

// Main entry point.
pub fn main() -> Result<(), Error>
{
	// Track the number of input cells that exist.
	let mut cell_count = 0;

	// Cycle through all the input cells to count them.
	for _cell in QueryIter::new(load_cell, Source::Input)
	{
		cell_count += 1;
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

The beginning of the file is all imports, so we will skip straight to line 13. This is the number of input cells required to unlock the cell. We hard-coded this to keep the code simple, but we'll use the script args in the next example.

On lines 22 to 25, we load and count all the input cells. The [load\_cell\(\)](https://nervosnetwork.github.io/ckb-std/riscv64imac-unknown-none-elf/doc/ckb_std/high_level/fn.load_cell.html) function is the syscall we use to query for a single cell using an index and a source as arguments. The index value is the index of a cell in the transaction, and the source is where to load the cell from. If we used `load_cell(0, Source::Input)`, it would load the first input cell. 

The `load_cell()` function normally returns a `Result<CellOutput, SysError>`, but when combined with `QueryIter` we can quickly iterate through all the cells available. You can use `QueryIter` with most of the syscall functions available.

On lines 28 to 31, we check if the cell count matches the cells required, and return successfully if it matches.

On line 34, we return an error if the cell count didn't match. The error `Unauthorized` is a custom error code that is defined in the `contracts/ic3type/src/error.rs` file. Error codes are defined by the individual script, meaning you can define as many error codes as you need here.



