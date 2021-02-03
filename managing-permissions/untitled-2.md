# Using the Multi-Sig Lock Script

Multi-sig functionality allows a single cell to be owned and unlocked by multiple users. Just like with the previous examples, any cell can use a multi-sig lock by specifying a multi-sig binary executable in the lock script and providing the correct arguments.

Open the `index.js` file from the `Using-the-Multi-Sig-Lock-Script-Example` directory. If you scroll down to the bottom and find the `main()` function you will see that there are three main sections.

1. Initialize - In the first three lines of code in `main()`, we initialize the Lumos configuration, start the Lumos Indexer, and initialize the lab environment.
2. Create Cell - The `createMultisigCell()` function creates a cell that uses the multi-sig lock.
3. Consume Cell - The `consumeMultisigCell()` function consumes the cell with the multi-sig lock that we just created.

### Multi-Sig Configuration

The multi-sig lock offers a few configuration options which are used to generate the lock args when creating the cell and are again used to generate the witness when unlocking the cell.

Near the top of the file, you will find the multi-sig configuration we are using. 

```javascript
// Multi-Sig configuration.
const multisigAddresses =
[
	"ckt1qyqf3z5u8e6vp8dtwmywg82grfclf5mdwuhsggxz4e",
	"ckt1qyqvsv5240xeh85wvnau2eky8pwrhh4jr8ts8vyj37",
	"ckt1qyqywrwdchjyqeysjegpzw38fvandtktdhrs0zaxl4"
];
const multisigReserved = 0;
const multisigMustMatch = 0;
const multisigThreshold = 1;
const multisigPublicKeys = multisigAddresses.length;
```

The `multisigAddresses` variable contains the accounts that have permission to unlock the multi-sig cell. The multi-sig lock doesn't actually take addresses. It uses hashed public keys, just like the default lock script. We will convert these addresses to hashed public keys before they are used.

The next four variables are referred to as S, R, M, and N in the multi-sig RFC. Each one is an unsigned integer ranging from 0 to 255. They have the following meaning:

* `multisigReserved` \(S\) - This is the format version number. As of the current specification, it should always be set to 0.
* `multisigMustMatch` \(R\) - These are the accounts that must sign as approvers, regardless of however other accounts have provided signatures. The value of R represents the "first R accounts must sign". If R is 1, then account `ckt1...xz4e` must always sign as the approver even if one of the other two accounts provided signatures.
* `multisigThreshold` \(M\) - This is the number of signatures that are needed to unlock the cell. If M is set to 1, then any of the three accounts can unlock the cell. If M is set to 2, then a combination of any two accounts will unlock the cell.
* `multisigPublicKeys` \(N\) - This is the number of public keys that are authorized to unlock the cell. 

Example: Say that we wanted to create a multi-sig lock for Alice, Bob, and Charles, where "any two people can unlock the cell, but Charles must approve. Our S/R/M/N values would then be 0/1/2/3, and our hashed public keys would be provided in the order Charles, Alice, Bob. The order of Alice and Bob don't matter, but Charles must be listed first to match the R value.

### Creating a Multi-Sig Cell

Let's go through the `createMultisigCell()` function, skipping straight to the important parts.

```javascript
	// Create a cell that uses the multi-sig lock.
	const outputCapacity1 = intToHex(ckbytesToShannons(61n) + txFee);
	const multisigScript = "0x"
		+ multisigReserved.toString(16).padStart(2, "0")
		+ multisigMustMatch.toString(16).padStart(2, "0")
		+ multisigThreshold.toString(16).padStart(2, "0")
		+ multisigPublicKeys.toString(16).padStart(2, "0")
		+ multisigAddresses.map((address)=>addressToScript(address).args.substr(2)).join("");
	const multisigHash = ckbHash(hexToArrayBuffer(multisigScript)).serializeJson().substr(0, 42);
	const lockScript1 =
	{
		code_hash: MULTISIG_LOCK_HASH,
		hash_type: "type",
		args: multisigHash
	};
	const output1 = {cell_output: {capacity: outputCapacity1, lock: lockScript1, type: null}, data: "0x"};
	transaction = transaction.update("outputs", (i)=>i.push(output1));
```

Starting on line 2, we are setting the capacity to 61 CKBytes + the transaction fee. The minimum for a multi-sig cell is still 61 CKBytes, just like we were using the default lock script. The reason we are adding an extra transaction fee here is to simplify the next transaction by providing the transaction fee now, so we don't have to do another cell collection later. This is probably not something you would do in production, but you will see later on why this makes our example code much more simple.

On line 3 we are constructing what is commonly known as the "multi-sig script". It is a binary structure consisting of the S/R/M/N values, followed by the hashed public keys in order. We're using a hex string to represent this, which is why it starts with "0x" on line 3. Lines 4-7 add the S/R/M/N values. Line 8 takes our `multisigAddresses` and extracts the hashed public key. The multi-sig lock uses the same lock arg as the default lock script, a 160-bit Blake2b hash of the Secp256k1 public key. Since it's the same, we can just strip off the leading "0x" hex prefix using `substr(2)`, then copy it directly.

On line 9 we take the entire multi-sig script structure and create a 160-bit Blake2b hash from it.

On line 10 we construct our lock script. Instead of using the default lock hash, we use the multi-sig lock hash, which is inserted on line 12. On line 14 we put the multi-sig script hash we created on line 9.

