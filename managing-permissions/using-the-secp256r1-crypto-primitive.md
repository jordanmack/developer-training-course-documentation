# Using the Secp256r1 Crypto Primitive

Nervos has the ability to incorporate any new crypto primitives without having to hard fork the chain. To use a new crypto primitive, a new lock script binary can be created using the standard crypto libraries.

 If you open `index.js` from the `Using-the-Secp256r1-Crypto-Primitive-Example` directory, you will see an example using Secp256r1 + SHA256 instead of the default Secp256k1 + Blake2b.

