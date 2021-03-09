# Using Type Scripts

Nervos has two types of scripts, lock scripts and type scripts. We've worked with many lock scripts in the previous examples, and now we will start working with type scripts.

There are a number of similarities between lock scripts and type scripts. Both scripts have the same `code_hash`, `hash_type`, and `args` fields. Both are executing RISC-V binaries in CKB-VM. Both have access to the same details of a transaction during execution. Both play a part in answering the overshadowing question: "is this transaction valid?"

However, there are strong semantical differences between lock scripts and type scripts. Lock scripts are concerned with ownership, and type scripts are concerned with state transitions. Lock scripts define how a cell is accessed, type scripts define how a cell behaves. When used as a form of identification, lock scripts define the owner or a cell, and type scripts define what the cell is.

### Script Execution Lifecycle

Many of the details surrounding lock scripts and type scripts are the same. Based on the similarities mentioned above, it might seem like the two are interchangeable. In some cases, this is true. Both have the same three fields, `code_hash`, `hash_type`, and `args`. If you were to swap the information of these three fields between the lock script and type script on a cell, then you would be swapping the type script and lock script. The code binaries would still execute, however, there is a high likelihood that the scripts would not function as intended.

There is one major distinction between lock scripts and type scripts in regards to execution: _when_ they execute. **Lock scripts execute on input cells. Type scripts execute on both input cells and output cells.** It may not seem like that big of a difference, but this is the single factor that leads to all the semantical differences mentioned above. If lock scripts and type scripts executed at the exact same times, then the two would be completely interchangeable.

A lock script is concerned with ownership, and this is the reason that a lock script must execute on inputs. Let's look at the default lock as an example. When a cell is added as an input, it will be consumed. The value contained in the cell is being extracted for use. The default lock is verifying that the owner authorized its use in the transaction by providing a signature, allowing the cell to be unlocked. If the same lock script executed on outputs, then it would also require a signature from the recipient to complete the transaction. This is not desirable. More logic could be added to the lock script to accommodate this situation if a lock script executed on both inputs and outputs, but it is more efficient to simply not check. What is most important is that the use of the values within a cell was authorized. What is done with the value after unlock is not a consideration for the default lock.

The default lock does not take into consideration what is done with the value extracted from a cell once it is unlocked, but that doesn't mean that lock scripts are incapable of this. The Anyone Can Pay \(ACP\) lock script that we mentioned earlier is a good example of this. The ACP lock script will automatically unlock when the user is receiving additional value. To identify this, the ACP lock is examining both the inputs and the outputs of the transaction.

A type script is concerned with state transition, and this is the reason that a type script must execute on both inputs and outputs. A very common use for type scripts is the creation of tokens. The type script will contain all the logic of the token, including the monetary policy. One of the most simple monetary policy requirements is that a user cannot create more tokens out of thin air. This is enforced with a single simple rule: `input_tokens >= output_tokens`. In effect, this means you cannot send more tokens than you already have.

A type script can easily enforce the logic of this rule. When a cell uses this type script, it will execute and validate the transaction to ensure token balances on both input cells and output cells are in compliance with the rule. However, it can only perform this validation if it actually executes.

![](../.gitbook/assets/transaction-compare%20%281%29.png)

In the above image, there are four simplified transactions where all cells are using a simple token type script that enforces the `input_tokens >= output_tokens` rule. We are omitting CKBytes and TX fees to make it easier to understand on a conceptual basis.

Transaction \#1 is a basic transfer. Alice is transferring 5 tokens to herself. The type script executes and checks to make sure that the rule is enforced. Alice has 5 tokens and is sending 5 tokens, so the type script would execute successfully.

Transaction \#2 is a burn operation. Bob doesn't want his tokens anymore, so he is destroying them. He provides 5 tokens to the transaction, but there are no outputs. Since the type script rule uses `>=`, this is a perfectly valid transaction and would execute successfully.

Transaction \#3 is another transfer. Alice has 5 tokens and is sending 3 to Bob, then sending 2 back to herself as change. The input and output token balances are the same, so this transaction is valid and would execute successfully.

Transaction \#4 is a mining operation. Charlie is attempting to create 5 tokens out of nothing. This is in violation of the token rule, and this transaction is therefore invalid and would fail.

Type scripts execute on both inputs and outputs. What if type scripts were like lock scripts, and executed on inputs but not on outputs? Transactions \#1, \#2, and \#3 would be unchanged since the token type script is still executing on inputs. However, transaction \#4 would result differently. If type scripts did not execute on outputs, then the token type script would not execute at all in transaction \#4. This would allow the transaction to succeed, allowing tokens to be created from nothing. This is why it is critical for type scripts to execute on both lock scripts and type scripts.

### Always Success and Always Fail

In the last chapter, we introduced the always success \(AS\) and always fail \(AF\) scripts. These are the most simple scripts that can be created. The AS script will always execute successfully, and the AF script will always result in an error.

The AS and AF scripts are two examples of script binaries that will function identically if they are used for the lock script or the type script. However, due to the difference of _when_ execution occurs, the end result will be different.

Let's look at the AS binary being used in a lock script first.

![](../.gitbook/assets/always-success-lock-script.png)

Once again, we're using a simplified transaction representation that omits CKBytes and TX fees to make it easier to understand on a conceptual basis. Focus only on how the lock script would execute.

Transaction \#1 is a minting operation. We are creating a cell with the AS lock script.

Transaction \#2 is a transfer operation. We are transferring a cell using the AS lock script.

Transaction \#3 is a burn operation. We are destroying a cell using the AS lock script.

All three of these transactions would be successful. In transaction \#1, the AS lock script would not execute because lock scripts do not execute on outputs. In transactions \#2 and \#3, the AS lock script would execute successfully.

![](../.gitbook/assets/always-fail-lock-script.png)



