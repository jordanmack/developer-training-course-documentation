# Lab: Send 100,000 CKBytes

From account `ckt1...gwga`, send 100,000 CKBytes to a newly created account using `ckb-cli`.

1. Use the command `account new` to create a new account.
2. Use the command `account list` to see all your addresses.
3. Use the command `wallet transfer` to send CKBytes.

```
wallet transfer --from-account <ADDRESS> --to-address <ADDRESS> --capacity <CKBYTES>
```

Once submitted, your transaction ID will be printed on the screen. We will use this in the next section, so be sure to copy this value somewhere that it can be retrieved later. &#x20;
