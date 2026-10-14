# CampusPass — Wallet-Based Token-Gated Access (Intermediate)

> **Part of the ShadowFox Blockchain Developer Internship**  
> **Level:** Intermediate Task  
> **Project Name:** CampusPass (MemberGate)

CampusPass is a decentralized application (dApp) that connects an Ethereum Web3 wallet (MetaMask) and verifies membership directly against an on-chain smart contract before granting access to exclusive student and member resources. 

Unlike traditional Web2 applications that rely on central databases, usernames, and passwords, CampusPass treats the user's cryptographic wallet address as their identity and an Ethereum smart contract as the single source of truth for access control.

---

## 🌟 Key Features

- **Web3 Wallet Authentication:** Seamless connection with MetaMask using `ethers.js` (v6). Automatically detects installed providers, fetches connected accounts, and monitors real-time account/network changes.
- **On-Chain Membership Verification:** Queries the `Membership.sol` smart contract's `isMember(address)` view function directly from the blockchain to determine permission levels without requiring a backend server.
- **Role-Based Token Gating & Protected Content:** 
  - **Guests:** Greeted with an onboarding overview and wallet connection prompt.
  - **Non-Members:** Displayed a "Membership Required" notification prompting them to request access.
  - **Verified Members:** Granted instant access to member-only materials (handbooks, workshop notes, events calendar, and career checklists) with browser-generated downloads.
- **Admin Dashboard (Owner Only):** Contract owner (`onlyOwner`) automatically unlocks a administrative control panel to grant (`grantMembership`) or revoke (`removeMembership`) membership on-chain.
- **Live Transaction Feedback:** Tracks transaction lifecycle states (`pending`, `success`, `failed`), displaying the real-time transaction hash, confirmations, and MetaMask rejection warnings.
- **Network Validation & Switching:** Detects chain mismatches against the configured target network (`VITE_CHAIN_ID`) and offers one-click network switching via `wallet_switchEthereumChain`.

---

## 🛠️ Tech Stack

| Layer | Technology | Description |
|---|---|---|
| **Frontend Framework** | React 18 + Vite 5 | Fast, modern Single-Page Application (SPA) |
| **Styling** | Tailwind CSS v4 (`@tailwindcss/vite`) | Modern, responsive, utility-first UI styling |
| **Blockchain Client** | `ethers.js` (v6.13.4) | Interacting with EVM nodes, providers, signers, and contracts |
| **Smart Contract** | Solidity (`^0.8.0`) | On-chain access registry with owner controls |
| **Wallet Provider** | MetaMask | EIP-1193 browser extension wallet |

---

## 📐 Smart Contract Architecture

The core access control logic is defined in `contracts/Membership.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Membership {
    address public owner;
    mapping(address => bool) public members;

    event MembershipGranted(address indexed user);
    event MembershipRemoved(address indexed user);

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can do this");
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function grantMembership(address user) external onlyOwner {
        members[user] = true;
        emit MembershipGranted(user);
    }

    function removeMembership(address user) external onlyOwner {
        members[user] = false;
        emit MembershipRemoved(user);
    }

    function isMember(address user) external view returns (bool) {
        return members[user];
    }
}
```

### Contract Functions:
- `owner()`: Returns the contract deployer address.
- `isMember(address user)`: Free, gasless `view` call returning `true` or `false`.
- `grantMembership(address user)`: State-changing transaction restricted to `onlyOwner` that records membership and emits `MembershipGranted`.
- `removeMembership(address user)`: State-changing transaction restricted to `onlyOwner` that revokes membership and emits `MembershipRemoved`.

---

## 📁 Project Structure

```text
Intermediate/
├── contracts/
│   └── Membership.sol          # Solidity smart contract
├── src/
│   ├── components/
│   │   ├── AdminDashboard.jsx      # Owner-only interface to grant/revoke access
│   │   ├── ConnectWallet.jsx       # Initial wallet connection CTA & guide
│   │   ├── MemberDashboard.jsx     # Member portal with downloadable resources
│   │   ├── MembershipStatus.jsx    # Status badges (checking, verified, not-member)
│   │   ├── Navbar.jsx              # App header with address & disconnect button
│   │   ├── ProtectedContent.jsx    # High-level gating component
│   │   ├── TransactionStatus.jsx   # Live on-chain tx alert with hash & status
│   │   └── WalletStatus.jsx        # Connected address, chain ID, and copy button
│   ├── contract/
│   │   ├── abi.js                  # Human-readable contract ABI definition
│   │   └── config.js               # Contract address & chain ID environment config
│   ├── hooks/
│   │   ├── useMembership.js        # Contract query & transaction state hook
│   │   └── useWallet.js            # MetaMask provider & account management hook
│   ├── App.jsx                     # Core application layout & routing state
│   ├── index.css                   # Tailwind v4 import & base styles
│   └── main.jsx                    # React entrypoint
├── .env.example                    # Template for environment variables
├── index.html                      # HTML entry with viewport & title
├── package.json                    # Project dependencies & scripts
├── vite.config.js                  # Vite configuration with Tailwind plugin
└── README.md                       # Project documentation
```

