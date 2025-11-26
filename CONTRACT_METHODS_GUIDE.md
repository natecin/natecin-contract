# NATECIN Smart Contract Methods & Features Guide

> **Complete guide for frontend integration with wagmi & viem using JSON ABI imports**

---

## 📁 ABI Import Structure

```
/lib/abi/
├── NatecinFactory.json
├── NatecinVault.json
└── VaultRegistry.json
```

### Importing ABIs in TypeScript

```typescript
import FACTORY_ABI from '@lib/abi/NatecinFactory.json';
import VAULT_ABI from '@lib/abi/NatecinVault.json';
import REGISTRY_ABI from '@lib/abi/VaultRegistry.json';

export const FACTORY_ADDRESS = "0x..."; // Your deployed factory address
export const REGISTRY_ADDRESS = "0x..."; // Your deployed registry address

export const ABIS = {
  factory: FACTORY_ABI.abi,
  vault: VAULT_ABI.abi,
  registry: REGISTRY_ABI.abi,
} as const;
```

---

## 🏭 NatecinFactory Contract

**Purpose**: Factory for creating and managing NATECIN vaults using EIP-1167 minimal proxies

### Key Features
- Gas-optimized vault creation (~94% gas reduction)
- Fee-based vault creation
- NFT fee management
- Vault tracking and discovery

### Core Methods

#### `createVault(address heir, uint256 inactivityPeriod)`
**Legacy method** - Creates a vault without NFT fee requirement

```typescript
import { useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
import { ABIS, FACTORY_ADDRESS } from './contracts/abi';

function CreateVaultComponent() {
  const { writeContract, data: hash, isPending } = useWriteContract();
  const { isLoading: isConfirming, isSuccess: isConfirmed } = 
    useWaitForTransactionReceipt({ hash });

  const handleCreate = async (heir: string, inactivityPeriod: number, value: bigint) => {
    writeContract({
      address: FACTORY_ADDRESS,
      abi: ABIS.factory,
      functionName: 'createVault',
      args: [heir as Address, BigInt(inactivityPeriod)],
      value, // ETH amount for deposit + fees
    });
  };

  return (
    <button onClick={() => handleCreate('0x...', 86400, parseEther('1.0'))}>
      {isPending ? 'Creating...' : 'Create Vault'}
    </button>
  );
}
```

#### `createVault(address heir, uint256 inactivityPeriod, uint256 estimatedNFTCount)`
**Enhanced method** - Creates vault with NFT fee calculation

```typescript
import { useReadContract } from 'wagmi';

function VaultCreationWithNFTs() {
  const { data: minFee } = useReadContract({
    address: FACTORY_ADDRESS,
    abi: ABIS.factory,
    functionName: 'calculateMinNFTFee',
    args: [BigInt(estimatedNFTCount)],
  });

  const handleCreate = async (heir: string, inactivityPeriod: number, nftCount: number) => {
    const deposit = parseEther('1.0'); // Regular deposit
    const totalValue = deposit + (minFee || 0n);
    
    writeContract({
      address: FACTORY_ADDRESS,
      abi: ABIS.factory,
      functionName: 'createVault',
      args: [heir as Address, BigInt(inactivityPeriod), BigInt(nftCount)],
      value: totalValue,
    });
  };
}
```

### View Methods

#### `getVaultsByOwner(address owner)`
```typescript
const { data: userVaults } = useReadContract({
  address: FACTORY_ADDRESS,
  abi: ABIS.factory,
  functionName: 'getVaultsByOwner',
  args: [userAddress],
});
```

#### `getVaultDetails(address vault)`
```typescript
interface VaultDetails {
  owner: Address;
  heir: Address;
  inactivityPeriod: bigint;
  lastActiveTimestamp: bigint;
  executed: boolean;
  ethBalance: bigint;
  canDistribute: boolean;
}

const { data: vaultDetails } = useReadContract<VaultDetails>({
  address: FACTORY_ADDRESS,
  abi: ABIS.factory,
  functionName: 'getVaultDetails',
  args: [vaultAddress],
});
```

### Fee Management

#### `calculateCreationFee(uint256 depositAmount)`
```typescript
const { data: creationFee } = useReadContract({
  address: FACTORY_ADDRESS,
  abi: ABIS.factory,
  functionName: 'calculateCreationFee',
  args: [parseEther('1.0')],
});
```

---

## 🏦 NatecinVault Contract

**Purpose**: Individual vault for storing crypto assets with automated inheritance

### Key Features
- Multi-asset support (ETH, ERC20, ERC721, ERC1155)
- Inactivity-based distribution
- NFT fee management
- Emergency withdrawal

