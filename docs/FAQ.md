??? question "I've ran into an issue with Quorum, where do I get support?"
    There are two places Quorum engineering team monitors on an on-going basis: issues in this and related repositories and on Quorum Slack. Quorum Slack is the best place to query the community and get immediate help. Auto-inviter is available [here](https://clh7rniov2.execute-api.us-east-1.amazonaws.com/Express/).
    
??? question "How does Quorum achieve Transaction Privacy?"
    Quorum achieves Transaction Privacy by:
    
     1. Enabling transaction Senders to create a private transaction by marking who is privy to that transaction via the `privateFor` parameter
     2. Replacing the payload of a private transaction with a hash of the encrypted payload, such that the original payload is not visible to participants who are not privy to the transaction
     3. Storing encrypted private data off-chain in a separate component called the Transaction Manager (provided by [Constellation](https://github.com/jpmorganchase/constellation) or [Tessera](https://github.com/jpmorganchase/tessera)).  The Transaction Manager distributes the encrypted data to other parties that are privy to the transaction and returns the decrypted payload to those parties 
    
    Please see the [Transaction Processing](../Transaction%20Processing/Transaction%20Processing) page for more info.
    
??? question "How does Quorum achieve consensus on Private Transactions?"
    In standard Ethereum, all nodes process all transactions and so each node has the same state root.  In Quorum, nodes process all 'public' transactions (which might include reference data or market data contracts for example) but only process the private transactions that they are party to.  
    
    Quorum nodes maintain two Patricia Merkle Tries: one for private state and one for public state. As a result, block validation includes a **state** check on the new-to-Quorum `public state root`. Block validation also includes a check of the `global Transaction hash`, which is a hash of **all** Transactions in a block - private and public. This means that each node is able to validate that it has the same set of Transactions as other nodes.  Since the EVM is provably deterministic through the synchronized public state root, and that the Private Transaction inputs are known to be in sync across nodes (global Transaction Hash), private state synchronization across nodes can be implied.  In addition, Quorum provides an API call, `eth_storageRoot`, that returns the private state hash for a given transaction at a given block height, that can optionally be called at the application layer to specifically perform an off-chain state validation with a counterparty.
    
    Please see the [Quorum Consensus](../Consensus/Consensus) and [Transaction Processing](../Transaction%20Processing/Transaction%20Processing) pages for more info.

??? question "Are there any restrictions on the transaction size for private transactions (since they are encrypted)?"
    The only restriction is the gas limit on the transaction. Constellation/Tessera does not have a size limit (although maybe it should be possible to set one). If anything, performing large transactions as private transactions will improve performance because most of the network only sees hash digests. In terms of performance of transferring large data blobs between geographically distributed nodes, it would be equivalent performance to PGP encrypting the file and transferring it over http/https..so very fast. If you are doing sequential transactions then of course you will have to wait for those transfers, but there is no special overhead by the payload being large if you are doing separate/concurrent transactions, subject to network bandwidth limits. Constellation/Tessera does everything in parallel.

??? question "Should I include originating node in private transaction?"
    No, you should not. In Quorum, including originating node's `privateFor` will result in an error. If you would like to create a private contract that is visible to the originating node only please use this format: `privateFor: []` per https://github.com/jpmorganchase/quorum/pull/165

??? question "Is it possible to run a Quorum node without Transaction Manager?"
    It is possible to run a node without a corresponding Transaction Manager, to do this instead of a matching Tessera/Constellation node's socket configuration should be set to `PRIVATE_CONFIG=ignore ...`. The node running such configuration is not going to broadcast matching private keys (please ensure that there is no transaction manager running for it) and will be unable to participate in any private transactions.

??? info "Known Raft consensus node misconfiguration"
    Please see https://github.com/jpmorganchase/quorum/issues/410
    
??? question "Is there an official docker image for Quorum/Constellation/Tessera?"
    Yes! The [official docker containers](https://hub.docker.com/u/quorumengineering/):
    
    `quorumengineering/quorum:latest`
    `quorumengineering/constellation:latest`
    `quorumengineering/tessera:latest`
    
??? question "Can I mix Quorum nodes with different consensus configuration?"
    Unfortunately, that is not possible. Quorum nodes configured with raft will only be able to work correctly with other nodes running raft consensus. This applies to all other supported consensus algorithms.
