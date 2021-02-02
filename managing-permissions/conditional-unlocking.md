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

    if total_capacity == 500
    {
        return 0;
    }
    else
    {
        return 1;
    }
}
```