### Asset Management Methods

#### `depositETH()`
```typescript
function DepositETH({ vaultAddress }: { vaultAddress: Address }) {
  const { writeContract } = useWriteContract();

  const handleDeposit = (amount: string) => {
    writeContract({
      address: vaultAddress,
      abi: ABIS.vault,
      functionName: 'depositETH',
      value: parseEther(amount),
    });
  };

  return <button onClick={() => handleDeposit('0.5')}>Deposit 0.5 ETH</button>;
}
```

#### `depositERC20(address token, uint256 amount)`
```typescript
import { approve } from 'wagmi/actions';
import { config } from './wagmi';

function DepositERC20({ vaultAddress }: { vaultAddress: Address }) {
  const handleDeposit = async (tokenAddress: string, amount: string) => {
    // First approve token spending
    await approve(config, {
      address: tokenAddress as Address,
      abi: er20Abi,
      spender: vaultAddress,
      amount: parseUnits(amount, 18),
    });

    // Then deposit
    writeContract({
      address: vaultAddress,
      abi: ABIS.vault,
      functionName: 'depositERC20',
      args: [tokenAddress as Address, parseUnits(amount, 18)],
    });
  };
}
```

#### `depositERC721(address collection, uint256 tokenId)`
```typescript
function DepositNFT({ vaultAddress }: { vaultAddress: Address }) {
  const handleDeposit = (collection: string, tokenId: string) => {
    writeContract({
      address: vaultAddress,
      abi: ABIS.vault,
      functionName: 'depositERC721',
      args: [collection as Address, BigInt(tokenId)],
    });
  };
}
```

### Vault Management

#### `updateActivity()`
```typescript
function UpdateActivity({ vaultAddress }: { vaultAddress: Address }) {
  const { writeContract } = useWriteContract();

  const handleUpdate = () => {
    writeContract({
      address: vaultAddress,
      abi: ABIS.vault,
      functionName: 'updateActivity',
    });
  };

  return <button onClick={handleUpdate}>Update Last Activity</button>;
}
```

#### `setHeir(address newHeir)`
```typescript
function UpdateHeir({ vaultAddress }: { vaultAddress: Address }) {
  const handleHeirUpdate = (newHeir: string) => {
    writeContract({
      address: vaultAddress,
      abi: ABIS.vault,
      functionName: 'setHeir',
      args: [newHeir as Address],
    });
  };
}
```

### View Methods

#### `getVaultSummary()`
```typescript
interface VaultSummary {
  owner: Address;
  heir: Address;
  inactivityPeriod: bigint;
  lastActiveTimestamp: bigint;
  executed: boolean;
  ethBalance: bigint;
  erc20Count: number;
  erc721Count: number;
  erc1155Count: number;
  canDistribute: boolean;
  timeUntilDistribution: bigint;
}

const { data: summary } = useReadContract<VaultSummary>({
  address: vaultAddress,
  abi: ABIS.vault,
  functionName: 'getVaultSummary',
});
```

#### `canDistribute()`
```typescript
const { data: canDistribute } = useReadContract({
  address: vaultAddress,
  abi: ABIS.vault,
  functionName: 'canDistribute',
});
```

#### `timeUntilDistribution()`
```typescript
const { data: timeLeft } = useReadContract({
  address: vaultAddress,
  abi: ABIS.vault,
  functionName: 'timeUntilDistribution',
});

// Convert to human-readable format
const formatTimeLeft = (seconds: bigint) => {
  const days = Number(seconds) / 86400;
  return `${days.toFixed(1)} days`;
};
```

### Fee Management

#### `topUpFeeDeposit()`
```typescript
function TopUpFee({ vaultAddress }: { vaultAddress: Address }) {
  const handleTopUp = (amount: string) => {
    writeContract({
      address: vaultAddress,
      abi: ABIS.vault,
      functionName: 'topUpFeeDeposit',
      value: parseEther(amount),
    });
  };
}
```

---

## 📋 VaultRegistry Contract

**Purpose**: Registry for tracking vaults and managing automated distribution via Gelato

### Key Features
- Vault registration and tracking
- Automated distribution coordination
- Fee collection management
- Batch processing

### Core Methods

#### `getDistributableVaults()`
```typescript
const { data: distributableVaults } = useReadContract<Address[]>({
  address: REGISTRY_ADDRESS,
  abi: ABIS.registry,
  functionName: 'getDistributableVaults',
});

// Display vaults ready for distribution
function VaultList() {
  return (
    <div>
      {distributableVaults?.map((vault) => (
        <div key={vault}>
          Vault: {vault} - Ready for distribution
        </div>
      ))}
    </div>
  );
}
```

