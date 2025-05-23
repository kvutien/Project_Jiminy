# Specs of Jiminy Cricket AI Mentor

## 0 Executive Summary

"Jiminy Cricket" is a key character in _The Adventures of Pinocchio_ (1883) by Carlo Collodi and later 
popularized by Disney's 1940 animated adaptation. In the original novel, he is simply known as the Talking 
Cricket ( _il Grillo Parlante_ ), while Disney named him "Jiminy Cricket" and gave him a more prominent role.

Jiminy Cricket plays a crucial role in Pinocchio’s journey from a reckless puppet to a responsible and
truthful boy, embodying the theme of a mentor in personal growth.

Our application Jiminy Cricket uses Artificial Intellligence (AI) to act as a patient and always available guide 
of a human student in his learning path.

### 0.1 Composition of the Learning Tool

**Jimini Cricket** is a web app composed of 4 panes:

1. One pane hosts the mentor Jiminy Cricket. It guides the student by questions, rates the student's 
    answer, keeps track of the progression and of the weak points of the student. It relaunches the learning 
    in another direction once the student is fluent on one topic. It goes back later with questions to 
    rehearse what the student has learned.
2. One pane is a chatbot representing the world's reference knowledge. It gives answers to the student's 
    questions. For some learnings, we may use "RAG" (Retrieval-Augmented Generation) to feed it with
    domain specific knowledge to avoid AI "hallucinations". In more specialized cases, the chatbot LLM 
    (Large Language Model) can be fine-tuned to improve thinking and reasoning.
3. One pane constitutes the electronic scratchpad where the student writes answers to challenges from 
    Jimini Cricket. The mentor can read the scratchpad to correct mistakes in the student's answer, to 
    evaluate the learning progress and guide further questions.
4. One pane displays the progress of the student as points gained when answering to questions. Like with 
    Duolingo the language learning app, there are personal scores and leaderboard levels in the 
    progression. Every week, the score of the student is reset and when the student exceeds a threshold at 
    the end of the week, he is promoted to a higher level. Conversely, if he doesn't reach a floor limit during 
    the week, he is demoted one level. If the student accepts and has a Polygon blockchain account, he may 
    join the leaderboard, claim NFT certificates and compare himself with the other students.

### 0.2 Implementation Details

1. Jiminy Cricket will be open-source, MIT License. It will be built preferably using open-weights models, 
    so that local servers may be deployed by users, who want to keep their confidential training data in 
    house and LLM can be fine-tuned if specific domain knowledge is required.
2. There will be in parallel a commercial company. It is named  **J(AI)miny Cricket** , to avoid potential
    trademark issues with Walt Disney.
3. The commercial company will support the costs of operating the Jiminy Cricket web app service,
    including the costs of running large AI models.
4. The business model of the commercial company is a "Fremium" B2B. The service is free for beginners 
    and for some popular learning domains. The service is paid for other learning domains. 
5. Registration is required for both the free and paid services. Identification serves only to keep control on 
    the API keys and protect against bots.
6. The commercial company participates in development of Jiminy Cricket. Its revenues come from 
    consulting services for corporate use of Jiminy Cricket. It helps build on-demand specialized trainings, 
    train client staff to maintain and expand the learning tool, deploy local AI servers and web app servers 
    in CD/CI (Continuous Development/ Continuous Integration)
7. The Jiminy Cricket AI Mentor will be developed as a generic agent, that will read scripts to drive 
    learning progression, scores, and leaderboard


8. The blockchain leaderboard will deliver NFTs to certify level achievements in specific knowledge 
    domains. Students will be incentivized to attract more students, to increase the value of their NFTs as 
    certificates.

## 1 Development Roadmap

### 1.1 Core Components

Jiminy Cricket development starts with an existing bootcamp hack. It is composed of modified components 
of this bootcamp hack.

1. The reference chatbot (currently the Solidity code helper) with its existing RAG complement.
2. The scrapbook.
3. A reduced Jiminy Cricket (currently the Solidity code auditor).
4. The existing blockchain Solidity deployment of the hack is modified to become a rudiment of the 
    blockchain-based leaderboard.

### 1.2 Technologies of the core Jiminy Cricket

- TypeScript, NextJS/CSS and Vercel are used for the frontend.
- OpenRouter interface to do API calls to AI inference servers.
- Gemini 2 Flash (as of May 2025) for Jiminy Cricket and Qwen 3 for the reference chatbot.
- Polygon blockchain for the leaderboard.

At the beginning and as of May 2025, Jiminy Cricket uses public inference servers. Only them have the GPU 
power that is required to run LLMs of 100B parameters or more. When newer LLM can offer decent results 
using consumer-grade hardware, Jiminy Cricket can consider using local inferences.

### 1.3 Jiminy Cricket Mentor Track

### 1.4 Jimini Cricket Blockchain Leaderboard Track

The Leaderboard can be developed in  3  stages:

- With a very small number of users, scores can be kept in an enumerable map, using OpenZeppelin 
    sample code: https://github.com/OpenZeppelin/openzeppelin-
    contracts/blob/master/contracts/utils/structs/EnumerableMap.sol
