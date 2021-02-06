# Using the Default Lock Script

The "default lock script" is the most common method of securing cells on Nervos. The code binaries are included in the genesis block. All wallets and all of the available tooling readily support this lock.

The default lock script uses a combination of the [Secp256k1](https://en.bitcoin.it/wiki/Secp256k1) and [Blake2b](https://en.wikipedia.org/wiki/BLAKE_%28hash_function%29#BLAKE2) algorithms. Secp256k1 is the same algorithm used in Bitcoin and Ethereum to provide private/public key signature functionality. Blake2b is a hashing algorithm that is also used by projects like Cardano and Sia. The combination of locking using Secp256k1 signatures and Blake2b hashing is also referred to as the "Secp256k1-Blake2b-Sighash", or simply the "Sighash".

### Usage in Lumos

Open the `index.js` file from the `Using-the-Default-Lock-Script-Example` directory. If you scroll down to the bottom and find the `main()` function you will see that there are three main sections.

![](../.gitbook/assets/example-flow%20%281%29.png)

1. Initialize - In the first three lines of code in `main()`, we initialize the Lumos configuration, start the Lumos Indexer, and initialize the lab environment.
2. Create Cells - The `createDefaultLockCell()` function creates a cell that uses the default lock.
3. Consume Cells - The `consumeDefaultLockCell()` function consumes the cell with the default lock that we just created.

There is no section for deploying code this time. This is because the default lock a well-known script, and it is deployed to the blockchain in the genesis block.

### Creating a Cell Using the Default Lock

Starting near the top of `index.js` we have the following definitions.

```javascript
// This is the private key, lock arg, and address which will be used.
const privateKey1 = "0x67842f5e4fa0edb34c9b4adbe8c3c1f3c737941f7c875d18bc6ec2f80554111d";
const lockArg1 = "0x988a9c3e74c09dab76c8e41d481a71f4d36d772f";
const address1 = "ckt1qyqf3z5u8e6vp8dtwmywg82grfclf5mdwuhsggxz4e";
```

Provided is a private key and the corresponding lock arg and address for that private key. All three of these values are directly related to each other.

The private key is a randomly generated 256-bit value \(32 bytes\). This is can be used with the Secp256k1 algorithm to derive a 264-bit public key \(33 bytes\). Below is how this would be done in pseudo-code.

```javascript
privateKey = "0x67842f5e4fa0edb34c9b4adbe8c3c1f3c737941f7c875d18bc6ec2f80554111d";
publicKey = derive_secp256k1_public_key(privateKey);
print(publicKey);
>> 0x03fe6c6d09d1a0f70255cddf25c5ed57d41b5c08822ae710dc10f8c88290e0acdf
```

Now that we have the public key, we can create a lock arg. As we mentioned earlier, a lock arg is a 160-bit Blake2b hash of the public key. Specifically, it is a 256-bit Blake2b hash, with the personalization key `ckb-default-hash`, truncated to 160 bits. Below is the pseudo-code to represent this.

```javascript
privateKey = "0x67842f5e4fa0edb34c9b4adbe8c3c1f3c737941f7c875d18bc6ec2f80554111d";
publicKey = derive_secp256k1_public_key(privateKey);
lockArg = truncate(blake2b(publicKey, 256, "ckb-default-hash"), 160);
print(lockArg);
>> 0x988a9c3e74c09dab76c8e41d481a71f4d36d772f
```

The "lock arg" was given its name because it is used in the `args` field of the default lock script to identify the owner of a cell. Since it is derived from the public key, it can provide proof of ownership.

Now let's continue through the relevant parts of the `createDefaultLockCell()` function.

```javascript
// Create a cell using the default lock script.
const outputCapacity1 = intToHex(ckbytesToShannons(61n));
const output1 = {cell_output: {capacity: outputCapacity1, lock: addressToScript(address1), type: null}, data: "0x"};
transaction = transaction.update("outputs", (i)=>i.push(output1));

// Create a cell using the default lock script, but expanded this time.
const outputCapacity2 = intToHex(ckbytesToShannons(61n));
const lockScript2 =
{
	code_hash: "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",
	hash_type: "type",
	args: lockArg1
}
const output2 = {cell_output: {capacity: outputCapacity2, lock: lockScript2, type: null}, data: "0x"};
transaction = transaction.update("outputs", (i)=>i.push(output2));
```



This code creates 10 cells, all of which use the default lock script, and are owned by the account represented by `address1`. To create our lock script, the Lumos function `addressToScript()` is being used. An address and lock script represent the same underlying data, but an address is a single human-readable value that also includes a checksum. Here is what `address1` looks like when it is converted back to a lock script:

```javascript
{
  code_hash: '0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8',
  hash_type: 'type',
  args: '0xc8328aabcd9b9e8e64fbc566c4385c3bdeb219d7'
}
```

The `code_hash` and `hash_type` values indicate the code that should execute, and the `args` field is a piece of data that is being passed to that program. The `args` value represents the owner of the cell, and this is a value that is encoded in a specific way that the default lock is expecting.

You may notice that our `hash_type` has a value of `type`, which is different than we used in previous examples. In the next lesson, we will describe in detail exactly what this means. For now, think of it as meaning that the code that executes can be upgraded in the future.

Let's look at the `account list` command from `ckb-cli`. The pictured account one of the two genesis account which contains a large amount of CKBytes. 

![](../.gitbook/assets/account-list%20%281%29.png)

If you look closely at the values, you will notice that the testnet address is the same one used in our code and the `lock_arg` matches the `args` value of our lock script.

When we refer to the "lock arg", we are specifically talking about the `args` value that is used specifically with the default lock script. This is a commonly used term you will see throughout much of Nervos' tooling. Don't confuse this with the `args` of a lock script. A "lock arg" implies we are using the default lock script. If we're not using the default lock script, the lock script will usually still have an `args` value, but this value could be very different because the value that is used in the `args` depends on what the lock script code expects.

In the case of the default lock script, the `args` is expected to contain a 160-bit Blake2b hash of the Secp256k1 public key that owns the account. 

```text
Private Key:    0xd00c06bfd800d27397002dca6fb0993d5ba6399b4238b2f29ee9deb97593d2bc (32 bytes)
Public Key:     0x03fe6c6d09d1a0f70255cddf25c5ed57d41b5c08822ae710dc10f8c88290e0acdf (33 bytes)
Lock Arg:       0xc8328aabcd9b9e8e64fbc566c4385c3bdeb219d7 (20 bytes)
```



