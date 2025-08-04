# Pureth [EIP-7919](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7919.md) Implementation for Nimbus-EL

## Motivation 

Ethereum users today rely heavily on trusted JSON-RPC endpoints and third-party indexers for accessing and verifying blockchain data. This centralized model introduces critical issues:
- users can be misled by incorrect or incomplete data if a provider is compromised
- wallets and applications risk privacy loss through request profiling
- developers face increasing infrastructure costs

The Pureth Meta-EIP (EIP-7919) addresses these systemic shortcomings by transitioning core execution-layer structures—blocks, transactions, and receipts—to a verifiable SSZ-based format. This enables correctness and completeness proofs over standard RPC responses, allowing any light client, dApp, or third-party tool to trustlessly verify the data they consume.

These changes lay the foundation for a trust-minimized, privacy-preserving, and cost-efficient data access infrastructure for Ethereum.

### LOG Reform for Pureth

Log Reform is really importance part of Pureth which addresses major inefficiencies in current Logs.Working on LOG Reform would solve following shortcomings :

- Smart contract wallets have to fetch and verify additional transaction receipts within a queried block range because Bloom filters produce false positives, leading to inefficient log retrieval.
- Fetching full receipts because of false positives is impractical for Light clients and mobile devices with resource constraints.
- Eth transfers from smart contracts don't emit logs which leads to problems in tracking transfers.

#### Why important ?

Working on Log Reform is important for following reasons

