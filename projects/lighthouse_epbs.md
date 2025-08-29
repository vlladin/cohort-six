# EPBS: EIP-7732 – Implementation Proposal

Enshrined Proposer Builder Separation (epbs) decouples execution payloads from consensus blocks.  The consensus block will undergo a lightweight validation before the attestation deadline whereas the execution payload will be processed independently.

## Motivation
 - Execution is removed from the consensus hot path
 - Execution could have up to 9 seconds for block validation, enabling a significantly increased gas limit
 - Notable increase in blob scaling
 - In-protocol block auctions allow proposers to gain exposure to MEV rewards without relying on external relays

## Project Description 
We will implement [EIP-7732](https://eips.ethereum.org/EIPS/eip-7732) in Lighthouse as it only contains CL modifications.

Scope:
- **CL Block/body changes**: Include signed execution bids and payload attestations in addition to remove the execution payload
- **Validator duties**: Implement PTC message production/aggregation and include aggregates in the next block
- **Networking**: Add gossip support for new libp2p topics
- **Fork-choice**: Traversal and tip selection consistent with the [consensus spec](https://ethereum.github.io/consensus-specs/specs/_features/eip7732/fork-choice/)


## Specification
1. `BeaconBlockBody` is gossiped as part of a `BeaconBlock` across the network.  It used to have an `execution_payload`, which contained the full transaction list.  Instead, it will now have a lightweight payload header signed by the builder, plus payload timeliness attestations from the previous slot.
```rust
 struct BeaconBlockBody {
    // remove execution_payload
     
    // new
    signed_execution_bid: SignedExecutionBid,   // builder’s bid + EL block hash
    payload_attestations: Vec<PayloadAttestation>,  // PTC aggregations from previous slot
     
   // existing fields below
}
```

2. The `BeaconState` will be extended to track fields signaling whether a payload was received by a builder and also track the status of builder to proposer payments:
```rust
pub struct BeaconState {
    // indicates if payload was locally received per slot
    pub execution_payload_availability: BitVector<SLOTS_PER_HISTORICAL_ROOT>,

    // gather the pending builder to proposer payments for each slot in the current epoch
    pub builder_pending_payments: Vector<BuilderPendingPayment, { 2 * SLOTS_PER_EPOCH }>,
    
    // track the pending builder to proposer payments from previous epochs that are ready to be processed by the EL
    pub builder_pending_withdrawals:
        List<BuilderPendingWithdrawal, BUILDER_PENDING_WITHDRAWALS_LIMIT>,
    
    // hash of the last fully executed payload
    pub latest_block_hash: Hash256,
    
    // slot at which the last payload was executed
    pub latest_full_slot: Slot,
    
    // withdrawals root processed in CL for the latest slot
    pub latest_withdrawals_root: Hash256,

   // existing fields below
}
```

3. We will add a `builder` duty in-protocol that is responsible for constructing an execution bid committing to releasing an `ExecutionPayload` later in the block if the proposer includes the bid in the `BeaconBlockBody`:
```rust
pub fn builder_duty(slot: Slot) {
    let bid = build_bid(slot);                    // commit to header + value
    gossip("execution_payload_header", bid);
    if bid_included_in_beacon_block(slot) {
        let payload = build_payload_envelope(bid);    // matches committed header
        gossip("execution_payload", payload);         // before PTC deadline
    }
}
```

4. A Payload Timeliness Committee (PTC) will consist of 512 validators per slot and attest to whether a builder revealed their payload timely within the slot.  The `PayloadAttestationMessage` will be gossiped from committee members and aggregated by the proposer as to include in the `BeaconBlockBody`:
```rust
pub struct PayloadAttestationData {
    // the HTR of the beacon block seen for the assigned slot
    pub beacon_block_root: Hash256,
    // the committee member's assigned slot
    pub slot: Slot,
    // indicates the execution payload was received by this validator
    pub payload_present: bool,
    // indicates the blob data was received by this validator
    pub blob_data_available: bool,
}
```

5. The block proposal flow will be modified such that the proposer will receive bids from builders, select the highest bid, and then broadcast a  block including the bid and aggregated `PayloadAttestation`:
```rust
fn propose_block() {
    // select the best bid for the slot
    let bid = select_best_bid(slot);
    let body = BeaconBlockBody {
        // existing fields
        signed_execution_bid: {block_hash, value, ...},
        payload_attestations: collect_previous_slots_ptc(),
    };
    
    sign_and_gossip_block(slot, body);
}
```

6.  Then, when receiving a new block gossiped by the proposer, the validator will process withdrawals and the `ExecutionPayloadHeader` without having to wait for the EL to process transactions anymore before attesting to block validity:
```rust
fn process_block(state: BeaconState, block: BeaconBlock) {
    process_withdrawals(state);
    process_execution_payload_header(state, block); // see below
}
```
```rust
fn process_execution_payload_header(state: BeaconState, block: BeaconBlock) {
    // Verify the header signature
    let signed_header = block.body.signed_execution_payload_header;
    assert!(
        verify_execution_payload_header_signature(state, signed_header)
    );

    // Check that the builder is active, non-slashed, and has funds to cover the bid
    assert!(
        state.balances[builder_index]
            >= amount + pending_payments + pending_withdrawals + MIN_ACTIVATION_BALANCE
    );

    // Verify that the bid is for the current slot
    assert!(header.slot == block.slot);

    // Verify that the bid is for the right parent block
    assert!(header.parent_block_hash == state.latest_block_hash);
    assert!(header.parent_block_root == block.parent_root);

    // Record the pending payment
    state.builder_pending_payments[SLOTS_PER_EPOCH + header.slot % SLOTS_PER_EPOCH] =
        pending_payment;

    // Cache the signed execution payload header
    state.latest_execution_payload_header = header;
}
```

7. Before the PTC deadline during a slot, the builder will reveal and gossip the`ExecutionPayload` inside a `SignedExecutionPayloadEnvelope` including transactions, which will be passed from the the CL to the EL like today for validation.  The validator will now execute `process_execution_payload` upon receiving the `SignedExecutionPayloadEnvelope`:
```rust
fn process_execution_payload(
    state: BeaconState,
    signed_envelope: SignedExecutionPayloadEnvelope,
    execution_engine: ExecutionEngine,
) {
    // Verify the builder's signature
    // check that the payload corresponds to the BeaconBlock for this slot

    // Pass payload to the EL for validation just like today
    assert!(execution_engine.verify_and_notify_new_payload(
        NewPayloadRequest {
            execution_payload: signed_envelope.message.payload,
            versioned_hashes: versioned_hashes,
            parent_beacon_block_root: state.latest_block_header.parent_root,
            execution_requests: requests,
        }
    ));
}
```

8. Fork-choice will incorperate payload timeliness into `head` selection by tracking whether a block's payload is `PENDING`, `EMPTY`, or `FULL`:
```rust
struct ForkChoiceNode {
    root: Root,
    payload_status: PayloadStatus, // PENDING | EMPTY | FULL
}
```
At each step, fork choice will make a payload decision and move to the selected `FULL` or `EMPTY` child. For the slot before the tip, both `FULL` and `EMPTY` have weight 0, so a payload tiebreaker uses PTC signaling. The `FULL` child is only selectable if the payload body is locally available.

9. New libp2p gossip topics:
    - `execution_payload_header` will be submitted by the builder and contain the bid
    - `execution_payload` will be sent by the builder to reveal the payload before the PTC deadline
    - `payload_attestation_message` will be gossiped by the PTC committee members


With these updates, Lighthouse will properly decouple the `ExecutionPayload` from the `BeaconBlock`, which is a clean separation and enables massive L1 scaling.  

## Roadmap (Weeks 6–21)

### Phase 1: Core Data Structures (Weeks 6–9)
**Deliverables:**
- SSZ serialization for all new containers, i.e. `SignedExecutionPayloadEnvelope`
- Extend `BeaconBlockBody` and `BeaconState` with EPBS fields
- Register libp2p topics with stub handlers

**Success Criteria:** All new containers serialize/deserialize correctly, and topic stubs are invoked in networking tests.

### Phase 2: Builder Flow (Weeks 10–13)
**Deliverables:**
- Add builder duty per [consensus specs](https://ethereum.github.io/consensus-specs/specs/_features/eip7732/builder/)
- Proposer bid selection and inclusion of `SignedExecutionBid` in `BeaconBlockBody`
- Record builder to proposer pending payment in `BeaconState` and roll into `builder_pending_withdrawals` at epoch boundary
- Process `SignedExecutionPayloadEnvelope` or withheld message with slot timing enforcement

**Success Criteria:** Valid bids are accepted, invalid ones rejected, and payloads are marked FULL or EMPTY as expected.

### Phase 3: PTC and Fork-Choice (Weeks 14–19)
**Deliverables:**
- Implement PTC committee, gossip handling, and proposer aggregation into the next block
- Extend fork choice nodes with `payload_status` (`PENDING`, `FULL`, `EMPTY`)
- Implement fork-choice rules per [consensus specs](https://ethereum.github.io/consensus-specs/specs/_features/eip7732/fork-choice/)

**Success Criteria:** Fork choice selects the correct head across FULL and EMPTY cases.

### Phase 4: Testing and Benchmarking (Weeks 20–21)
**Deliverables:**
- Kurtosis devnet scenarios to emulate different builder behaviors, i.e. valid or withheld payload
- Fix any race conditions or consensus mismatches found in testing

**Success Criteria:** Devnet finalizes blocks correctly and benchmarks pass within target thresholds.



## Possible challenges
- Maintaining backward compatibility pre-fork boundary
    - Fork-choice head selection with (root, payload_status) nodes adds complexity
    - Pipelined slot structure requires PTC validation and handling of new libp2p topics
    - CL managing builder payments is non-trivial
- Consensus spec changes (FOCIL compatibility, dual deadline PTC, payment flow changes)


## Goal of the project
The overarching goal is to implement EIP-7732 in Lighthouse so that execution payloads are decoupled from consensus blocks, enabling payload validation after the attestation deadline.  The builder’s reveal and withhold paths must work end-to-end so that nodes converge on the same head, finalize without unnecessary reorgs, and maintain stability under all payload availability scenarios. We want a correct and maintainable implementation that integrates cleanly with existing Lighthouse components and is ready for testing on devnets ahead of the Glamsterdam hardfork.

## Collaborators

### Fellow 
[Shane](https://github.com/shane-moore)

### Mentor & Contributor
Lighthouse - [Mark](https://github.com/ethdreamer)

## Resources
[Lighthouse Github](https://github.com/sigp/lighthouse)
[EIP-7732](https://eips.ethereum.org/EIPS/eip-7732)
[Consensus Specs](https://ethereum.github.io/consensus-specs/specs/_features/eip7732/beacon-chain/)