The lock args length for a multi-sig cell is 20 bytes \(160 bits\), just like a cell that uses the default lock script. This is why both only require 61 CKBytes of capacity as the minimum. 

On lines 16 and 17 we construct the output cell and insert it into the transaction.

You may notice that nowhere in the cell does it actually contain the multi-sig script that has the configuration. All it has is the hash of the multi-sig script. It's not needed here because that would take up more space to store. Instead, it will be provided as part of the witness when we need to unlock the cell.

### Consuming a Multi-Sig Cell

Next, let's look at the important parts of the `consumeMultisigCell()` function.

```javascript
// Add the cell dep for the lock script.
transaction = transaction.update("cellDeps", (cellDeps)=>cellDeps.push(locateCellDep({code_hash: MULTISIG_LOCK_HASH, hash_type: "type"})));
```

On this line, we're adding our cell deps. However, we're not using the default lock script on any inputs, so we don't need the usual cell deps. This transaction will have only one input, the multi-sig cell, so we need to include the code binary for multi-sig. Just like the default lock, this is considered well-known, and is included in `config.json`. Therefore, we can use Lumos' `locateCellDep()` function to provide the necessary out point.

```javascript
// Add the input cell to the transaction.
const input = await getLiveCell(nodeUrl, multisigCellOutPoint);
transaction = transaction.update("inputs", (i)=>i.push(input));

// Get the capacity sums of the inputs.
const inputCapacity = transaction.inputs.toArray().reduce((a, c)=>a+hexToInt(c.cell_output.capacity), 0n);

// Create a change Cell for the remaining CKBytes.
const outputCapacity2 = intToHex(inputCapacity - txFee);
const output2 = {cell_output: {capacity: outputCapacity2, lock: addressToScript(address1), type: null}, data: "0x"};
transaction = transaction.update("outputs", (i)=>i.push(output2));
```

These blocks of code add the multi-sig cell as an input, calculate the input capacity, then create a change cell that returns the capacity back to `address1`. We don't need to perform any additional cell collection here because the input cell has enough capacity to cover a 61 CKByte change cell, and the transaction fee.

```javascript
// Add in the witness placeholders.
const multisigScript = "0x"
	+ multisigReserved.toString(16).padStart(2, "0")
	+ multisigMustMatch.toString(16).padStart(2, "0")
	+ multisigThreshold.toString(16).padStart(2, "0")
	+ multisigPublicKeys.toString(16).padStart(2, "0")
	+ multisigAddresses.map((address)=>addressToScript(address).args.substr(2)).join("");
const multisigPlaceholder = multisigScript + "0".repeat(130).repeat(multisigThreshold);
const witness = arrayBufferToHex(core.SerializeWitnessArgs(normalizers.NormalizeWitnessArgs({lock: multisigPlaceholder})));
transaction = transaction.update("witnesses", (w)=>w.push(witness));
```

This block of code adds in the witness placeholders for the transaction. The placeholder needed for the multi-sig lock is slightly different than the default lock. When we created the multi-sig cell, we included a hash of the multi-sig script, but we didn't actually include the script. The multi-sig script has the configuration information, so it is needed to unlock, and it is provided during the unlock procedure by placing it into the Witnesses structure.

On lines 2-7, we create a hex string representation of the binary multi-sig script. This is the S/R/M/N values and the list of hashed public keys. This is identical to how it was done before.

Line 8 creates the actual placeholder, consisting of the multi-sig script, and zero-filled placeholders which will later be replaced with signatures. These zero-filled placeholders are necessary because this is used to generate the signing message which will be signed by the authorized private keys. Each Secp256k1 signature requires 65 bytes to store. In hex, every byte is represented by two characters, which is why we are creating a string of zeros that is 130 characters in length. A signature placeholder is needed for every required signature, so we add in one 65 byte zero-filled placeholder for each M value. This process is represented on line 8 with the code `"0".repeat(130).repeat(multisigThreshold)`.

Lines 9-10 serialize this structure in the expected format, and add it to the Witnesses of the transaction.

```javascript
// Sign the transaction.
transaction = secp256k1Blake160Multisig.prepareSigningEntries(transaction);
const signingEntries = transaction.get("signingEntries").toArray();
const signature1 = signMessage(privateKey1, signingEntries[0].message);
const multisigSignature = multisigScript + signature1.substr(2);
const signedTx = sealTransaction(transaction, [multisigSignature]);
```

This block of code generates the signing entries, signs with the required signatures, and seals the transaction by adding the signatures to the transaction.

Lines 2-3 generate the signing entries for the multi-sig specifically since it's using the  `secp256k1Blake160Multisig` library. If we needed to add capacity from cells using the default lock script, then we would need to use the `secp256k1Blake160` library in addition. If there were multiple input cells owned by different accounts, then the `signingEntries` array would hold multiple messages.

Line 4 generates the Secp256k1 signature for the first message. Since we only have a single input, we know that only a single signing entry will be generated.

Line 5 creates the multi-sig signature, which consists of the multi-sig script + each required signature. Our multi-sig configuration's M value \(`multisigThreshold`\) is 1, so we only add a single signature. If the M value is higher, we would continue to append each signature here.

Line 6 seals the transaction by replacing the zero-filled witness values in the transaction with the real signatures. Our transaction only has a single multi-sig cell as an input, so we only need one signature. If our transaction had more input cells with the default lock script which were needed for capacity, then we would have more than one signature here.

