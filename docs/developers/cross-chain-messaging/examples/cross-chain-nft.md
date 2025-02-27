---
sidebar_position: 3
sidebar_label: Cross-Chain NFT
hide_title: true
id: cross-chain-nft
title: Build a Cross-Chain NFT
---

# Build a cross-chain NFT

In this tutorial you will create an NFT collection with cross-chain transfer
capabilities using
[Zeta Connector](/developers/cross-chain-messaging/connector).

![Cross-chain NFT transfer](/img/graphs/cross-chain-nft-transfer.svg)

## Set up your environment

```
git clone https://github.com/zeta-chain/template
```

Install the dependencies:

```
yarn add --dev @openzeppelin/contracts
```

## Create a new contract

To create a cross-chain NFT collection you will need to deploy NFT contracts
implementing Zeta's interfaces and functions to multiple chains.

In a nutshell, your contracts will need to:

- Implement a `crossChainTransfer` function, that burns the NFT on the current
  chain and uses `zeta.send` to send a message that mints it on the other one.
- Implement `onZetaMessage` to mint the NFT that was burned on the other chain.
- Implement `onZetaRevert` to handle `crossChainTransfer` errors, and re-mint
  the previously burned NFT.

```solidity title="contracts/CrossChainWarriors.sol" reference
https://github.com/zeta-chain/example-contracts/blob/feat/import-toolkit/messaging/warriors/contracts/CrossChainWarriors.sol
```

## Create a deployment task

The deploy task is almost exactly the same as the one used in the cross-chain
messaging example. The only difference is that CrossChainWarriors' constructor
function accepts one additional argument (`useEven`), which is used to prevent
collisions between cross-chain token IDs. We're using `true` as a value for
`useEven` in the `factory.deploy` function call.

The deploy task will deploy the CrossChainWarriors contract to two or more
chains and set the interactors for all chains.

```ts title="tasks/deploy.ts" reference
https://github.com/zeta-chain/example-contracts/blob/feat/import-toolkit/messaging/warriors/tasks/deploy.ts
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

## Create a mint task

The mint task accepts a contract address as an argument, calls the `mint`
function on it, searches the events for a "Transfer" event and prints out the
token ID.

```ts title="tasks/mint.ts" reference
https://github.com/zeta-chain/example-contracts/blob/feat/import-toolkit/messaging/warriors/tasks/mint.ts
```

```
npx hardhat mint --network goerli --contract 0xFeAF74733B6f046F3d609e663F667Ba61B19A148

🔑 Using account: 0x2cD3D070aE1BD365909dD859d29F387AA96911e1

✅ "mint" transaction has been broadcasted to goerli
📝 Transaction hash: 0x48498d595f7aa5ab2c1569fe56b9861c4902a05420af2b7054008d1c283d8e40
🌠 Minted NFT ID: 12

Please, refer to ZetaChain's explorer for updates on the progress of the cross-chain transaction.

🌍 Explorer: https://explorer.zetachain.com/cc/tx/0x48498d595f7aa5ab2c1569fe56b9861c4902a05420af2b7054008d1c283d8e40
```

## Create a transfer task

The transfer task accepts the following arguments:

- contract: the CrossChainWarriors contract address
- address: recipient address
- destination: destination chain ID
- token: token ID
- amount: amount of tokens to cover gas costs

```ts title="tasks/transfer.ts" reference
https://github.com/zeta-chain/example-contracts/blob/feat/import-toolkit/messaging/warriors/tasks/transfer.ts
```

```
npx hardhat transfer --network goerli --contract 0xFeAF74733B6f046F3d609e663F667Ba61B19A148 --address 0x2cD3D070aE1BD365909dD859d29F387AA96911e1 --destination 97 --token 2 --amount 0.4
```

After the transfer transaction is confirmed, you will be able to see the NFT on
the recipient address page on the destination chain (in this case, BSC).

![](/img/docs/ccm-warriors-explorer.jpg)

## Source Code

You can find the source code for the example in this tutorial here:

https://github.com/zeta-chain/example-contracts/blob/feat/import-toolkit/messaging/warriors