- With more users, we can try and use events to store scores.
- With a lot users, the cost of storage on the blockchain is prohibitive , it is preferable to use The Graph 
    and have a dedicated Subgraph to support le Leaderboard: https://github.com/graphprotocol/graph-
    tooling/tree/main/examples

### 1.4.1 Keeping scores in an Enumerable Map

If the leaderboard is stored in a contract, we can use an enumerable pattern (like OpenZeppelin's 
EnumerableSet) to track participants.

```
function getAllScores() public view returns (address[] memory, uint256[] memory) {
    address[] memory participants = new address[](participantCount);
    uint256[] memory scores = new uint256[](participantCount);
    for (uint i = 0; i < participantCount; i++) {
        participants[i] = participantList[i];
        scores[i] = scores[participantList[i]];
}
return (participants, scores);
}
```
Then, off-chain, we sort and compute the rank.

### 1.4.2 Event-Based Indexing (Using Ethers.js + Backend)

- Whenever a student self-registers to Jimini Cricket, a blockchain event ParticipantAdded is emitted 
    with the participant ID.


- Whenever a student answers a challenge from Jimini Cricket, a blockchain event ScoreUpdated is 
    emitted with the student new score.
- Off-chain, we have a JavaScript fetches all ParticipantAdded/ScoreUpdated events emitted by the 
    contract and reconstruct the leaderboard.
    const events = await contract.queryFilter("ScoreUpdated");
    const scores = {};
    events.forEach(e => scores[e.args.participant] = e.args.newScore);
- Off-chain, every week at  12  AM UTC, a blockchain function will reset the leaderboard and each 
    participant smart contract will self-demote or self-promote its ranking (logic to be defined).

### 1.4.3 Keeping scores in TheGraph

A Subgraph (from The Graph Protocol) stores indexed blockchain data in a decentralized network of 
Indexers, but the data itself is stored in a PostgreSQL database managed by the Indexer nodes. Here's how 
it works:

### 1.4.3.1 Where Subgraph Data is Stored

### Storage is decentralized in indexers.

- Subgraphs are hosted by **Indexers** , who run Graph Nodes that index and store the data. For Jimini 
    Cricket, the company J(AI)mini will be a subgraph indexer, indexing only Jimini Cricket subgraphs.
- The data is stored in a PostgreSQL database on the Indexer's server.
- The Graph Network uses Ethereum/IPFS for metadata (subgraph manifests), but the actual indexed 
    data is in the Indexer's PostgreSQL database.

Data is queried via GraphQL

- Once indexed, the data is served via GraphQL APIs (e.g.,
    https://api.thegraph.com/subgraphs/name/...).

### 1.4.3.2 How Data Gets into the Subgraph

- The leaderboard smart contract emits events (e.g., ScoreUpdated(address user, uint256 score)).
- The subgraph’s subgraph.yaml defines which events to index.
- The indexer node listens to these events and processes them via mapping functions (written in 
    AssemblyScript).
- The processed data is stored in tables (e.g., Participant, Score) in the Indexer’s PostgreSQL DB.

### 1.4.3.3 Example: Leaderboard Subgraph Storage

- Subgraph Schema (schema.graphql)

```
type Participant @entity {
    id: ID! # Ethereum address
    score: BigInt!
}
```
- Generated PostgreSQL Tables

```
COLUMN TYPE
ID TEXT (Primary Key)
SCORE NUMERIC
```
When you query the subgraph, the Graph Node (indexer node) fetches data from these tables.

### 1.4.3.4 Accessing the Data

You query the subgraph via GraphQL:

```
{
  participants(orderBy: score, orderDirection: desc) {
    id
    score
  }
}
```
This returns all participants sorted by score, allowing you to compute ranks off-chain.


### 1.4.3.5 Is the Data Persisted Forever?

**Hosted Service (Deprecated):**

- The Graph’s legacy hosted service stored data centrally (AWS-backed).

**Decentralized Network:**

- The deprecated legacy hosting service stored data centrally.
- Now, data persists as long as Indexers keep serving it (incentivized by GRT tokens).
- If no Indexers support your subgraph, data may become unavailable.

**GitHub Example**

Check out a full leaderboard subgraph implementation: Basic Subgraph Example

## 2 Current Bootcamp Hack

Work in progress.

## 3 Jiminy Cricket Core

Work in progress.

## 4 Bonus: Jiminy Cricket Role in Pinocchio’s Learning Progression

- **Moral Conscience & Guide** – In both versions, the cricket serves as Pinocchio’s conscience, advising 
    him against mischief and urging him to make good choices.
- In _Collodi’s original_ , the cricket scolds Pinocchio for his disobedience, prompting the puppet to kill 
him in a fit of rage (though he later returns as a ghost).
- In _Disney’s version_ , Jiminy is appointed by the Blue Fairy as Pinocchio’s official "conscience" and 
remains a loyal companion.
- **Symbol of Wisdom & Consequences** – The cricket’s warnings often foreshadow the troubles 
    Pinocchio will face if he ignores advice (e.g., skipping school, trusting strangers like the Fox and Cat, or 
    going to Pleasure Island).
- **Redemption & Growth** – By the end of the story, Pinocchio learns to heed Jiminy’s guidance, 
    demonstrating his moral growth. In Disney’s version, Jiminy is rewarded for his efforts when Pinocchio 
    becomes a real boy.
