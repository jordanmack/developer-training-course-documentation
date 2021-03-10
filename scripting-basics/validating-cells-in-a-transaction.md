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

On lines 6 to 8, we load the input cells and cycle through them, but we don't actually look at the content of the cells since we are just counting them. It may seem like it would be more efficient to just use `input_cells.len()` to get the number of cells that were returned, but this is not actually possible in real code. This is because cells can only be loaded one at a time in a syscall.

Here is the real code that uses the Capsule framework, and is written in Rust. This code is available in the Developer Training Course Scripts repo in the file `contracts/ic3type/src/entry.rs` if you want to examine it.

```rust
// Import from `core` instead of from `std` since we are in no-std mode
use core::result::Result;

// Import heap related library from `alloc`
// https://doc.rust-lang.org/alloc/index.html
use alloc::{vec::Vec};

// Import CKB syscalls and structures.
// https://nervosnetwork.github.io/ckb-std/riscv64imac-unknown-none-elf/doc/ckb_std/index.html
use ckb_std::ckb_constants::Source;
use ckb_std::ckb_types::{packed::CellOutput};
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
	let input_cells: Vec<CellOutput> = QueryIter::new(load_cell, Source::Input).collect();
	for _ in input_cells.iter()
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





