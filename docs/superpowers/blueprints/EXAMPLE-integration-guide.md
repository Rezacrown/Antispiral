# Integration Guide: Voting DApp

## Architecture

```
Frontend (React + wagmi) ──REST──→ Backend (Express + JWT)
       │                                    │
       └──────────Web3──────────→ Smart Contract (Solidity / Sepolia)
                                         │
                                         ▼
                                    Database (PostgreSQL)
```

## Layer Connections

| UI Action | API Endpoint | SC Function | SDK |
|-----------|-------------|-------------|-----|
| createProposal | POST /api/proposals | createProposal(string,uint256) | wagmi + axios |
| castVote | — | castVote(uint256) | wagmi |
| viewResults | GET /api/proposals | getProposal(uint256) | wagmi + axios |
| connectWallet | — | — | wagmi (useConnect) |

## Data Flow: createProposal

1. **UI** — Admin fills form (title + deadline), clicks Submit
2. **Validation** — Frontend checks: wallet connected? admin role?
3. **API Call** — `POST /api/proposals` with JWT in Authorization header
4. **Backend validation** (5 steps):
   - authenticateUser (JWT valid?)
   - authorizeAdmin (role == "admin"?)
   - validateTitle (1-200 chars?)
   - validateDeadline (future + 1h?)
   - checkDuplicate (unique title?)
5. **Backend** — Insert to `proposals` table → return `{ id, title, deadline }`
6. **UI** — Get proposal ID, call `VotingContract.createProposal(title, deadline)`
7. **Contract validation** (3 steps):
   - onlyOwner (msg.sender == owner?)
   - validateDeadline (deadline > block.timestamp?)
   - checkDuplicate (keccak256(title) unique?)
8. **UI** — Wait for tx receipt → redirect to /dashboard

## Data Flow: castVote

1. **UI** — User clicks Vote on a proposal
2. **UI** — Call `VotingContract.castVote(proposalId)` via wagmi `useContractWrite`
3. **Contract validation**:
   - proposalExists (proposalId in storage?)
   - votingOpen (block.timestamp < deadline?)
   - preventDoubleVote (voter hasn't voted?)
4. **UI** — Wait for tx receipt
5. **UI** — Update local state (increment vote count)
6. **UI** — Show success toast

## SDK / Libraries

| Connection | Library | Purpose |
|-----------|---------|---------|
| Frontend ↔ Contract | wagmi + viem | Wallet connect, contract read/write |
| Frontend ↔ Backend | axios + @tanstack/react-query | REST API calls + caching |
| Backend ↔ DB | Prisma | ORM, migrations |
| Backend Auth | jsonwebtoken | JWT generation & verification |

## Environment Requirements

```
# Frontend
VITE_CONTRACT_ADDRESS=0x...
VITE_CHAIN_ID=11155111

# Backend
DATABASE_URL=postgresql://...
JWT_SECRET=...
PORT=3001

# Smart Contract
NETWORK=sepolia
RPC_URL=https://sepolia.infura.io/...
```
