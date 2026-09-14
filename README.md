# MultiSig Wallet

An Ethereum multisignature wallet built with Solidity, Hardhat 3, ethers.js, and a Next.js frontend. The wallet can hold ETH, propose arbitrary contract calls, collect approvals from a configurable group of owners, and execute a proposal after it reaches the required confirmation threshold.

> **Important:** This project is intended for learning and development. The contracts have not been audited and should not be used to custody production funds.

## Features

- Configure wallet owners and an approval threshold at deployment
- Deposit ETH through `receive` or `fallback`
- Submit ETH transfers or arbitrary calldata
- Confirm and revoke confirmations on pending transactions
- Execute transactions after the threshold is reached
- Add and remove owners
- Inspect owners, balance, threshold, transactions, and confirmation status
- Manage the wallet from a responsive MetaMask-enabled web interface
- Run Solidity and TypeScript tests with Hardhat

## How it works

1. Deploy the wallet with a unique list of owner addresses and a confirmation threshold.
2. Fund the deployed contract with ETH.
3. Submit a transaction containing a destination, value, and optional calldata.
4. Wallet owners confirm the pending transaction.
5. Once enough confirmations have been collected, the transaction can be executed.

The current contract allows anyone to submit or execute a transaction, while confirmations are restricted to owners. Owner addition and removal require only one owner call; they do not go through the multisig proposal process.

## Tech stack

- Solidity 0.8.28
- Hardhat 3
- ethers.js 6
- Mocha and Chai
- Next.js 14, React 18, TypeScript, and Tailwind CSS

## Project structure

```text
contracts/MultiSigWallet.sol           Core wallet contract
test/MultiSigWallet.ts                 TypeScript contract tests
ignition/modules/MultiSigWallet.ts     Hardhat Ignition deployment module
scripts/multiSigWallet.ts              Script-based deployment
frontend/                              Next.js wallet interface
```

The repository also contains Hardhat's sample `Counter` contract and tests; they are not part of the multisig wallet.

## Prerequisites

- A recent Node.js release supported by Hardhat 3
- npm
- MetaMask or another EIP-1193 browser wallet for the frontend
- Sepolia ETH if deploying to the Sepolia testnet

## Installation

Install the smart-contract dependencies from the repository root:

```bash
npm install
```

Install the frontend dependencies separately:

```bash
cd frontend
npm install
cd ..
```

## Test and compile

Run the complete test suite:

```bash
npx hardhat test
```

Run only the TypeScript tests or compile the contracts:

```bash
npx hardhat test mocha
npx hardhat compile
```

## Local development

Start a local Hardhat node in one terminal:

```bash
npx hardhat node
```

In a second terminal, deploy the wallet:

```bash
npx hardhat run scripts/multiSigWallet.ts --network localhost
```

The deployment script currently uses three development owner addresses and requires two confirmations. To use different owners or a different threshold, edit `scripts/multiSigWallet.ts` before deploying.

Copy the printed contract address into the frontend configuration:

```bash
cp frontend/env.example frontend/.env.local
```

Then set:

```dotenv
NEXT_PUBLIC_DEFAULT_NETWORK=localhost
NEXT_PUBLIC_CONTRACT_ADDRESS_LOCALHOST=0xYourDeployedContractAddress
```

Start the frontend:

```bash
cd frontend
npm run dev
```

Open [http://localhost:3000](http://localhost:3000), connect MetaMask to `http://127.0.0.1:8545` (chain ID `31337`), and enter or select the deployed wallet address.

## Deploy to Sepolia

Create a root `.env` file with a funded deployer account and Sepolia RPC endpoint:

```dotenv
SEPOLIA_RPC_URL=https://your-sepolia-rpc-url
SEPOLIA_PRIVATE_KEY=0xyour-private-key
```

Never commit this file or expose the private key through a `NEXT_PUBLIC_` frontend variable.

Review the owner addresses in the deployment script, then deploy:

```bash
npx hardhat run scripts/multiSigWallet.ts --network sepolia
```

Configure the frontend with the resulting address:

```dotenv
NEXT_PUBLIC_DEFAULT_NETWORK=sepolia
NEXT_PUBLIC_CONTRACT_ADDRESS_SEPOLIA=0xYourDeployedContractAddress
NEXT_PUBLIC_RPC_URL=https://your-sepolia-rpc-url
```

Restart the frontend after changing its environment variables.

Hardhat Ignition is also available, but its module contains its own fixed owner list. Review it before deployment:

```bash
npx hardhat ignition deploy ignition/modules/MultiSigWallet.ts --network sepolia
```

## Contract interface

| Function | Access | Purpose |
| --- | --- | --- |
| `submitTransaction(to, value, data)` | Anyone | Create a pending transaction |
| `confirmTransaction(txIndex)` | Owner | Add the caller's confirmation |
| `revokeConfirmation(txIndex)` | Confirming address | Remove the caller's confirmation |
| `executeTransaction(txIndex)` | Anyone | Execute a sufficiently confirmed transaction |
| `addOwner(owner)` | Owner | Add a wallet owner |
| `removeOwner(owner)` | Owner | Remove an owner while keeping the threshold valid |
| `getOwners()` | Anyone | Return all current owners |
| `getTransaction(txIndex)` | Anyone | Return a transaction proposal |
| `canExecute(txIndex)` | Anyone | Check whether a pending transaction is executable |

## Security notes

- The contract is unaudited.
- Owner-management actions are controlled by any single owner, not by threshold approval.
- The confirmation threshold cannot be changed after deployment in the current Solidity contract.
- External calls can invoke arbitrary code. The transaction is marked executed before the call, which provides basic reentrancy protection for replaying that transaction.
- Always verify the network, owners, threshold, destination, value, and calldata before signing.
- The frontend contains optional UI paths for newer contract methods such as threshold changes and vote counts. Those methods are not implemented by the contract in this repository and will fail when used with it.

## License

The Solidity contract is published under the MIT SPDX identifier. The root package currently declares the ISC license. Add a repository-level `LICENSE` file before distributing the project.
