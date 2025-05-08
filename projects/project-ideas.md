# Proposed projects

Project ideas proposed by client teams and mentors. These are projects that fit the EPF scsope and would be the most valuable to the team. 

If you are interested in the project, take your time to learn about it, the team it's coming from and related area of the protocol. Reach out to client organizers if you have any questions or need connection to a mentor.

## Previous cohorts

In project ideas from previous cohorts, you might find some up to date ideas which haven't been solved yet.

- [Project ideas in the fifth cohort](https://github.com/eth-protocol-fellows/cohort-five/blob/master/projects/project-ideas.md), [Projects executed](https://github.com/eth-protocol-fellows/cohort-five/blob/master/projects/)
- [Project ideas in the fourth cohort](https://github.com/eth-protocol-fellows/cohort-four/blob/master/projects/project-ideas.md), [Projects executed](https://github.com/eth-protocol-fellows/cohort-four/blob/master/projects/)
- [Project ideas in the third cohort](https://github.com/eth-protocol-fellows/cohort-three/blob/master/projects/project-ideas.md), [Projects executed](https://github.com/eth-protocol-fellows/cohort-three/blob/master/projects/)
- [Project ideas in the second cohort](https://github.com/ethereum-cdap/cohort-zero/issues?q=is%3Aopen+is%3Aissue+label%3A%22help+wanted%22)
- [Project ideas in the first cohort](https://github.com/ethereum-cdap/cohort-one/issues?q=is%3Aissue+Project+idea)

## Ideas proposed by core devs 

### PandaOps tooling wishlist

By Pari

Ongoing wishlist of ideas for tools that help with development and testing of the protocol. 

https://github.com/ethpandaops/tooling-wishlist

### RIG Opened Problems

By Barnabé Monnot

Explore Robus Incentives Group Opened Problems. Most relevant for EPF are tagged https://efdn.notion.site/ROPs-RIG-Open-Problems-c11382c213f949a4b89927ef4e962adf

### Ephemery testnet

By Mario Havel

Contribute to integrations of public ephemeral testnet defined in eip-6916. Missing implementations in clients, deployments, devops tooling and client testing.
https://github.com/ephemery-testnet/ephemery-resources/blob/master/client-implementations.md
https://ephemery.dev/

### Lodestar: Backfill

By Lodestar

Lodestar's TypeScript-based Ethereum consensus client currently does not have a backfill mechanism to retain blocks/blobs required, conduct backfill syncing and eventually serve custody column groups when moving towards PeerDAS. This feature will expose a candidate to various parts of the Ethereum consensus protocol, including new chain features whilst producing a high priority need for the implementation. For more information, please check: https://github.com/ChainSafe/lodestar/issues/7753

### Lodestar: ERA file support

By Lodestar

Lodestar's TypeScript-based Ethereum consensus client is seeking a fellow interested in helping to produce and use Era files for historical state regeneration as an alternative to backfill syncing and minimizing unnecessary data stored on nodes. For more information, please check: https://github.com/ChainSafe/lodestar/issues/7048

### Besu: discv5

Besu - implementing discv5 support https://github.com/hyperledger/besu/issues/4089

By Besu

### Besu: eth/70

Besu - implementing eth/70 - https://github.com/hyperledger/besu/issues/8288

By Besu

### Besu: ephemery 

Besu - ephemery - basic support is implemented but there are still some remaining tasks - https://github.com/hyperledger/besu/issues?q=is%3Aissue%20state%3Aopen%20ephemery

By Besu

### Teku / Nimbus / Lodestar / Grandine: Implement EIP-7917 - Deterministic proposer lookahead  

[EIP-7917](https://eips.ethereum.org/EIPS/eip-7917) is a mechanism to pre-calculate and store a deterministic proposer lookahead in the beacon state at the start of every epoch.
Implementing this EIP in one or more of the above mentioned clients is a low hanging-fruit as well as [super valuable](https://hackmd.io/@linoscope/eip-7917-from-preconf-protocol), and may serve as an excellent starting point for a fellow to get involved with core protocol work.

By Justin Drake & Lin Oshitani


### Erigon: FOCIL & Alternatives 

By Mark Holt

[EIP-7805](https://eips.ethereum.org/EIPS/eip-7805) FOCIL implements a robust mechanism to preserve Ethereum’s censorship resistance properties by guaranteeing timely transaction inclusion.  We are starting to implement this for both Erigon's CL and EL components and would like to catch up with  other [client's progress](https://meetfocil.eth.limo/)  so we can become an active participant in the public devnet process once it commences.

We are also interested in exploring alternatives for censorship resistance such as [BRIAD](https://ethresear.ch/t/censorship-insurance-markets-for-braid/20288) so we envision this project as a combination of background research and implementation.

### Erigon: Parallel Execution

By Mark Holt

We have recently completed development of a parallel execution for Erigon's EL based on [Block STM](https://arxiv.org/abs/2203.06871).  This is an adaption of previous work by Polygon for the [POS Chain](https://polygonscan.com/).  Our adaption is intended to work for all chains supported by Erigon.  We expect to release this as an experimental feature on Erigon 3.1.

The POS chain contains header hints to help the parallel execution process by including dependency data in its block header. [EIP-7928](https://github.com/ethereum/EIPs/blob/954213305aa55eeda61a3fe3fe247011c038a9d4/EIPS/eip-7928.md) provides a similar but more extensive project.

From the current testing of our execution process we are aware that execution throughput in terms of gas is an optimization problem which trades off parallelism (number of active threads), disk/cache read time and failures due to cross dependencies.  When optimising this at the network level (which is the basis of 7928) there is an additional factor which is the size and associated storage and transmission costs of passing the hint data.  (Polygon uses a minimal dependency set).

As we have a working parallel implementation we intend this project to explore the options for optimizing this process in the context of Ethereum and to contribute to the practical development of EIP-7928 as a standards proposal and to parallelism across all network clients in general.   

### Erigon: Parallel Embedded Indexing

By Mark Holt

With erigon 3.1/3.2 we are developing the functionality of the data storage technology at the core of the product to facilitate the distribution of out of protocol data sets.  We are also developing a streaming mechanism which allows historical versions of that data to be generated by 3rd party code.

We believe that this will add significantly to the efficiency of the services that surround a typical blockchain client by allowing the data these services rely on to be generated in parallel using the spare CPU cycles available as blocks are processed in parallel.

In order to make this spare capacity available we intent to initiate a project to standardize the map/reduce framework we already use for the otter scan api and for custom tracing so that it can be used as a plug in framework for external developers.

It is likely this this framework will work an an analogous fashion to existing cloud function models such as [Google Cloud Functions](https://github.com/GoogleCloudPlatform/functions-framework).  The work on this project would involve reviewing the available frameworks and working with erigon team member to come up with a POC of such a framework.

Examples of indexing tasks which could form part of this POC include DEX market data, e.g. embedded EBBO as a price benchmark and the project below which is aimed at generating provable RISCV output inline with the current execution pipeline. 

### Erigon: RISCV Executable Proof Sourcing

### Erigon: L1 Clearing Bridge

