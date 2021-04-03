# Managing Script State

Script state is any form of data that needs to persist between transactions. This can include things like token balances, NFT attributes, or authentication information. State data is stored in a cell's data field, just like the other data we have stored. However, in our previous examples the script logic had little meaningful interaction with any values stored within a cell.

To demonstrate how we can work with script state, we will build a basic counter type script. The cell's data area will hold a single number that must be incremented by exactly 1 each time it is included in a transaction. We will call this the "Counter" type script.

![](../.gitbook/assets/valid-invalid-transaction%20%282%29.png)

On the top left of the image is a transaction where the Counter type script is used. The data area of the input cell contains a value of 1, and the output cell contains a value of 2. The type script will execute without error and the transaction will process successfully.

On the bottom left is a similar transaction using the Counter type script. The input cell has a value of 1, and the output cell has a value of 3. This is incrementing by more than 1. When the type script executes it will return an error and this transaction would be rejected by the network.

### UNFINISHED BELOW

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

Now let's look at the real version of the Data10 type script, written in Rust. This is located in the `entry.rs` file in`developer-training-course-script-examples/contracts/data10/src`.

