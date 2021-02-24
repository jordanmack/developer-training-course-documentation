# Creating a JSON Storage Cell

Using a type script, we can create a cell that only allows valid JSON strings to be stored in it. Any time a cell is created, the type script will read the data that the cell is being created with and validate it as a JSON string. Trying to create a cell containing invalid JSON will result in the transaction being rejected.

![](../.gitbook/assets/valid-invalid.png)

On the left of the image is a cell that uses the `jsoncell` type script. The data area of the cell contains a valid JSON string. If this cell were put into a transaction as an output, meaning we are creating this cell, the type script would execute without error, allowing the transaction to proceed.

On the right is a similar cell using the `jsoncell` type script, but the data area contains invalid JSON. If this cell were put into a transaction as an output, the type script would execute and return an error. This transaction would be rejected.

Let's take a look at how this type script would look in pseudo-code.

```javascript
function main()
{
    outputGroup = load_output_group();
    for cell in outputGroup
    {
        if(!is_valid_json(cell.data))
        {
            return 1;
        }
    }

    return 0;
}
```

On line 3, we load the output group. When this is called from a type script, it retrieves all the output cells that have the same type script. The outputs could contain many different cells, but this script is only concerned with those using the `jsoncell` type script.

On lines 4 to 10, we cycle through every cell in the output group, checking the data field of each one. If any of them contain invalid JSON data, an error is returned immediately.

On line 12, we return successfully after no errors are found.

![](../.gitbook/assets/create-transaction-structure%20%284%29.png)





