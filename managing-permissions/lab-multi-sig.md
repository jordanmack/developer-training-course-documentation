# Lab: Creating a 2/3 Multi-Sig Cell

Complete the transaction in `index.js` found in the folder `Lab-Creating-a-2-3-Multi-Sig-Cell-Exercise` by adding code and values as necessary.

Your task is to create the multi-sig configuration by filling in the addresses and S/R/M/N values, then create the multi-sig script, calculate the multi-sig hash, and populate the lock script.

Your resulting transaction should contain:

* One or more input cells from `address1` to use as capacity.
* One multi-sig output with exactly 61 CKBytes of capacity.
* One output to `address1` with the change from the transaction, if necessary.
* A transaction fee.

![](../.gitbook/assets/transaction-structure%20%283%29.png)

In this lab exercise, we will descript the syntax, but it is up to you to construct it. You may copy and paste some of your code from previous exercises to complete this lab exercise if needed, but it's recommended that you try to write as much of the code as possible without looking at previous examples.

1. The multi-sig addresses you must use are listed below in the exact order necessary:

   ```text
   ckt1qyqf3z5u8e6vp8dtwmywg82grfclf5mdwuhsggxz4e
   ckt1qyqvsv5240xeh85wvnau2eky8pwrhh4jr8ts8vyj37
   ckt1qyqywrwdchjyqeysjegpzw38fvandtktdhrs0zaxl4
   ```

2. Configure the S/R/M/N values to require **any two** of the authorized accounts to sign.
3. Configure the `multisigScript` as a hex string consisting of the S/R/M/N values, followed by the hashed public keys of the authorized accounts.
   * Hint: You can use Lumos' `addressToScript()` function to convert an address to a lock script object. The \`args\` values are the hashed public keys you need. Make sure your hex string does not include extra "0x" hex prefixes in the middle when you concatenate values.
4. Generate the `multisigHash` value as a 160-bit Blake2b hash of the multi-sig script.
   * Hint: You can use the `ckbHash().serializeJson()` function to generate a 256-bit Blake2b hash with the proper personalization, as a hex string. You will need to truncate this 256-bit hash to 160 bits. The `ckbHash()` function requires an `ArrayBuffer`, which can be created using the `hexToArrayBuffer()` utility function.
5. Generate the `lockScript1` object to use the multi-sig lock.
   * Hint: The object should have three keys: `code_hash`, `hash_type`, and `args`. The code hash and hash type values can be found in the `config.json` file. The code hash is also available as a constant in the shared library. The args value should be set to the multi-sig script hash

Run your code by opening a terminal to the `Lab-Creating-a-2-3-Multi-Sig-Cell-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-Creating-a-2-3-Multi-Sig-Cell-Solution` folder.

Once your code successfully executes, the resulting transaction ID will be printed on the screen.

