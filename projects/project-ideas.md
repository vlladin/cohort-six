# Proposed projects

Project ideas proposed by client teams and mentors. These are projects that fit the EPF scope and would be the most valuable to the team. 

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

By Mark Holt

As zk based proving has matured it has started to use indistry standard VM's for driving its proof set.  These rely on producing and analysing executable code in one of these instruction setc, RISC-V and MIPS being the current candidates used by various market participants.  RISCV seems to have an edge in adoption due as it is has simpler instructions and this has lead to an edge in delivery. 

At presnent most provers for EVM code work by compiling an EVM implementation into the instruction set architecture and then proving the operation of the resultant code.  Given the availibility of various the components necessary to run a stand alone EVM this has been relatively quick to delivery and provides an operating prover.  However this has a number of drawbaccks the most signifigant being that this leads to proving a far larger set of executable instructions than is needed to execute a simple set evm transations in a block, it also introduces a signifigant number of external dependencies which need to be proved.

An alternative is to create an ISA based execution module which is directly provable, this has implications for the tooling of current chains as it means a transition of a core component of the current Ethereum platform, a component which has a large surface area of dependencies.  Another alternative is to add a transpilation process to the existing execution module - so that it provides an executable equivalent which can be used by the proving system.

We are interested in using the erigon execution component with instrtuction level hooks to explore the practicality of the transpilation approach generating an ISA compatable executable object inline with the executions process.  We see this as a POC so it does not necessarily need t be a complete solution at this stage but it does noeed to deal with enough instructions to process a subset of viable smart contract based transactions.

Some simplifying assumtions could for example be:

    * All state is provided in the execution model as constants
    * Precomiles can be treated as idempotent functions which check the input values and return a constant outpout
    * Output value and side effects are pushed to a static constant output function or similar for runtime checking - this may require a target supplied library function

The intention is the produced output can be assembled and executed by an existing zkvm to prove the process. One option that we have considered is to use the Taiko prover framework for this, but part of the project would be to identify prover candidates to test and intergrate to.

### Erigon: L1 Clearing Bridge

By Mark Holt

This project is more R&D than development based.  We curently interact with a number of L2's each of wich has an associated bridging product.  These have a common pattern but largely incompatable operating models typically they rely on the use of logs and proofs from the source chain and rely on a trusted off chain component to initiate activity on the target chain.  They have a variety of trust and confirmation processes which may automated or semi-automated depending upon the implementation details and the cross chain economics.

WIth the pectra hard fork and the deployment of EIP-7702 one of the main points of friction in this process, namely who pays for gas transactions on the target chain is eliviated.  7702 allows payment to be transfered to an onchain paymester rather than the bridge provider.  This makes by directional bridging easier to achieve as payment transfer can become more seamless.  Whilst this still leaves several trust and finality basied inconsistencies it opens the way for a more seamless interactio0n.  It also provided the potential for bridging to be conducted by as a technical operation by node operators without them having to bear the cost of financial costs of bridge transations.

A further extension of the paymaster model could be for an L1 inchain party to act as a trustless clearer of cross chain transactions.  Exactly how this model would work and the implications and practicalities for the parties involved is unclear at this stage.  We think that post Pectra there is an interesting oportunity to explore how this process could work using an L1 CL/EL pair to expore the varios components that would be necessary to facilitate a trustless clearing based bridging model where the L1 acts as a default bridge between L2's.

We see this project as an R&D project which may provide an example contract design and at a streach, given the time scales involved, some form of working model which can be used as the basis for future phases of development.

### Ream Client - A Beam client in Rust: Build Beacon Validator Client

By Kolby ML and Kayden ML

We want to support transitioning Validators from the Beacon Chain to the Beam Chain. We also expect a lot of lessons and tooling required to build a Beacon Validator client will expedite our progress in building a Beam Validator client when the specifications are released later this year. This project would be good for one or multiple people to collaborate. https://github.com/ReamLabs/ream/issues/361

### Ream Client - A Beam client in Rust: Build out P2P testing

By Kolby ML and Kayden ML

We are building out the Beacon Chain’s P2P stack as there is likely to be a lot of overlap with the Beam Chain, this will also allow us to participate in Beacon to Beam transition testnets. We want to add tests that ensure our P2P stack is working as expected and will properly interop with other clients.

### Ream Client - A Beam client in Rust: Implement Beacon API Endpoints

By Kolby ML and Kayden ML

The Beacon API allows us to interact/test our Beacon Chain implementation to verify if everything is working. The Beacon API is also required to be able to run a validator client. It would be valuable for us to implement all the endpoints required to run a validator. Another important task would be to add tests that our implementation is compliant with the specification.

### Ream Client - A Beam client in Rust: Benchmark zkVM performance on Ream's Beacon state transition functions

By Unnawut

We've started benchmarking SP1 and RISC Zero against ream's consensus codebase, giving us early data on performance and how different zkVMs behave when running consensus logic. This lets us track performance regressions as we iterate on the codebase and pinpoint the specific roadblocks and bottlenecks that arise when running consensus specs inside zkVM environments.

We are looking to extend our benchmarking setup to cover other zkVMs like OpenVM, Zisk, Jolt, Brevis Pico, Valida, etc. You'll be implementing benchmarks for these systems, modifying the ream codebase as needed to support the snarkification, while keeping the benchmark framework modular enough to keep track of differences across code iterations, and to easily convert for beam chain specs down the road.

We hope this project provides essential data for making informed zkVM choices in Beam Chain development. For the fellow, it's a chance to get hands-on with both beacon chain consensus implementation details and the rapidly evolving zkVM ecosystem, and prepare yourself to snarkify the upcoming Beam Chain specs.
