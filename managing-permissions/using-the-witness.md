# Using the Witness

In Nervos, the Witness is a data structure that is the part of the transaction that is designated to hold signature data \(also known as witness data\). This is similar to the [Witness data structure in Bitcoin](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki). Nervos extends the usage of the Witness beyond signatures, to include any data which is needed by the transaction to confirm.

The Witness is similar to the `args` field of a script, but it is distinctly different in several ways.

* The scope of the Witness is a transaction instead of a script on a single cell.
* The data contained within the Witness is not part of the state, and therefore doesn't require state rent.
* The Witness is the part of the transaction that specifically takes into account the design considerations required for signatures.