#### `getTotalVaults()`
```typescript
const { data: totalVaults } = useReadContract({
  address: REGISTRY_ADDRESS,
  abi: ABIS.registry,
  functionName: 'getTotalVaults',
});
```

#### `getVaults(uint256 offset, uint256 limit)`
```typescript
function VaultPagination({ page = 0, pageSize = 10 }) {
  const { data: vaults } = useReadContract<Address[]>({
    address: REGISTRY_ADDRESS,
    abi: ABIS.registry,
    functionName: 'getVaults',
    args: [BigInt(page * pageSize), BigInt(pageSize)],
  });

  return (
    <div>
      {vaults?.map((vault) => (
        <VaultCard key={vault} address={vault} />
      ))}
    </div>
  );
}
```

### Vault Information

#### `vaultInfo(address vault)`
```typescript
interface VaultInfo {
  owner: Address;
  heir: Address;
  active: boolean;
}

const { data: info } = useReadContract<VaultInfo>({
  address: REGISTRY_ADDRESS,
  abi: ABIS.registry,
  functionName: 'vaultInfo',
  args: [vaultAddress],
});
```

---

## 🔄 Complete Integration Example

### React Component for Vault Management

```typescript
import { useAccount, useReadContract, useWriteContract } from 'wagmi';
import { ABIS, FACTORY_ADDRESS } from './contracts/abi';

function VaultManager() {
  const { address } = useAccount();
  const { writeContract } = useWriteContract();

  // Get user's vaults
  const { data: userVaults } = useReadContract({
    address: FACTORY_ADDRESS,
    abi: ABIS.factory,
    functionName: 'getVaultsByOwner',
    args: [address],
  });

  // Create new vault
  const createVault = (heir: string, inactivityPeriod: number, nftCount: number) => {
    writeContract({
      address: FACTORY_ADDRESS,
      abi: ABIS.factory,
      functionName: 'createVault',
      args: [heir as Address, BigInt(inactivityPeriod), BigInt(nftCount)],
      value: parseEther('1.0'),
    });
  };

  // Deposit ETH to vault
  const depositToVault = (vaultAddress: string, amount: string) => {
    writeContract({
      address: vaultAddress as Address,
      abi: ABIS.vault,
      functionName: 'depositETH',
      value: parseEther(amount),
    });
  };

  return (
    <div>
      <h2>My Vaults</h2>
      {userVaults?.map((vault) => (
        <VaultCard 
          key={vault} 
          address={vault}
          onDeposit={(amount) => depositToVault(vault, amount)}
        />
      ))}
      
      <CreateVaultForm onCreate={createVault} />
    </div>
  );
}

function VaultCard({ address, onDeposit }: { 
  address: Address; 
  onDeposit: (amount: string) => void;
}) {
  const { data: summary } = useReadContract({
    address,
    abi: ABIS.vault,
    functionName: 'getVaultSummary',
  });

  return (
    <div className="vault-card">
      <h3>Vault: {address}</h3>
      <p>Balance: {formatEther(summary?.ethBalance || 0n)} ETH</p>
      <p>Heir: {summary?.heir}</p>
      <p>Status: {summary?.executed ? 'Distributed' : 'Active'}</p>
      <button onClick={() => onDeposit('0.1')}>
        Deposit 0.1 ETH
      </button>
    </div>
  );
}
```

### Error Handling

```typescript
import { useWriteContract } from 'wagmi';
import { BaseError } from 'viem';

function VaultOperations() {
  const { writeContract, error } = useWriteContract();

  const handleOperation = () => {
    try {
      writeContract({
        address: FACTORY_ADDRESS,
        abi: ABIS.factory,
        functionName: 'createVault',
        args: [heir, inactivityPeriod, nftCount],
        value: depositAmount,
      });
    } catch (err) {
      if (err instanceof BaseError) {
        // Handle specific contract errors
        if (err.shortMessage.includes('InsufficientValue')) {
          alert('Insufficient funds sent');
        } else if (err.shortMessage.includes('InvalidPeriod')) {
          alert('Invalid inactivity period');
        }
      }
    }
  };

  return (
    <div>
      {error && <div className="error">{error.message}</div>}
      <button onClick={handleOperation}>Create Vault</button>
    </div>
  );
}
```

---

## 📊 Event Listening

### Listening for Contract Events