- Recent implementation from `Geth` has shown that new log indexer returns the results for `eth_getLogs` in milliseconds which other clients with BloomFilter might take few seconds.[ref link](https://xcancel.com/sina_mahmoodi/status/1930565142183387627).
- It will help lightClient and dapps to be more efficient by reducing the unnecessary data fetching.
- Reduce data overhead with compact `SSZ` encoding, enable trusless log retrieval with `Merkle proofs`.
- SSZ encoded logs will be available for all Eth Transfers(e.g., CALL, SELFDESTRUCT) and will be good for tracing.

### Normalized transactions / receipts + Serialization harmonization

Ethereum’s execution layer today relies on RLP/MPT-based data structures and JSON-RPC endpoints that together introduce friction, inefficiency, and trust assumptions at multiple levels:

- Core RPC fields (e.g. from, contractAddress, per-tx gasUsed, logIndex, and the “standard” txHash) are not committed on-chain in a directly accessible form
- RLP encoding and linear Merkle-Patricia Tries force clients to fetch and hash entire blobs (full transaction, full receipt, or whole trie branches) even when they only need to verify a single field or a single log.
- withdrawals are stored in SSZ on the Consensus layer but still hashed via an MPT in the Execution layer, creating duplicate codepaths and two different proof formats for the exact same data.
- most client-to-client and client-to-dApp communication remains JSON over HTTP, which is verbose to encode/decode, difficult to prove, and prevents clients from independently recomputing block hashes or verifying payloads without syncing whole blobs.

## Project description

This project will implement EIP-7919 (Pureth) – and its constituent EIPs – in the Nimbus Execution Layer (Nimbus-EL) client.
Each sub-section below outlines what the proposal aims to achieve for the respective EIP:

###  EIP-7745:Two dimensional log filter data structure

We will start the work by focussing on the implemention of [EIP-7745:Two dimensional log filter data structure](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7745.md) in Nimbus EL :-

- Implement a two-dimensional SSZ index (by block and transaction) for precise, efficient log retrieval.
- Provide SSZ-encoded logs with Merkle proofs, verifiable against block headers.
- Remove the need for Bloom filters.

Once the work on EIP-7745 is complete, we will prioritize other log reform EIPs.

1. EIP-7708: ETH Transfers Emit a Log
   The purpose of this EIP is to make all Eth transfers emit logs because currently smart contracts don't emit logs for Eth transfers and this leads to problem in tracking the transfers sometimes.

2. EIP-7799: System Logs
   Some Eth balance changes are missing even with after the implementation of EIP-7708, So this EIP aims to add a new list to add all Block level logs so that `eth_getLogs` will be able to provide all the history.

### EIP-6404: SSZ transactions and EIP-6466: SSZ receipts

On the parallel front we will start with the [EIP-6404: SSZ Transactions](https://eips.ethereum.org/EIPS/eip-6404) and followed by [EIP-6466: SSZ receipts](https://eips.ethereum.org/EIPS/eip-6466) in Nimbus El.

#### SSZ transactions

- Define the SSZ Transaction container (including all legacy and new fields) using the ProgressiveContainer.

- Migrate Nimbus’s transaction serialization and deserialization from RLP to SSZ.

- Replace the MPT-based transactions_root in the block header with an SSZ Merkle‐tree root over the SSZ‐encoded transaction list.

- Preserve backward compatibility: accept legacy RLP transactions (converting them into SSZ internally).

- Update the Engine API (executePayload, getPayload, etc.) to consume and produce SSZ‐encoded transactions.

#### SSZ receipts

- Define the SSZ Receipt container(s)—BasicReceipt, CreateReceipt, SetCodeReceipt—with explicit from_, gas_used, contract_address, logs, status, and optional authorities.

- Migrate Nimbus’s receipt generation to produce SSZ receipts instead of RLP tuples.

- Compute the block header’s receipts_root as the SSZ Merkle‐tree root over the SSZ receipt list.

- Remove the legacy logsBloom field (via EIP-7668) from headers and receipts.

- Ensure eth_getTransactionReceipt and related JSON-RPC methods return consistent data while sourcing from the new SSZ receipts.


EIP-7807: SSZ execution blocks (End goal of Tamaghna Stretch) 

## Specification

### EIP-7745:Two dimensional log filter data structure

Implementing EIP-7745 in [Nimbus-EL](https://github.com/status-im/nimbus-eth1) involves query the SSZ log filter, generate Merkle proofs, and handle pattern matching.

1.  Define LogIndex, LogEntry and LogFilter structs

```nim
type LogEntry = object
  address: Address
  topics: List[Bytes32, 4]
  data: Bytes
  blockNumber: uint64
  txHash: Bytes32
  logIndex: uint64

type LogFilter = ProgressiveList[LogEntry]
```

2. Reorg Handling

Reorgs may occur, and the log index may have to be partially rewinded to switch to a different chain head. LogIndex should be implemented and tested to handle reorgs.

- Use `ProgressiveList` for dynamic log growth

3. Build Two-Dimensional Log Index

The log index is constructed during receipt processing to enable efficient log retrieval. Each log entry is stored with its associated metadata, and logs are organized in a two-dimensional structure (by block and transaction). Additionally, Logs are stored in per-block Merkle trees, with roots linked to the block’s state or receipt trie for verifiable queries.

```nim
proc buildLogIndex(receipts: seq[Receipt], block: Block): LogFilter =
    for receipt in receipts:
        for log in receipt.logs:
            result.add(LogEntry(
                address: log.address,
                topics: log.topics,
                data: log.data,
                blockNumber: block.number,
                txHash: receipt.txHash,
                logIndex: log.index
            ))

    # Add a dummy log entry to mark the end of the block's logs
    result.add(LogEntry(
        address: Address(zeros(20)),
        topics: @[],
        data: @[],
        blockNumber: block.number,
        txHash: Bytes32(zeros(32)),
        logIndex: high(uint64)
    ))
```

4. Implement FNV-1a Hashing for Filtering

```nim
def get_column_index(log_value_index, log_value):
    log_value_width = MAP_WIDTH // VALUES_PER_MAP                      # constant
    column_hash = fnv_1a(to_binary64(log_value_index) + log_value)     # 64-bit FNV-1A hash
    collision_filter = (column_hash // (2**64 // log_value_width) + column_hash // (2**32 // log_value_width)) % log_value_width
    return log_value_index % VALUES_PER_MAP * log_value_width + collision_filter
```

5. Pattern Matching and Verification

   - Implement `singleProver`, `matchAnyProver`, and `matchSequenceProver` (inspired by Geth PoC)
   - Use set operations for union (matchAnyProver) and intersection (matchSequenceProver)

6. Test with Provided Vectors

   - Implement binary proof parsing (little-endian) and validate against JSON test cases. Integrate with Nimbus’s test suite (tests directory).

7. Test with Kurtosis (Local testnet)
   - Send transactions (e.g., ERC-20, EIP-7708 logs), verify SSZ logs and proofs

### EIP-6404: SSZ Transactions

1. Define the SSZ Transaction container

    ```
    type Transaction* = CompatibleUnion[
      RlpTransaction
    ]

    type RlpTransaction* = CompatibleUnion[
      RlpLegacyReplayableBasicTransaction,
      RlpLegacyReplayableCreateTransaction,
      RlpLegacyBasicTransaction,
      RlpLegacyCreateTransaction,
      RlpAccessListBasicTransaction,
      RlpAccessListCreateTransaction,
      RlpBasicTransaction,
      RlpCreateTransaction,
      RlpBlobTransaction,
      RlpSetCodeTransaction
    ]

    ```

2. Serialization & Deserialization

   -  Replace all RLP-based encodeTransaction and decodeTransaction calls with SSZ serializers.

    - Implement lossless conversion from legacy RLP transactions to SSZ (fromRLP(txRlp): SSZTransaction).

3. Block Header Changes

    In the block builder, compute
    ```
    blockHeader.transactions_root = SSZHashTreeRoot(txList: seq[SSZTransaction])
    ```
    instead of the MPT root.

4. Engine API Updates

    - Update engine_getPayload / engine_executePayload handlers to accept/return SSZ‐encoded transactions in the ExecutionPayload.

    - Maintain backward compatibility by detecting the fork epoch and routing legacy clients through an RLP codepath.

5. Testing

    - Unit tests for SSZ Transaction round-trip encoding/decoding.

    - Consensus tests: verify that the SSZ transactions_root matches reference vectors. 

    - Mempool & devp2p: ensure SSZ txs are hashed and propagated correctly.

### EIP-6466: SSZ Receipts


1. Define the SSZ Receipt union and container types
    


    ```
    type Receipt* = CompatibleUnion[
      BasicReceipt,
      CreateReceipt,
      SetCodeReceipt
    ]

    ## Receipt for regular transactions (no contract create)
    type BasicReceipt* = ProgressiveContainer[active_fields=[1,1,0,1,1]](
      from_: ExecutionAddress,         # signer address
      gas_used:    GasAmount,          # exact gas used
      logs:        ProgressiveList[Log],
      status:      bool                # success/failure
    )

    ## Receipt for contract-creation transactions
    type CreateReceipt* = ProgressiveContainer[active_fields=[1,1,1,1,1]](
      from_:           ExecutionAddress,
      gas_used:        GasAmount,
      contract_address: ExecutionAddress,
      logs:             ProgressiveList[Log],
      status:           bool
    )

    ## Receipt for SetCode (EIP-7702) transactions
    type SetCodeReceipt* = ProgressiveContainer[active_fields=[1,1,0,1,1,1]](
      from_:       ExecutionAddress,
      gas_used:    GasAmount,
      logs:        ProgressiveList[Log],
      status:      bool,
      authorities: ProgressiveList[ExecutionAddress]
    )
    ```
    - Wraps every historical RLP-based transaction type in one SSZ union.

    - Retains the original RLP payload and TransactionType tag in the SSZ bytes, enabling perfect round-trip back to RLP and preserving legacy sig_hash and txHash.

    - Allows SSZ serializers to treat all transactions uniformly while still encoding the original RLP shape when requested.


2. Receipt Generation

    After each transaction execution, collect:

    - from_ (sender address, from signature recovery)

    - gas_used (measure per‐tx gas rather than cumulative)

    - logs (as SSZ Log objects, preserving address/topics/data)

    - contract_address if it was a CREATE tx

    - authorities for SetCode txs

        Instantiate the appropriate SSZ container and append to the block’s receipt list.

3. Block Header Changes

    - Replace the MPT receipts root with an SSZ hash tree root:

        ```
        blockHeader.receipts_root = SSZHashTreeRoot(receiptList: seq[Receipt])

        ```
    - Remove the legacy logsBloom field from both headers and receipts.

4. JSON-RPC & Engine API Updates

5. Backwards Compatibility

    - For blocks prior to the SSZ‐receipts fork, continue serving RLP receipts and computing the MPT root.

    - After fork activation, Nimbus switches to SSZ receipts automatically.

6. Testing & Validation

    Unit Tests:

    - Round-trip SSZ encode/decode for each receipt variant.

    - Verify that from_, gas_used, contract_address, and authorities match expected values.

    Consensus Vectors:

    - Compare receipts_root against reference SSZ roots for sample blocks.

    Integration Tests:

    Run a private devnet with the SSZ receipts+Transactions fork block; ensure eth_getTransactionReceipt returns correct data and legacy clients remain compatible.

    Performance Benchmarks:

    Measure RPC latency and storage impact before vs. after using SSZ.

### EIP-6465: SSZ Withdrawals Root

Aligning withdrawals with SSZ removes MPT complexity and unifies proof formats.

1. SSZ Withdrawals List
```
    Withdrawal(ProgressiveContainer[active_fields=[1, 1, 1, 1]]):
    index: WithdrawalIndex
    validator_index: ValidatorIndex
    address: ExecutionAddress
    amount: Gwei
```
2. Block Header & Payload Changes

4. Codepath Cleanup
    - Remove MPT‐related withdrawal hashing and merkle proof code.

    - Ensure CL‐EL test compatibility: Nimbus-CL proofs of withdrawals now match Nimbus-EL’s SSZ root.
    
4. Unit tests for SSZ Withdrawals list.

    - Cross‐client sync test: process a block in Nimbus-EL and verify withdrawals_root in Nimbus-CL. 

### EIP-7807: SSZ Execution Blocks

1. Define the SSZ ExecutionBlockHeader

    ```
    class ExecutionBlockHeader(
        ProgressiveContainer[active_fields=[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]]):
        parent_hash: Root
        miner: ExecutionAddress
        state_root: Bytes32
        transactions_root: Root  # EIP-6404 transactions.hash_tree_root()
        receipts_root: Root  # EIP-6466 receipts.hash_tree_root()
        number: uint64
        gas_limits: GasAmounts
        gas_used: GasAmounts
        timestamp: uint64
        extra_data: ByteList[MAX_EXTRA_DATA_BYTES]
        mix_hash: Bytes32
        base_fees_per_gas: BlobFeesPerGas
        withdrawals_root: Root  # EIP-6465 withdrawals.hash_tree_root()
        excess_gas: GasAmounts
        parent_beacon_block_root: Root
        requests_hash: Bytes32  # EIP-6110 execution_requests.hash_tree_root()
        system_logs_root: Root  # EIP-7799 system_logs.hash_tree_root()
    ```
    
2. Block Import/Export

    - Migrate Nimbus-EL’s block serialization to SSZ for both headers and full payloads.

    - Remove encodings in engine_getPayload / engine_executePayload and transition to ssz

    - Update consensus‐to‐execution and execution‐to‐consensus flows to use SSZ objects directly.

3. Testing & Benchmarking

    - End-to-end tests: import/export SSZ blocks, verify block hashes in CL.

    - Performance benchmarks: measure payload serialization/deserialization speed and binary API throughput(maybe for binary api).


### STEEL Effort 

These ssz changes need a stable static testing suite to be tested against and validated upon,Hence there will also be a set of tests developed to test these impl upon initially for the part that consistutes the SSZ changes and lated for the logs parts of the EIP.

## Roadmap

The roadmap and implemention timeline can be broken as follows:-

The following is the timeline by Vineet:

#### Phase 1: 2 weeks

- Finalize the technical implementation details.
- Nimbus-EL architechture understanding.
- Basic Understanding of Nim language

#### Phase2: 7-8 weeks

- EIP-7745: Two-Dimensional Log Filter Data Structure.
  - Replace bloom filter with 2D log index
  - Test new logs against JSON RPCs
  - Test with Kurtosis

#### Phase3: 6-7 weeks

- For other Log reform EIPs

  - EIP-7708: ETH Transfers Emit a Log
    - Effort estimate
      - Should take 5 weeks (approx) to implement
      - Emit SSZ logs for opcodes like SELFDESTRUCT or CALL transfers.
      - Simulate tests for Eth transfer in smart contract wallet
    - Additional Transfers which should be logged and tested
      - Deployment of a new contract with a starting balance
      - Fee payments (both for the burn and also for the priority fee)

  - EIP-7799: System Logs
    - Effort estimate
      - Shold take same effort as EIP-7708

The following is by tamaghna:

Phase 1: 5 weeks
- Implement EIP-6404: SSZ Transactions

  - Define SSZ transaction container using CompatibleUnion and ProgressiveContainer.
  - Add support for decoding legacy RLP transactions into SSZ internally.
  - Compute SSZ-based transactions_root in block headers.
  - Modify Engine API (getPayload, executePayload, etc.) to use SSZ transactions.
  - Create round-trip unit tests and consensus vectors.

- Implement EIP-6466: SSZ Receipts

  - Define Receipt, BasicReceipt, CreateReceipt, SetCodeReceipt using SSZ containers.
  - Integrate SSZ receipts in transaction execution output.
  - Replace logsBloom and compute receipts_root as SSZ tree root.
  - Modify JSON-RPC methods to source from SSZ receipts.

- Write test coverage in eels (SSZ reference tests):

  - Add encode/decode tests for all transaction and receipt types.
  - Build cross-compatibility test harness between eels and Nimbus-EL.

Phase 2: 2 weeks

- Implement EIP-6465: SSZ Withdrawals Root

  - Define Withdrawal container and compute withdrawals_root using SSZ.
  - Remove MPT-based withdrawal path and harmonize with CL withdrawals.

- Implement EIP-7807: SSZ Execution Blocks

  - Define full SSZ block header container.
  - Use SSZ objects for block import/export, engine_getPayload, engine_executePayload.
  - Integrate with Vineet’s log roots (e.g., system_logs_root).

- Add equivalent SSZ implementations and tests in eels.

Phase 3: 5 weeks
- Final integration and end-to-end testing:

  - Launch local devnet with Vineet using Kurtosis.
  - Interop: verify Nimbus-EL’s SSZ transaction, receipt, withdrawal, and block roots against CL and eels.
  - Validate full SSZ-based execution payloads.

- Hive test integration:

  - Ensure SSZ changes pass Hive tests.
  - Compare executionPayloadRoot and executionBlockHeaderRoot.

- Refine and document legacy compatibility:

  - Ensure clients consuming RLP txs/receipts still interoperate.


## Possible challenges

- Learning Nim and Nimbus Architecture
- Building a two-dimensional SSZ-based log index (ProgressiveList, EIP-7916) with block and transaction dimensions.
- Validating the implementation against test vectors (proof_test_vectors) and ensuring compatibility with Kurtosis testnet queries is time-consuming.
- Maintaining backward compatibility with existing JSON-RPC clients while switching over to SSZ encoding internally.
- Coordinating structural changes to Nimbus block headers and Engine API without introducing regressions in CL–EL communication.
- Deep testing is required for root hashes, proof generation, and cross-client SSZ consistency
- creating eels tests a parellel harness to consume-rlp and consume-engine
- Debugging potential mismatches between eels, Nimbus-EL, and testnet payloads during SSZ integration.


## ToDo 

Some additional points need to be studied and updated in the proposal as the implementation progresses.

- Evaluate Querying efficiency of the linearized index structure.
- Storage growth with big/old blocks and if client receives many eth_getLogs queries.
- How much performance overhead it adds with SSZ
- Finalize SSZ containers for transactions and receipts.
- Implement SSZ serialization/deserialization for Transaction, Receipt.
- Update block header logic to compute SSZ roots.
- Modify Engine API handlers for SSZ types.
- Write eels test harness and SSZ round-trip tests.
- Migrate execution payloads and headers to full SSZ form.
- Align withdrawal roots with CL via EIP-6465.
- Generate and validate test vectors for Hive and devnet.


## Collaborators

Vineet and @samyxandy (Tamaghna)

- Vineet :- Focussing on Log Reform EIP-7745: Two dimensional log filter data structure
- Tamaghana :- Focussing on EIP-6404: SSZ transactions

We will coordinate and work parallely on separate EIPs for implementation, once core EIPs (7745 and 6404) are done , we will work on other EIPs for Log Reform and Remove old tx types categories in Pureth.



#### Goal of the project

- Trustless Log Retrieval

  - Support decentralization goals of Ethereum by reduding reliance on trusted JSON RPC providers.
  - Providing ssz encoded logs with Merkle proofs will enable light clients, dapps and smart contract wallets to verify correcness of logs against block headers.

- Eliminate Bloom Filter Inefficiencies

  - Replace bloom filter with 2D log Log index will reduce unnecessary data fetching due to false positives nature of bloom filters

- Improve scalability and efficiency of log queries `eth_getLogs`

- SSZ Part 

  - Replace legacy RLP/MPT structures in Nimbus-EL with efficient, verifiable SSZ containers for transactions, receipts, and blocks.
  - Enable execution-layer correctness proofs that align with consensus-layer SSZ structures.
  - Lay the groundwork for decentralized, trustless clients that can verify execution payloads and logs without syncing entire block bodies.
  - Harmonize all Nimbus data formats (txs, receipts, withdrawals, blocks) under a single SSZ schema for future-proofing.

### Mentors

[Etan](https://github.com/etan-status) is the mentor for Pureth EIP-7919 implementation, [Advaita](https://github.com/advaita-saha)  will be mentor for Nimbus-EL specific questions (receipts processing / logs / reorgs).

### Resources

| Resource                                    | link                                                                   |
| ------------------------------------------- | ---------------------------------------------------------------------- |
| Pureth EIP-7919                             | https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7919.md          |
| EIP-7745 : 2D log filter data structure     | https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7745.md          |
| EIP-7745 details by Zsolt                   | https://github.com/zsfelfoldi/eip-7745/blob/main/proof_verification.md |
| Performance improvement of new Log indexer  | https://xcancel.com/sina_mahmoodi/status/1930565142183387627           | 
