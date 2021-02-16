# Using Lock Args

We mentioned before that a lock script's `args` field can contain any data in any format. It is up to the developer on what this data should be, and how it is encoded. For a developer to be able to make these decisions, it's important to understand the design decisions involved with a lock script.

```text
{
    "code_hash": "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",
    "hash_type": "type",
    "args": "0x988a9c3e74c09dab76c8e41d481a71f4d36d772f"
}
```

The code above is a typical example of a lock script. The `code_hash` and `hash_type` are those required for the default lock, and the `args` field indicates who the owner is.  The `args` field is providing data to the default lock, so this means that the data has to be formatted to the exact requirements of the default lock.

If the  `code_hash` specified a different lock, then the `args` field would need to contain data that was relevant to that new lock. Let's explore that using a new lock described in the pseudo-code below.

```javascript
function main()
{
    lock_args = load_lock_args();
    ckb_required = int_from_le_bytes(lock_args[0..8]);
    
    input_cells = load_input_cells();
    for(cell in input_cells)
    {
        if(cell.capacity == ckb_required)
        {
            return 0;
        }
    }
    
    return 1;
}
```

This lock is similar to the "CKB 500" example from the previous topic, but there are a few changes. This code will unlock only if there is at least one input cell in the transaction that has a capacity equal to a specific amount. That specific amount is contained within the lock's `args` field.

On lines 3 and 4, the `args` data is loaded, and then it is converted from raw bytes to an integer. It's reading exactly 8 bytes because it is a 64-bit integer.

On lines 6 to 13, the code loads every input cell and checks the capacity. If it finds an input cell with a matching capacity, then it immediately unlocks, otherwise, it will return an error on line 15.



