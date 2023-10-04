import Button from '@site/src/components/button'

# TON Node Types

When diving into the world of The Open Network (TON), understanding the distinct node types and their functionalities is crucial. This article breaks down each node type to provide clarity for developers wishing to engage with the TON blockchain.

## Types of TON Nodes

### Full Node

A **Full Node** in TON is a node that maintains synchronization with the blockchain.

It retains the _current state_ of the blockchain and can house either the entire block history or parts of it. This makes it the backbone of the TON blockchain, facilitating the network's decentralization and security.

<Button href="/participate/run-nodes/full-node"
colorType="primary" sizeType={'sm'}>
Running a Full Node
</Button>

### Archive Node

An **Archive Node** is essentially a full node that archives the entire block history.

Such nodes are indispensable for creating blockchain explorers or other tools that necessitate a full blockchain history.

  [Running an Archive Node](/participate/run-nodes/archive-node)


### Liteserver Node

When an endpoint is activated in a full node, the node assumes the role of a **Liteserver**. This node type can field and respond to requests from Lite Clients, allowing for seamless interaction with the TON Blockchain.

#### Interaction with Lite Clients

Liteservers enable swift communication with Lite Clients, facilitating tasks like balance checks or transaction submissions without necessitating the full block history.

#### Public Liteservers

The TON Foundation provides several public Liteservers, integrated into the global config, which are accessible for universal use. These endpoints, such as those used by standard wallets, ensure that even without setting up a personal liteserver, interaction with the TON Blockchain remains possible.

- [Public Liteserver Configurations - mainnet](https://ton.org/global-config.json)
- [Public Liteserver Configurations - testnet](https://ton.org/testnet-global.config.json)

[Running a Liteserver](/participate/run-nodes/liteserver)


### Validator Node

A **Validator Node** is activated when it holds a necessary amount of Toncoin as a stake. Validator nodes are vital for the network's operability, participating in the validation of new network blocks.

#### Proof-of-Stake Mechanism and Rewards

TON operates on a Proof-of-Stake mechanism, where validators are pivotal in maintaining network functionality. Validators are [rewarded in Toncoin](/participate/network-maintenance/staking-incentives) for their contributions, incentivizing network participation and ensuring network security.

  [Running a Full Node as a Validator](/participate/run-nodes/full-node#become-a-validator)