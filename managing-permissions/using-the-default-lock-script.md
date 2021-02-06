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
>> 0x02fdd2f22bb7605479655ef1fc7f55699d5002712cbc7417e3e4a4c86274e709e4
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

```javascript
privateKey = "0x67842f5e4fa0edb34c9b4adbe8c3c1f3c737941f7c875d18bc6ec2f80554111d";
publicKey = derive_secp256k1_public_key(privateKey);
lockArg = truncate(blake2b(publicKey, 256, "ckb-default-hash"), 160);
lockScript = {
                  "code_hash": "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",
                  "hash_type": "type",
                  "args": lockArg
             };
```

The `code_hash` value you see in the lock script above is for the default lock on a local development blockchain. The default lock requires the lock arg to be used as the `args` value of the lock script. It is always important to make sure that the formatting of the data in the `args` matches the requirements of the lock that is being used. Failure to do so would result in the cell being unlockable, and the assets contained within the cell would be lost.

```javascript
privateKey = "0x67842f5e4fa0edb34c9b4adbe8c3c1f3c737941f7c875d18bc6ec2f80554111d";
publicKey = derive_secp256k1_public_key(privateKey);
lockArg = truncate(blake2b(publicKey, 256, "ckb-default-hash"), 160);
lockScript = {
                  "code_hash": "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",
                  "hash_type": "type",
                  "args": lockArg
             };
lockHash = blake2b(serialize(lockScript), 256, "ckb-default-hash");
print(lockHash);
>> 0x6ee8b1ea3db94183c5e5a47fbe82110101f6f8d3e18d1ecd4d6a5425e648da69
```

A lock hash is a 256-bit Blake2b hash of the lock script after it has been serialized to binary. The [Molecule](https://github.com/nervosnetwork/molecule) library is used by Nervos for most binary serialization operations. All three values of the lock script are required to describe the ownership of a cell. The lock hash is a single value that can represent all three values in a shorter form.

The lock hash is the format that a CKB node uses internally to determine ownership. This is the reason why querying for cells with a specific lock script always requires all three values. All three must be provided in order to generate the lock hash which can be located by the CKB node.

```javascript
privateKey = "0x67842f5e4fa0edb34c9b4adbe8c3c1f3c737941f7c875d18bc6ec2f80554111d";
publicKey = derive_secp256k1_public_key(privateKey);
lockArg = truncate(blake2b(publicKey, 256, "ckb-default-hash"), 160);
lockScript = {
                  "code_hash": "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",
                  "hash_type": "type",
                  "args": lockArg
             };
lockHash = blake2b(serialize(lockScript), 256, "ckb-default-hash");
address = ckb_address_encode(serialize(lockScript));
print(address);
>> ckt1qyqf3z5u8e6vp8dtwmywg82grfclf5mdwuhsggxz4e
```

An address on Nervos is a lock script that has been serialized with Molecule and encoded using the [CKB address format](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0021-ckb-address-format/0021-ckb-address-format.md). An address is a single value that represents all three fields of a lock script, similar to a lock hash. However, unlike a lock hash, an address is a human-readable format that includes a checksum to prevent typos and is reversible back to a lock script since it is an alternate encoding of the data, not a hash of the data.

![](../.gitbook/assets/lock-value-relationships.png)

The above image shows the relationship between these values. All the values are ultimately derived from the initial private key and are used to represent ownership in different ways.

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

This code is creating two identical output cells, but the lock script is shown in two different ways. The first method uses the Lumos function `addressToScript()` to convert the address to a lock script. The second method fully defines a lock script. Both lock scripts in this code are identical and use the default lock script.

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



