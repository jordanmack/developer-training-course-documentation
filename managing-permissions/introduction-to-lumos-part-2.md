# Lab: Unlocking a 2/3 Multi-Sig

Complete the transaction in `index.js` found in the folder `Lab-Unlocking-a-2-3-Multi-Sig-Cell-Exercise` by adding code and values as necessary.

Your task is to create the multi-sig configuration by filling in the addresses and S/R/M/N values, then create the multi-sig script, populate the cell deps, create the proper witness placeholders, then provide the necessary signatures.

Your resulting transaction should contain:

* One multi-sig input cell.
* One or more additional input cells from `address1` to use as capacity.
* One output to `address2` with the change from the transaction.
* A transaction fee.

![](../.gitbook/assets/transaction-structure%20%284%29.png)

In this lab exercise, we will descript the syntax, but it is up to you to construct it. You may copy and paste some of your code from previous exercises to complete this lab exercise if needed, but it's recommended that you try to write as much of the code as possible without looking at previous examples.

1. The multi-sig addresses you must use are listed below but they are **not** in the correct order. They must be ordered correctly to match the S/R/M/N values in the next step.

   ```text
   ckt1qyqf3z5u8e6vp8dtwmywg82grfclf5mdwuhsggxz4e
   ckt1qyqvsv5240xeh85wvnau2eky8pwrhh4jr8ts8vyj37
   ckt1qyqywrwdchjyqeysjegpzw38fvandtktdhrs0zaxl4
   ```

2. Configure the S/R/M/N values to require **any two** of the authorized accounts to sign, but one of them **must be** `ckt1...xz4e`.
3. Configure the `multisigScript` as a hex string consisting of the S/R/M/N values, followed by the hashed public keys of the authorized accounts.
   * Hint: You can use Lumos' `addressToScript()` function to convert an address to a lock script object. The \`args\` values are the hashed public keys you need. Make sure your hex string does not include extra "0x" hex prefixes in the middle when you concatenate values.
4. Configure the cell deps as needed.
   * Hint: You will need to include a cell dep for every unique lock script binary that is used.
5. Generate the appropriate witness placeholders.
   * Hint: A zero-filled placeholder is needed for the first occurrence of each unique lock script.
6. Generate the signing entities and sign them with the correct signatures.
   * Hint: You will need to generate signing entities with both `secp256k1Blake160.prepareSigningEntries()` and `secp256k1Blake160Multisig.prepareSigningEntries()` since both the default lock and the multi-sig lock are used on input cells.

Run your code by opening a terminal to the `Lab-Unlocking-a-2-3-Multi-Sig-Cell-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-Unlocking-a-2-3-Multi-Sig-Cell-Solution` folder.

Once your code successfully executes, the resulting transaction ID will be printed on the screen.

