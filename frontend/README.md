# MultiSig Wallet Frontend

A modern, responsive interface for managing the MultiSig Wallet contract, built with Next.js 14 and Tailwind CSS.

## Features

- **Wallet connection:** Connect with MetaMask
- **Wallet overview:** View the contract balance, owner list, and confirmation threshold
- **Transaction management:**
  - Create transaction proposals
  - Confirm or revoke transaction confirmations
  - Execute transactions that have reached the required threshold
  - View transaction details and status
- **Owner management:**
  - Add owners
  - Remove owners
  - Display and attempt to change the confirmation threshold
- **Data refresh:** Refresh wallet and transaction data after on-chain actions
- **Modern interface:** Responsive layout with light and dark themes

> **Contract compatibility:** The Solidity contract currently included in this repository does not implement `changeThreshold`, `version`, `getOwnerVoteCount`, or `incrementOwnerVoteCount`. The frontend detects the absence of version and vote-count methods, but attempting to change the threshold will fail with the current contract.

## Tech stack

- **Framework:** Next.js 14 with the App Router
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **Web3:** ethers.js 6
- **Icons:** Lucide React
- **Date utilities:** date-fns

## Getting started

### 1. Install dependencies

From the repository root:

```bash
cd frontend
npm install
```

### 2. Configure environment variables

Copy the provided example:

```bash
cp env.example .env.local
```

Then configure the default network and the deployed contract address:

```dotenv
NEXT_PUBLIC_DEFAULT_NETWORK=sepolia
NEXT_PUBLIC_CONTRACT_ADDRESS_SEPOLIA=0xYourDeployedContractAddress
NEXT_PUBLIC_RPC_URL=https://your-sepolia-rpc-url
```

For local development, use:

```dotenv
NEXT_PUBLIC_DEFAULT_NETWORK=localhost
NEXT_PUBLIC_CONTRACT_ADDRESS_LOCALHOST=0xYourDeployedContractAddress
```

All variables prefixed with `NEXT_PUBLIC_` are exposed to the browser. Never place private keys or other secrets in `.env.local`.

### 3. Start the development server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### 4. Create a production build

```bash
npm run build
npm start
```

## Usage

### Connect a wallet

1. Install the MetaMask browser extension.
2. Select **Connect Wallet**.
3. Approve the connection in MetaMask.
4. Make sure MetaMask is connected to the network where the contract is deployed.

The current wallet provider is configured to request a switch to Sepolia when another network is detected.

### Manage a multisig wallet

1. Enter the address of a deployed `MultiSigWallet` contract. If an address is configured for the active network, the app loads it automatically.
2. Review the wallet balance, owners, threshold, and transaction history.
3. As an owner, create a transaction proposal with a recipient and ETH amount.
4. Confirm a pending transaction or revoke an existing confirmation.
5. Execute the transaction after it reaches the required number of confirmations.
6. Use the **Owners** tab to add or remove owners.

All state-changing operations require a wallet signature and enough ETH to pay network gas fees.

## Supported networks

Network definitions are located in `lib/config.ts`:

| Network | Chain ID | Environment variable |
| --- | ---: | --- |
| Localhost | 31337 | `NEXT_PUBLIC_CONTRACT_ADDRESS_LOCALHOST` |
| Sepolia | 11155111 | `NEXT_PUBLIC_CONTRACT_ADDRESS_SEPOLIA` |
| Ethereum Mainnet | 1 | `NEXT_PUBLIC_CONTRACT_ADDRESS_MAINNET` |

Although these networks are defined in the configuration module, the current `Web3Provider` actively requests Sepolia. Additional provider changes are required for a fully selectable multi-network experience.

## Project structure

```text
frontend/
├── app/
│   ├── globals.css                 Global styles
│   ├── layout.tsx                  Root layout
│   └── page.tsx                    Main page
├── components/
│   ├── AccountDetailsModal.tsx     Account details dialog
│   ├── AlertDialog.tsx             Alert dialog
│   ├── AlertProvider.tsx           Application notifications
│   ├── CreateTransactionModal.tsx  Transaction proposal form
│   ├── Header.tsx                  Application header
│   ├── OwnersManagement.tsx        Owner-management interface
│   ├── TransactionCard.tsx         Transaction display
│   ├── TransactionList.tsx         Transaction collection
│   ├── WalletConnectPrompt.tsx     Wallet connection prompt
│   ├── WalletDashboard.tsx         Main wallet dashboard
│   ├── WalletInfo.tsx              Wallet summary cards
│   └── Web3Provider.tsx            Browser-wallet context
├── hooks/
│   ├── useMultiSigWallet.ts        Wallet data hook
│   └── useTransactions.ts          Transaction data hook
├── lib/
│   ├── abis.ts                     Contract ABI
│   ├── config.ts                   Network and contract configuration
│   ├── contracts.ts                Contract helpers
│   └── network.ts                  Network helpers
└── types/
    └── window.d.ts                 Browser-wallet types
```

## Troubleshooting

- **The contract cannot be loaded:** Confirm that the address is valid, deployed on the connected network, and configured under the matching environment variable.
- **Environment changes are ignored:** Restart the Next.js development server after editing `.env.local`.
- **MetaMask switches away from localhost:** The current provider automatically requests Sepolia. Update `components/Web3Provider.tsx` before using the local network through the UI.
- **A transaction fails:** Check that the connected account has permission for the requested action and enough ETH for gas.
- **Threshold changes fail:** The current contract has no `changeThreshold` function.

## Security

This frontend and its associated contract are intended for development and education. The contract has not been audited; do not use it to manage production funds.
