# Glossary

Standard terminology for The Institutional Crypto Research Handbook.

Definitions here are intentionally short and operational. They are meant to stabilize language across documents, not replace the full chapter discussions.

## A

`ABI`
: Application Binary Interface. The standardized encoding and function-call interface that lets external applications and contracts interact with a smart contract.

`Active Addresses`
: A metric counting addresses that participated in transactions during a period. It is a network-activity proxy, not a direct user-count metric.

`AMM`
: Automated Market Maker. A smart-contract market structure where traders swap against pooled liquidity instead of an order book.

## B

`Beacon Chain`
: Ethereum’s consensus-layer chain introduced before the Merge and later used as the PoS consensus backbone for Ethereum mainnet.

`Bridge`
: A system that moves assets or messages across chains under a specific trust and validation model.

## C

`Case Study`
: A historical event analysis that applies the handbook’s research method to a real market or protocol episode.

`Chain Reorganization`
: A change in the local accepted best chain when a node switches from one valid chain tip to another with more cumulative work or stronger fork-choice weight.

`Coin Days Destroyed`
: An age-weighted activity metric that increases when older coins move.

`Consensus`
: The rule set and process by which a distributed network agrees on valid state history.

`Counter Evidence`
: Evidence that weakens, contradicts, or competes with a preferred explanation.

`Cumulative Chainwork`
: Bitcoin’s measure of total expected work represented by a chain, used to compare competing valid chains.

## D

`DeFi`
: Decentralized Finance. Onchain financial systems built from smart contracts rather than centralized intermediaries.

`Delegation`
: The assignment of voting power from one token holder to another address in governance systems.

`Difficulty Adjustment`
: Bitcoin’s periodic recalibration of mining difficulty to target roughly 10-minute block intervals.

`Dormancy`
: A metric capturing how long moved coins remained inactive before spending.

`Double SHA-256`
: Bitcoin’s use of SHA-256 applied twice in core block-header hashing and related contexts.

`Draft`
: A document status used before full review and quality-gate completion.

`DEX`
: Decentralized Exchange. An onchain trading protocol that executes asset swaps without centralized custody.

## E

`ECL`
: Evidence Confidence Level. A label expressing how strongly a claim is supported by the available evidence.

`EIP`
: Ethereum Improvement Proposal. A formal standard or process document for Ethereum changes.

`ERC-20`
: The core fungible-token interface standard on Ethereum.

`ERC-721`
: The core non-fungible-token interface standard on Ethereum.

`ERC-1155`
: A multi-token Ethereum standard that can represent fungible, non-fungible, and semi-fungible assets in one contract.

`Exchange Flows`
: Metrics tracking transfers into and out of labeled exchange entities.

`Exchange Reserve`
: The estimated balance of an asset held by exchange-labeled addresses or entities.

## F

`Finality`
: The degree to which a block or state transition is considered irreversible under a given system’s rules and assumptions.

`Fork`
: A divergence in blockchain state history or rule set. The term can refer to temporary chain splits, software rule changes, or social/protocol governance splits.

## G

`Gas`
: Ethereum’s metering unit for computation and storage access during transaction execution.

`Governance Token`
: A token used to allocate or delegate decision power over protocol parameters, upgrades, or treasury actions.

## H

`Hashcash`
: A proof-of-work anti-spam construction that historically influenced Bitcoin’s PoW model.

`Health Factor`
: Aave-style solvency buffer metric comparing collateral value and liquidation thresholds against borrowed value.

`Hypothesis`
: A testable explanation built from observations and evidence, held provisionally rather than treated as fact.

## I

`Impermanent Loss`
: The opportunity-cost difference between providing AMM liquidity and simply holding the underlying assets.

`Institutional Thinking`
: The handbook’s discipline of separating observation, evidence, interpretation, risk, and alternative explanations before forming conclusions.

## L

`Layer 2`
: A scaling system built on top of a base chain that moves execution or batching off the base layer while retaining some relationship to base-layer security or settlement.

`Lightning Network`
: A Bitcoin payment-channel network designed to support faster and cheaper offchain transactions with onchain settlement guarantees.

`Liquidity Pool`
: A shared asset reserve used by AMMs or other DeFi protocols for swaps, lending, or related actions.

`Liquidation`
: The forced partial or full resolution of an undercollateralized borrowing position according to protocol rules.

## M

