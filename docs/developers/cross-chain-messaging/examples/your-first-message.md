---
sidebar_position: 1
sidebar_label: Your First Cross-Chain Message
hide_title: true
id: hello-world
title: Your First Cross-Chain Message
---

# Your First Cross-Chain Message

In this tutorial we will create a simple contract that allows sending a message
from one chain to another using the
[Connector API](/developers/cross-chain-messaging/connector/).

## Set up your environment

```
git clone https://github.com/zeta-chain/template
```

Install the dependencies:

```
yarn add --dev @openzeppelin/contracts
```

## Create a new contract

```solidity title="contracts/CrossChainMessage.sol" reference
https://github.com/zeta-chain/example-contracts/blob/feat/import-toolkit/messaging/message/contracts/CrossChainMessage.sol
```

The `sendHelloWorld` function is used to send a "Hello, Cross-Chain World!"
message to another chain. It first checks the validity of the destination chain
ID. Then it fetches the required ZETA from the native chain token provided in
the transaction value, and approves the connector for the calculated Zeta value.
Then, it sends the message using the connector's `send` function.

The `onZetaMessage` function is used to handle incoming cross-chain messages. It
verifies the message signature and the message type. If it's a valid "Hello,
Cross-Chain World!" message, it emits a HelloWorldEvent with the message data.

The `onZetaRevert` function is triggered if a message fails to send. It decodes
the message similarly to `onZetaMessage` and emits a `RevertedHelloWorldEvent`
in case of failure.

## Create a deployment task

To establish cross-chain messaging between blockchains via ZetaChain, you should
deploy identical contracts on two or more connected chains.

You can specify the desired chains by adding a `--networks` parameter to the
deploy task, which accepts a list of network names separated by commas. For
instance, `--networks goerli,bsc-testnet`.

The main function maintains a mapping of network names to their corresponding
deployed contract addresses, iterating over the networks to deploy the contract
on each one.

The contract's constructor requires three arguments: the connector contract's
address, the ZETA token's address, and the ZETA token consumer contract's
address. These addresses are obtained using ZetaChain's `getAddress`.

The main function subsequently sets interactors for each contract. An interactor
is a mapping between a chain ID of the destination and the contract address on
that chain.

When deploying to two chains (like Goerli and BSC testnet), you will invoke
`setInteractorByChainId` on a Goerli contract and pass the BSC testnet chain ID
(97) and the BSC testnet contract address. You then perform the same operation
on a BSC testnet contract, passing the Goerli chain ID (5) and the Goerli
contract address. If deploying to more than two chains, you must call
`setInteractorByChainId` for each link between the chains.

```ts title="tasks/deploy.ts" reference
https://github.com/zeta-chain/example-contracts/blob/feat/import-toolkit/messaging/message/tasks/deploy.ts
```

Clear the cache and artifacts, then compile the contract:

```
npx hardhat compile --force
```

Run the following command to deploy the contract to two networks:

```
npx hardhat deploy --networks bsc-testnet,goerli
```

```
🚀 Successfully deployed contract on bsc-testnet.
📜 Contract address: 0x6Fd784c16219026Ab0349A1a8A6e99B6eE579C93

🚀 Successfully deployed contract on goerli.
📜 Contract address: 0xf1907bb130feb28D6e1305C53A4bfdb32140d8E6

🔗 Setting interactors for a contract on bsc-testnet
✅ Interactor address for 5 (goerli) is set to 0xf1907bb130feb28d6e1305c53a4bfdb32140d8e6

🔗 Setting interactors for a contract on goerli
✅ Interactor address for 97 (bsc-testnet) is set to 0x6fd784c16219026ab0349a1a8a6e99b6ee579c93
```

## Send a message

Create a new task to send a message from one chain to another. The task accepts
two parameters: the contract address and the destination chain ID.

The task calls sendHelloWorld on the contract, passing the arguments.

```ts title="tasks/message.ts" reference
https://github.com/zeta-chain/example-contracts/blob/feat/import-toolkit/messaging/message/tasks/message.ts
```

Send a message from BSC testnet to Goerli (chain ID: 5) using the contract
address (see the output of the `deploy` task). Make sure to submit enough native
tokens with `--amount` to pay for the transaction fees.

```
npx hardhat message --network bsc-testnet --destination 5 --contract 0x6Fd784c16219026Ab0349A1a8A6e99B6eE579C93 --amount 2
```

```
🔑 Using account: 0x2cD3D070aE1BD365909dD859d29F387AA96911e1

✅ "sendHelloWorld" transaction has been broadcasted to bsc-testnet
📝 Transaction hash: 0xa3a507d34056f4c00b753e7d0b47b16eb6d35b3c5016efa0323beb274725b1a1

Please, refer to ZetaChain's explorer for updates on the progress of
the cross-chain transaction.

🌍 Explorer: https://explorer.zetachain.com/cc/tx/0xa3a507d34056f4c00b753e7d0b47b16eb6d35b3c5016efa0323beb274725b1a1
```

After the cross-chain transaction is processed on ZetaChain, look up the
contract on the Goerli explorer by the contract address
(`0xf1907bb130feb28D6e1305C53A4bfdb32140d8E6`) and you should be able to see the
emitted `HelloWorldEvent` event.

![](/img/docs/ccm-message-explorer.png)

## Source Code

You can find the source code for the example in this tutorial here:

https://github.com/zeta-chain/example-contracts/blob/feat/import-toolkit/messaging/message
