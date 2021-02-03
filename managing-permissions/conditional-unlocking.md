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

This code should be fairly easy to understand. The input cells are loaded from the transaction, and then the capacity of each input cell is tallied. If the total input capacity is exactly 500 CKBytes, then the lock script will return with 0, indicating success.

This lock script is conceptually different than the default lock script because it shows how a script can expand its scope of concern. The code is examining all the input cells unconditionally. It doesn't matter if the input cells have a matching lock script, or are even owned by different people. All the code cares about is the input capacity amount.

This code is insecure and unsafe to use outside of a test environment, but it is a good example to demonstrate how funds can be unlocked with smart contract-like conditions instead of signatures.

### Usage in Lumos

Open the `index.js` file from the `Conditional-Unlocking-Example` directory and scroll down to the `main()` function. Just like the code from the previous topic, this code has four main sections.

![](../.gitbook/assets/example-flow.png)

This should look familiar because it is the same basic process. All that is changing is the lock script in use and a few details in the transaction structure which we'll explain.