---

## 🚀 Getting Started

### Prerequisites

1. **Node.js**: Install Node.js LTS (v18 or higher) and `npm`.
2. **MetaMask Extension**: Install the [MetaMask browser extension](https://metamask.io/download/).
3. **EVM Testnet or Local Chain**: 
   - Ethereum Sepolia testnet, Polygon Amoy, or a local Hardhat/Anvil node.
   - Testnet tokens to cover gas fees for contract deployment and admin transactions.

---

### Step 1: Deploy the Smart Contract

You can deploy `contracts/Membership.sol` using **Remix IDE** or **Hardhat**:

#### Using Remix IDE (Quickest):
1. Open [Remix IDE](https://remix.ethereum.org/).
2. Create a new file `Membership.sol` and paste the contents of `contracts/Membership.sol`.
3. Compile the contract using Solidity compiler `0.8.20` or any `^0.8.0` version.
4. Under **Deploy & Run Transactions**:
   - Environment: Select **Injected Provider - MetaMask**.
   - Ensure MetaMask is connected to your desired network (e.g. Sepolia or local node).
   - Click **Deploy** and confirm the transaction in MetaMask.
5. Copy the deployed contract address.

---

### Step 2: Configure Environment Variables

In the `Intermediate/` directory, create a `.env` file based on `.env.example`:

```bash
cp .env.example .env
```

Open `.env` and fill in the values:

```env
VITE_CONTRACT_ADDRESS=0xYourDeployedContractAddressHere
VITE_CHAIN_ID=11155111
```

> **Chain ID Reference:**
> - `11155111`: Ethereum Sepolia Testnet
> - `31337`: Local Hardhat Network
> - `1337`: Local Ganache
> - `1`: Ethereum Mainnet

---

### Step 3: Install Dependencies & Run

1. Open your terminal in the `Intermediate` directory:
   ```bash
   cd Intermediate
   npm install
   ```

2. Start the Vite development server:
   ```bash
   npm run dev
   ```

3. Open the provided local URL in your browser (usually `http://localhost:5173/`).

---

## 🧪 Testing & Demonstration Guide

### 1. Guest Flow (Not Connected)
- Open the application.
- Observe the step-by-step onboarding cards and the "Connect Wallet" button.
- Notice that no protected content or admin panels are visible.

### 2. Connected Non-Member Flow
- Click **Connect Wallet** and authorize an account in MetaMask that has **not** been granted membership.
- The dApp queries `isMember(account)` on-chain.
- The interface displays the **Membership Required** screen showing the wallet address and advising the user to contact the contract owner.

### 3. Contract Owner & Admin Flow
- Switch in MetaMask to the wallet address that deployed the `Membership` contract.
- The dApp recognizes `isOwner === true` by comparing `owner()` with the active account.
- The **Admin Dashboard** is unlocked.
- In the **Grant Membership** form:
  - Enter the non-member wallet address.
  - Click **Grant Membership**.
  - Confirm the transaction in MetaMask.
  - Watch the live transaction status card transition from `pending` (with transaction hash) to `success`.

### 4. Verified Member Flow
- Switch MetaMask back to the newly granted member wallet (or click "Refresh Membership Status").
- The status updates to **Membership status: VERIFIED**.
- The **Member Dashboard** renders, unlocking exclusive student documents (Handbooks, Notes, Events Calendar, Checklists) available for download.

### 5. Revoking Access
- Reconnect with the owner account.
- Enter the member's wallet address into the **Remove Membership** section and confirm the transaction.
- Once mined, that wallet instantly loses access to the Member Dashboard on their next refresh or session.

---

## 🔒 Security & Web3 Best Practices

- **Zero Passwords / Off-Chain Credentials:** Authentication is purely cryptographic based on public-key cryptography (EIP-1193).
- **On-Chain Enforcement:** Modifiers (`onlyOwner`) prevent non-owners from modifying access lists even if they attempt direct contract interaction.
- **Client-Side Secret Isolation:** Never commit `.env` containing sensitive credentials or deployment private keys to version control.
- **Fail-Safe UI States:** Includes dedicated views for disconnected wallets, unconfigured contract variables, wrong network connections, and rejected transactions (`code: 4001`).

---

## 📜 License

This project is developed for educational purposes as part of the **ShadowFox Blockchain Developer Internship**.
