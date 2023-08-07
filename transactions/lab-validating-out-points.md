# Lab: Validating Out Points

Determine and validate the out points for the two outputs from the transaction in the previous lab exercise.

1. An out point is the tx\_hash (transaction id) of the transaction and the index of the output in the transaction.
2. Once you have your out points, verify that they are valid and the status is "live" using the `rpc get_live_cell` command in ckb-cli. It will also return a `lock_arg` (cell.output.lock.args) which you will also need. We will explain exactly what this terminology means in the next lesson.

```
rpc get_live_cell --tx-hash <TX_HASH> --index <INDEX>
```

Once you have verified your out points, copy both of them along with the `lock_arg` somewhere that they can be retrieved later. We will be using them in the next lesson.