```typescript
import { useWatchContractEvent } from 'wagmi';

function VaultEventListener() {
  // Listen for new vault creations
  useWatchContractEvent({
    address: FACTORY_ADDRESS,
    abi: ABIS.factory,
    eventName: 'VaultCreated',
    onLogs: (logs) => {
      logs.forEach((log) => {
        console.log('New vault created:', {
          vault: log.args.vault,
          owner: log.args.owner,
          heir: log.args.heir,
        });
      });
    },
  });

  // Listen for asset deposits
  useWatchContractEvent({
    address: vaultAddress,
    abi: ABIS.vault,
    eventName: 'ETHDeposited',
    onLogs: (logs) => {
      logs.forEach((log) => {
        console.log('ETH deposited:', {
          from: log.args.from,
          amount: formatEther(log.args.amount),
        });
      });
    },
  });
}
```

---

## 🔧 Advanced Usage

### Multi-Call for Batch Operations

```typescript
import { useReadContracts } from 'wagmi';

function BatchVaultQueries({ vaultAddresses }: { vaultAddresses: Address[] }) {
  const { data: results } = useReadContracts({
    contracts: vaultAddresses.map((address) => ({
      address,
      abi: ABIS.vault,
      functionName: 'getVaultSummary',
    })),
  });

  return (
    <div>
      {results?.map((result, index) => (
        <VaultSummary 
          key={vaultAddresses[index]} 
          summary={result.result} 
        />
      ))}
    </div>
  );
}
```

### Fee Calculation Utility

```typescript
// utils/fees.ts
import { readContract } from 'wagmi/actions';
import { config } from '../wagmi';
import { ABIS, FACTORY_ADDRESS } from '../contracts/abi';

export async function calculateTotalCreationFee(
  depositAmount: string,
  estimatedNFTCount: number
) {
  const [creationFee, nftFee] = await Promise.all([
    readContract(config, {
      address: FACTORY_ADDRESS,
      abi: ABIS.factory,
      functionName: 'calculateCreationFee',
      args: [parseEther(depositAmount)],
    }),
    readContract(config, {
      address: FACTORY_ADDRESS,
      abi: ABIS.factory,
      functionName: 'calculateMinNFTFee',
      args: [BigInt(estimatedNFTCount)],
    }),
  ]);

  return {
    total: creationFee + nftFee,
    creationFee,
    nftFee,
  };
}
```

---

## 🚀 Deployment Checklist

### Frontend Integration Steps

1. **ABI Setup (on smart-contract repo, not frontend)**
   ```bash
   # Compile contracts to generate ABIs
   forge build
   
   # Verify ABIs are in /out/ directory
   ls out/NatecinFactory.sol/NatecinFactory.json
   ```

2. **Contract Addresses**
   ```typescript
   // Update with your deployed addresses
   export const CONTRACT_ADDRESSES = {
     factory: "0x...",
     registry: "0x...",
   } as const;
   ```

3. **Wagmi Configuration**
   ```typescript
   import { createConfig, http } from 'wagmi';
   import { mainnet, sepolia } from 'wagmi/chains';
   
   export const config = createConfig({
     chains: [mainnet, sepolia],
     transports: {
       [mainnet.id]: http(),
       [sepolia.id]: http(),
     },
   });
   ```

4. **Type Safety**
   ```typescript
   // Generate types with viem
   npx viem type --config ./contracts/types.ts
   ```

---

## 🐛 Common Issues & Solutions

### Error: "InsufficientValue"
**Cause**: Not enough ETH sent for vault creation
**Solution**: Calculate fees before transaction

```typescript
const fees = await calculateTotalCreationFee(depositAmount, nftCount);
const totalValue = parseEther(depositAmount) + fees.total;
```

### Error: "Unauthorized"
**Cause**: User is not the vault owner
**Solution**: Verify ownership before operations

```typescript
const { data: owner } = useReadContract({
  address: vaultAddress,
  abi: ABIS.vault,
  functionName: 'owner',
});

// Only allow owner to perform operations
if (owner === userAddress) {
  // Proceed with operation
}
```

### Error: "AlreadyExecuted"
**Cause**: Vault has already distributed assets
**Solution**: Check vault status before operations

```typescript
const { data: executed } = useReadContract({
  address: vaultAddress,
  abi: ABIS.vault,
  functionName: 'executed',
});

if (!executed) {
  // Proceed with operation
}
```

---

## 📚 Additional Resources

- [wagmi Documentation](https://wagmi.sh/)
- [viem Documentation](https://viem.sh/)
- [EIP-1167 Minimal Proxy Standard](https://eips.ethereum.org/EIPS/eip-1167)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/)

---

*This guide provides comprehensive examples for integrating NATECIN smart contracts with modern React applications using wagmi and viem.*