`Mempool`
: A node’s local pool of valid but unconfirmed transactions awaiting inclusion in a block.

`MEV`
: Maximal Extractable Value. The value obtainable from transaction inclusion, exclusion, or ordering beyond standard block rewards and fees.

`Merkle Root`
: The root hash of a Merkle tree summarizing a set of items such as transactions.

`MVRV`
: Market Value to Realized Value ratio. A valuation framework comparing current market capitalization to realized capitalization.

## N

`nBits`
: Bitcoin’s compact encoding format for the proof-of-work target inside the block header.

`NUPL`
: Net Unrealized Profit/Loss. A metric estimating aggregate embedded unrealized profit relative to market capitalization.

`Nonce`
: A mutable field miners change while searching for a valid Bitcoin block hash below the current target.

## O

`Observation`
: A measurable fact stated without interpretation.

`Onchain Metrics`
: Quantitative indicators derived from blockchain data and related entity models.

`Oracle`
: A system that brings external data into smart-contract environments or coordinates offchain information for onchain use.

## P

`PoS`
: Proof-of-Stake. A consensus mechanism where validators explicitly bond capital and face penalties for rule violations or certain failures.

`PoW`
: Proof-of-Work. A consensus mechanism in which block production depends on computational work meeting a cryptographic difficulty target.

`Primary Source`
: The highest-priority evidence class in this handbook, including protocol documentation, standards, code, whitepapers, and original datasets where applicable.

`Proxy Pattern`
: A contract architecture where one address delegates logic execution to another contract, commonly used for upgradeability.

## Q

`Quality Gate`
: The final checklist a document must pass before remaining in reviewed status.

## R

`Realized Cap`
: A valuation metric that prices each coin at the value when it last moved onchain instead of using current spot price.

`Research Brief`
: The scoped document specification that defines the research question, prerequisites, related topics, required sections, and out-of-scope boundaries.

`Restaking`
: The reuse of staked security, typically Ethereum-related stake or liquid staking exposure, to secure additional services beyond base consensus.

`Reviewed`
: A document status indicating technical, evidence, editorial, and institutional review are complete.

## S

`SegWit`
: Bitcoin’s Segregated Witness upgrade that changed transaction serialization and fixed malleability issues relevant to scaling and second-layer systems.

`Smart Contract`
: A program with persistent state that executes on Ethereum or a similar blockchain environment according to consensus rules.

`Smart Money`
: A label-based heuristic for apparently sophisticated or historically successful wallets. It is an analytic convenience, not a consensus fact.

`SOPR`
: Spent Output Profit Ratio. A realized profit-and-loss metric for spent coins or outputs.

`Stablecoin`
: A tokenized asset or system designed to maintain a relatively stable reference value, usually through reserves, collateral, redemption, or algorithmic mechanisms.

`Staking`
: Bonding capital to participate in proof-of-stake security and earn rewards while accepting penalties and slashing risk.

`Taproot`
: A Bitcoin upgrade that introduced Schnorr signatures and improved spending flexibility and privacy properties for some transaction structures.

`Technical Review`
: The review stage that checks terminology, behavior, formulas, and implementation or protocol accuracy.

`Thesis`
: A structured research view connecting observation, evidence, assumptions, and probability into a decision-relevant conclusion.

`Tokenomics`
: The economic design of a token system, including issuance, utility, governance, incentive flows, and downside absorption.

`TVL`
: Total Value Locked. The aggregate value held in DeFi protocols or systems, often useful but easily overstated or misunderstood.

## U

`UTXO`
: Unspent Transaction Output. The spendable unit in Bitcoin’s transaction model.

`Utility Token`
: A token whose protocol role includes operational functions such as payment, staking, access, coordination, or backstop mechanisms beyond simple passive holding.

## V

`Validator`
: A proof-of-stake participant responsible for proposing or attesting to blocks and subject to rewards and penalties.

## W

`Wallet`
: A tool for key management and transaction authorization. In practice this may mean software, hardware, or coordinated account infrastructure.

`Whale Activity`
: Large-holder or large-balance activity inferred from address, entity, or transfer-size behavior. It should not be treated as a perfect beneficial-ownership measure.

`Wrapped Asset`
: A tokenized representation of another asset held, locked, bridged, or custodied elsewhere.

## Y

`Yield Farming`
: The layering of incentive rewards on top of base DeFi positions to attract liquidity, deposits, or other target behaviors.
