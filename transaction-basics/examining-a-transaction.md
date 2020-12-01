# Examining a Transaction

### Lesson Introduction

In this lesson, we will examine the transaction that was created in the last lab exercise. To do this we will, once again, be using the ckb-cli command line tool. You will need the transaction id that was generated from the last lab exercise, so make sure it's handy.

### Retrieving Transaction Details

To get the details of a transaction you use the `rpc get_transaction` command in ckb-cli. In the command below, replace `<TRANSACTION_ID>` with the transaction you generated in the last lab exercise.

```text
rpc get_transaction --hash <TRANSACTION_ID>
```

Your output should look similar to this, but some of your hashes will be different.

![](../.gitbook/assets/tx.png)

The two sections we will focus on are the inputs and outputs. All the other sections can be ignored for the time being. If it seems confusing as we're going through it, don't be intimidated. By the end of this lesson, it should be much more clear.

### Inputs and Outputs

In the previous lab exercise, you were asked to send 100,000 CKBytes between two accounts. Looking closer that the outputs, you should see something familiar.

![](../.gitbook/assets/tx-inputs-outputs.png)

One of the outputs has 100,000 capacity. Remember, capacity is another way of describing the CKBytes. This is the 100,000 CKBytes that was transferred from the sending account to the receiving account.

Below that you see another output with 199,999.9999 capacity. This is the change of the transaction. Think of it like paying for a $5 item with a $20 bill. You would hand your $20 to the cashier, and they would hand you $15 back as change. This is the same basic process but in a purely digital system, our bills are often not in the same denomination.

These CKBytes had to have been transferred from somewhere, and that place is the inputs. If we look at the inputs, it has one "previous\_output" and it gives a `tx_hash` and `index` as a reference. If we use ckb-cli to look up that transaction hash and look at the matching index, we see an output with 300,000 capacity.

![](../.gitbook/assets/tx-previous-output.png)

Every input is an output from a previous transaction, and that's why it's called a "previous output" in ckb-cli. A transaction can have many outputs, which is why the index is specified in addition to the tx\_hash. The combination of a tx\_hash and an index is called an "outpoint", and it is used to describe where an output originates from.

The image below combines the two images above so the formatting on the inputs matches the outputs. This should help to clarify what's going on in the transaction.

![](../.gitbook/assets/tx-combined.png)

With this combined image, the amounts should start to make more sense. The inputs have a total of 300,000 CKBytes. The outputs have one for 100,000 CKBytes, and one for 199,999.9999 CKBytes, for a total of 299,999.9999 CKBytes.

There is a difference of 0.0001 CKBytes. This difference is the transaction fee that we specified when sending the transaction. Whenever there is an unaccounted difference in CKBytes between the inputs and outputs, that is the fee paid to miners for processing the transaction.

With the transaction fee taken into account, the CKBytes are now 100% accounted for.

* The sender started with 300,000 CKBytes.
* The sender sent 100,000 CKBytes to the receiver.
* The sender sent 199,999.9999 CKBytes back to themselves as change.
* The sender paid a 0.0001 CKByte transaction fee to the miners.

### Lab Exercise

Task: Determine the outpoints for the two outputs from the transaction in the previous lab exercise.

1. An outpoint is the tx\_hash of the transaction and the index of the output in the transaction.
2. Once you have your outpoints, verify that they are valid and the status is "live" using the `rpc get_live_cell` command in ckb-cli. We will explain exactly what this terminology means in the next lesson.

Once you have verified your outpoints, copy them somewhere that they can be retrieved later. We will be using them in the next lesson. 

