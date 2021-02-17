# Using the Witness

In Nervos, the Witness is a data structure that is the part of the transaction that is designated to hold signature data \(also known as witness data\). This is similar to the [Witness data structure in Bitcoin](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki). Nervos extends the usage of the Witness beyond signatures, to include any data which is needed by the transaction to confirm.

The Witness is similar to the `args` field of a script, but it is distinctly different in several ways.

* The scope of the Witness is a transaction instead of a script on a single cell.
* The data contained within the Witness is still part of the blockchain, but it is not part of the state, and therefore doesn't require state rent.
* The Witness is the part of the transaction that specifically takes into account the design considerations required for signatures.

The most common usage for the Witness is to hold the signatures required by the transaction, but can also be used for more advanced functionality which we will cover in later lessons. In the most simple sense, the Witness is like an `args` field for the transaction.

### The Structure of the Witness

The Witness is a top-level part of a transaction, just like input, outputs, and cell deps. Similarly, the Witness is also segmented the same way, in an array-like structure. This allows the data in the Witness to be matched with inputs and outputs more easily.

Below is a line of code that has been used frequently in our examples. 

```javascript
// Add in the witness placeholders.
transaction = addDefaultWitnessPlaceholders(transaction);
```

This code adds the witness placeholders for the default lock to the transaction. We've used this code many times before, but we never went into detail about what it actually does.





