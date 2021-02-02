# Conditional Unlocking

When the lock script executes it has access to any values that are provided in the transaction. This includes the inputs, outputs, and cell deps. The lock script can use any of these resources as part of its criteria to determine if the cell should unlock or not.

Let's look at another example lock script in pseudo-code. This one will examine the transaction, and only unlock if the total capacity in all of the inputs is exactly 500 CKBytes.

```javascript

```

