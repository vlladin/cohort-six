# Implementation of FOCIL in Nimbus Cl

Bringing Fork-Choice Enforced Inclusion Lists to the Nimbus Consensus Layer .
(Execution work left as an optional stretch goal)

### Collaborators 

- Tamaghna (@razorclient)
- Akash (@h3lio5)

We will be persuing this work together and divide work as we see fit with blockers and work items



## Motivation

Fork-Choice Enforced Inclusion Lists (FOCIL) represent a protocol-level mechanism to guarantee timely and censorship-resistant transaction inclusion by embedding inclusion constraints directly into Ethereum’s fork-choice rule. 

Basically:
- **FOCIL (EIP-7805)** restores censorship-resistance by letting a 16-validator committee publish inclusion lists (ILs) each slot
- blocks that omit valid IL transactions receive no fork-choice weight.

The Nimbus Cl does not have an implemntation of this proposed changed implemented in its client so we propose to persue the implement for the the aferomentioned during this during EPF cohort

## Project description

We will upstream complete support for FOCIL into Nimbus:

1. **Validator duties** – selection of IL committee, mem-pool sampling via existing Engine-API, IL construction, signature & gossip.
2. **P2P layer** – new global IL topic, gossip validation (slot freshness, equivocation limit, sig checks, ≤8 KiB lists).
3. **Fork-choice enforcement** – store & evaluate ILs, call `engine_newPayloadV5`, give *zero weight* to blocks flagged `INVALID_INCLUSION_LIST`.
4. **Honest-validator logic & CLI flags** – duties scheduling, view-freeze timing, cut-off rules.
5. **Interop test** – local dev-net with Prysm to show Nimbus correctly publishes ILs and rejects censoring blocks
6. **Focil Devnet deployment** - Following the interop testing phase,we move on to deploy nimbus in a devnet to monitor its focil-relaed caopabilities.
7. **Stretch goal (time-permitting)** – collaborate further to allow nimbus EL to produce and consume FOCIL blocks



## Specification 
 
### Nimbus-CL implementation 

#### Validator

The Focil Spec involves changes to the honest Validator Spec,which not limited to 

- New get_inclusion_list method
- Changes to prepare_execution_payload
- New get_inclusion_list_signature

#### Networking

Changes to P2P 

- New Topic inclusion_list
- Changes to batch and gossip validation whoch included signed_inclusion_list validation and propagation

#### Fork-Choice

- is_inclusion_list_satisfied method
- notify_forkchoice_updated method
- Modify PayloadAttributes
- Modified get_forkchoice_store
- New validate_inclusion_lists
- New get_attester_head
- New on_inclusion_list

changes to other function based on inclusion Lists

#### Inclusion List Logic

- New InclusionList Store: Implements a cache
- New method get_inclusion_list_store
- New process_inclusion_list
- New get_inclusion_list_transactions

#### Engine Related changes

- New engine_newPayloadV5
- New engine_getInclusionListV1
- New engine_updatePayloadWithInclusionListV1



## Roadmap

| Phase                           | Weeks | Deliverables                                                                                                                |
| ------------------------------- | ----- | --------------------------------------------------------------------------------------------------------------------------- |
| **1 – Coding**               | 5-10   | Implemtation of FOCIL spec based on Release Pikachu Of consensus spec.                                                       |
| **2 – Local Devnet Testing** | 11-13  | dev-net (Nimbus + Prysm + Geth) showing IL propagation & block rejection via CL logic working.  |
| **3  devnet Deployment**         | 13-25 | Deploy,monitor and assist in Focil related problems           |
| **(Stretch)**                  | 17-25 | If ahead of schedule: integrate Nimbus EL to the fix by iml                     |



## Possible challenges

- Lack of static testcases to validate upon


## Goal of the project

*Mainnet mergeready consensus layer FOCIL in Nimbus*

Success looks like:

1. Nimbus nodes create & gossip ILs when in committee.
2. Nimbus rejects censoring blocks on dev-net .
3. Code passes CI, spec test-vectors, and is merged to `status-im/nimbus-eth2` master prior to Glamsterdam test-net.
4. Stretch: demo full Nimbus EL enforcement on a private dev-net two weeks before cohort end.




### Mentors

@Agnishghosh from the nimbus team 



## Resources

* EIP-7805 – [https://eips.ethereum.org/EIPS/eip-7805](https://eips.ethereum.org/EIPS/eip-7805)
* Consensus release Pikachu – https://github.com/ethereum/consensus-specs/tree/a0f12bf7d8ba7390da67d4fe173241b5471769f8
* HackMD note – [https://hackmd.io/@jihoonsong/ryTt1Fv7le]
* Nimbus Branch - https://github.com/status-im/nimbus-eth2/tree/focil
* Nimbus-web3 branch - TODO(not yet merged)
