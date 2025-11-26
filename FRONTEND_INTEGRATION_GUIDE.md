# NATECIN Frontend Integration Guide

> **Complete wagmi & viem integration for NATECIN smart contracts**

---

## 🚀 Quick Start

### Installation

```bash
npm install wagmi viem @tanstack/react-query
npm install @wagmi/core @wagmi/connectors
```

---

## ⚙️ wagmi Configuration with JSON ABIs

```typescript
// wagmi.config.ts
import { http, createConfig } from 'wagmi';
import { sepolia } from 'wagmi/chains';
import { injected, metaMask, walletConnect } from 'wagmi/connectors';
import { CONTRACTS, ABIS } from './config/contracts';

export const config = createConfig({
  chains: [sepolia],
  connectors: [
    injected(),
    metaMask(),
    walletConnect({ projectId: 'your-project-id' }),
  ],
  transports: {
    [sepolia.id]: http('https://sepolia.infura.io/v3/YOUR_INFURA_ID'),
  },
});
```

---

## 📝 Contract ABIs

### Using JSON Import (Recommended Method)

```typescript
// config/contracts.ts
import FACTORY_JSON from '../../out/NatecinFactory.sol/NatecinFactory.json';
import VAULT_JSON from '../../out/NatecinVault.sol/NatecinVault.json';
import REGISTRY_JSON from '../../out/VaultRegistry.sol/VaultRegistry.json';

export const CONTRACTS = {
  FACTORY: "0x65ac91c0f205653e7387B20c8392dbac5A48Da3B",
  REGISTRY: "0xcE5DCDd570a8AE19479c02fe368d92778a2AC4E8",
  IMPLEMENTATION: "0x6da475642a3fc043A697E3c2c2174b4DDdA81dc8"
} as const;

export const ABIS = {
  FACTORY: FACTORY_JSON.abi,
  VAULT: VAULT_JSON.abi,
  REGISTRY: REGISTRY_JSON.abi,
} as const;

// Type-safe contract interfaces
export type FactoryABI = typeof FACTORY_JSON.abi;
export type VaultABI = typeof VAULT_JSON.abi;
export type RegistryABI = typeof REGISTRY_JSON.abi;
```

### Alternative: Extract ABI Only

If you prefer to import only the ABI without metadata:

```bash
# Create abi-only files
jq '.abi' out/NatecinFactory.sol/NatecinFactory.json > src/abis/factory.json
jq '.abi' out/NatecinVault.sol/NatecinVault.json > src/abis/vault.json
jq '.abi' out/VaultRegistry.sol/VaultRegistry.json > src/abis/registry.json
```

```typescript
// Using abi-only files
import factoryABI from '../abis/factory.json';
import vaultABI from '../abis/vault.json';

export const ABIS = {
  FACTORY: factoryABI,
  VAULT: vaultABI,
} as const;
```

### Key Functions Reference

**Factory Contract:**
- `getVaultsByOwner(address)` - Get all vaults by owner
- `getVaultsByHeir(address)` - Get vaults where user is beneficiary  
- `getVaultDetails(address)` - Complete vault info without ABI
- `getVaults(offset, limit)` - Paginated vault list
- `createVault(address, uint256)` payable - Create new vault
- `totalVaults()` - Total vault count

**Vault Contract:**
- `getVaultSummary()` - Complete vault status
- `calculateNFTFee()` - Required fee for NFT distribution
- `canDistribute()` - Check if ready for distribution
- `updateActivity()` - Reset inactivity timer
- `setHeir(address)` - Update beneficiary
- `depositERC20(address, uint256)` - Deposit tokens
- `depositERC721(address, uint256)` - Deposit NFTs
- `topUpFeeDeposit()` payable - Add ETH for NFT fees
- `emergencyWithdraw()` - Close vault and withdraw all

**Registry Contract:**
- `checker()` - Get vaults ready for automation
- `executeBatch(address[], uint256)` - Batch distribution
- `getDistributableVaults()` - Manual vault check
- `vaultInfo(address)` - Vault registration status
});
```

---

## 📝 Contract ABIs

### Using JSON Import (Recommended Method)

```typescript
// config/contracts.ts
import FACTORY_JSON from '../../out/NatecinFactory.sol/NatecinFactory.json';
import VAULT_JSON from '../../out/NatecinVault.sol/NatecinVault.json';
import REGISTRY_JSON from '../../out/VaultRegistry.sol/VaultRegistry.json';

export const CONTRACTS = {
  FACTORY: "0x65ac91c0f205653e7387B20c8392dbac5A48Da3B",
  REGISTRY: "0xcE5DCDd570a8AE19479c02fe368d92778a2AC4E8",
  IMPLEMENTATION: "0x6da475642a3fc043A697E3c2c2174b4DDdA81dc8"
} as const;

export const ABIS = {
  FACTORY: FACTORY_JSON.abi,
  VAULT: VAULT_JSON.abi,
  REGISTRY: REGISTRY_JSON.abi,
} as const;

// Type-safe contract interfaces
export type FactoryABI = typeof FACTORY_JSON.abi;
export type VaultABI = typeof VAULT_JSON.abi;
export type RegistryABI = typeof REGISTRY_JSON.abi;
```

### Alternative: Extract ABI Only

If you prefer to import only the ABI without metadata:

```bash
# Create abi-only files
jq '.abi' out/NatecinFactory.sol/NatecinFactory.json > src/abis/factory.json
jq '.abi' out/NatecinVault.sol/NatecinVault.json > src/abis/vault.json
jq '.abi' out/VaultRegistry.sol/VaultRegistry.json > src/abis/registry.json
```

```typescript
// Using abi-only files
import factoryABI from '../abis/factory.json';
import vaultABI from '../abis/vault.json';

export const ABIS = {
  FACTORY: factoryABI,
  VAULT: vaultABI,
} as const;
```

### Key Functions Reference

**Factory Contract:**
- `getVaultsByOwner(address)` - Get all vaults by owner
- `getVaultsByHeir(address)` - Get vaults where user is beneficiary  
- `getVaultDetails(address)` - Complete vault info without ABI
- `getVaults(offset, limit)` - Paginated vault list
- `createVault(address, uint256)` payable - Create new vault
- `totalVaults()` - Total vault count

**Vault Contract:**
- `getVaultSummary()` - Complete vault status
- `calculateNFTFee()` - Required fee for NFT distribution
- `canDistribute()` - Check if ready for distribution
- `updateActivity()` - Reset inactivity timer
- `setHeir(address)` - Update beneficiary
- `depositERC20(address, uint256)` - Deposit tokens
- `depositERC721(address, uint256)` - Deposit NFTs
- `topUpFeeDeposit()` payable - Add ETH for NFT fees
- `emergencyWithdraw()` - Close vault and withdraw all
```

---

## 🔄 Updated Hooks with JSON Import

### 1. useVaults Hook (Updated)

```typescript
// hooks/useVaults.ts
import { useReadContract, useAccount } from 'wagmi';
import { formatEther } from 'viem';
import { CONTRACTS, ABIS } from '../config/contracts';

export function useVaults() {
  const { address } = useAccount();

  const { data: vaults, isLoading } = useReadContract({
    address: CONTRACTS.FACTORY,
    abi: ABIS.FACTORY,
    functionName: 'getVaultsByOwner',
    args: address ? [address] : undefined,
    query: {
      enabled: !!address,
    },
  });

  const { data: vaultDetails } = useReadContract({
    address: CONTRACTS.FACTORY,
    abi: ABIS.FACTORY,
    functionName: 'getVaults',
    args: [0, 100],
  });
  // Read Functions
  {
    inputs: [],
    name: "owner",
    outputs: [{ name: "", type: "address" }],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "heir", 
    outputs: [{ name: "", type: "address" }],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "inactivityPeriod",
    outputs: [{ name: "", type: "uint256" }],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "lastActiveTimestamp",
    outputs: [{ name: "", type: "uint256" }],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "executed",
    outputs: [{ name: "", type: "bool" }],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "canDistribute",
    outputs: [{ name: "", type: "bool" }],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "calculateNFTFee",
    outputs: [{ name: "", type: "uint256" }],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "feeDeposit",
    outputs: [{ name: "", type: "uint256" }],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "feeRequired",
    outputs: [{ name: "", type: "uint256" }],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "hasNonFungibleAssets",
    outputs: [{ name: "", type: "bool" }],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "getVaultSummary",
    outputs: [
      { name: "owner", type: "address" },
      { name: "heir", type: "address" },
      { name: "inactivityPeriod", type: "uint256" },
      { name: "lastActiveTimestamp", type: "uint256" },
      { name: "executed", type: "bool" },
      { name: "ethBalance", type: "uint256" },
      { name: "erc20Count", type: "uint256" },
      { name: "erc721Count", type: "uint256" },
      { name: "erc1155Count", type: "uint256" },
      { name: "timeUntilDistribution", type: "uint256" },
      { name: "canDistribute", type: "bool" }
    ],
    stateMutability: "view",
    type: "function",
  },
  // Write Functions
  {
    inputs: [{ name: "newHeir", type: "address" }],
    name: "setHeir",
    outputs: [],
    stateMutability: "nonpayable",
    type: "function",
  },
  {
    inputs: [{ name: "newPeriod", type: "uint256" }],
    name: "setInactivityPeriod", 
    outputs: [],
    stateMutability: "nonpayable",
    type: "function",
  },
  {
    inputs: [],
    name: "updateActivity",
    outputs: [],
    stateMutability: "nonpayable",
    type: "function",
  },
  {
    inputs: [
      { name: "token", type: "address" },
      { name: "amount", type: "uint256" }
    ],
    name: "depositERC20",
    outputs: [],
    stateMutability: "nonpayable",
    type: "function",
  },
  {
    inputs: [
      { name: "collection", type: "address" },
      { name: "tokenId", type: "uint256" }
    ],
    name: "depositERC721",
    outputs: [],
    stateMutability: "nonpayable",
    type: "function",
  },
  {
    inputs: [
      { name: "collection", type: "address" },
      { name: "id", type: "uint256" },
      { name: "amount", type: "uint256" },
      { name: "data", type: "bytes" }
    ],
    name: "depositERC1155",
    outputs: [],
    stateMutability: "nonpayable",
    type: "function",
  },
  {
    inputs: [],
    name: "topUpFeeDeposit",
    outputs: [],
    stateMutability: "payable",
    type: "function",
  },
  {
    inputs: [],
    name: "emergencyWithdraw",
    outputs: [],
    stateMutability: "nonpayable",
    type: "function",
  },
  // Events
  {
    anonymous: false,
    inputs: [
      { indexed: true, name: "heir", type: "address" },
      { indexed: true, name: "oldHeir", type: "address" },
      { name: "timestamp", type: "uint256" }
    ],
    name: "HeirUpdated",
    type: "event",
  },
  {
    anonymous: false,
    inputs: [
      { name: "newTimestamp", type: "uint256" }
    ],
    name: "ActivityUpdated",
    type: "event",
  },
  {
    anonymous: false,
    inputs: [
      { indexed: true, name: "from", type: "address" },
      { name: "amount", type: "uint256" }
    ],
    name: "ETHDeposited",
    type: "event",
  },
] as const;
```

---

## 🎯 React Hooks & Components

### 1. useVaults Hook

```typescript
// hooks/useVaults.ts
import { useReadContract, useAccount } from 'wagmi';
import { formatEther } from 'viem';
import { CONTRACTS, ABIS } from '../config/contracts';

export function useVaults() {
  const { address } = useAccount();

  const { data: vaults, isLoading } = useReadContract({
    address: CONTRACTS.FACTORY,
    abi: ABIS.FACTORY,
    functionName: 'getVaultsByOwner',
    args: address ? [address] : undefined,
    query: {
      enabled: !!address,
    },
  });

  const { data: vaultDetails } = useReadContract({
    address: CONTRACTS.FACTORY,
    abi: ABIS.FACTORY,
    functionName: 'getVaults',
    args: [0, 100],
  });

  return {
    vaults: vaults || [],
    vaultDetails: vaultDetails || { vaults: [], total: 0 },
    isLoading,
  };
}
```

### 2. Vault Creation Component

```typescript
// components/CreateVault.tsx
import { useState } from 'react';
import { useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
import { parseEther, formatEther } from 'viem';
import { CONTRACTS, FACTORY_ABI } from '../contracts';

export function CreateVault() {
  const [heir, setHeir] = useState('');
  const [inactivityDays, setInactivityDays] = useState(90);
  const [ethAmount, setEthAmount] = useState('1');

  const { writeContract, isPending } = useWriteContract();
  const { isLoading: isConfirming } = useWaitForTransactionReceipt();

  const handleCreate = async () => {
    if (!heir || !ethAmount) return;

    try {
      writeContract({
        address: CONTRACTS.FACTORY,
        abi: FACTORY_ABI,
        functionName: 'createVault',
        args: [
          heir as `0x${string}`,
          BigInt(inactivityDays * 24 * 60 * 60) // Convert days to seconds
        ],
        value: parseEther(ethAmount),
      });
    } catch (error) {
      console.error('Failed to create vault:', error);
    }
  };

  return (
    <div className="max-w-md mx-auto p-6 bg-white rounded-lg shadow-lg">
      <h2 className="text-2xl font-bold mb-6">Create Inheritance Vault</h2>
      
      <div className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-2">Heir Address</label>
          <input
            type="text"
            value={heir}
            onChange={(e) => setHeir(e.target.value)}
            placeholder="0x..."
            className="w-full px-3 py-2 border rounded-lg"
          />
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">
            Inactivity Period: {inactivityDays} days
          </label>
          <input
            type="range"
            min="1"
            max="3650"
            value={inactivityDays}
            onChange={(e) => setInactivityDays(Number(e.target.value))}
            className="w-full"
          />
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">ETH Amount</label>
          <input
            type="number"
            value={ethAmount}
            onChange={(e) => setEthAmount(e.target.value)}
            step="0.1"
            min="0"
            className="w-full px-3 py-2 border rounded-lg"
          />
        </div>

        <button
          onClick={handleCreate}
          disabled={isPending || isConfirming || !heir || !ethAmount}
          className="w-full bg-blue-600 text-white py-2 px-4 rounded-lg hover:bg-blue-700 disabled:opacity-50"
        >
          {isPending || isConfirming ? 'Creating...' : 'Create Vault'}
        </button>
      </div>
    </div>
  );
}
```

### 3. Vault Management Component

```typescript
// components/VaultManager.tsx
import { useState } from 'react';
import { useReadContract, useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
import { formatEther, parseEther } from 'viem';
import { CONTRACTS, ABIS } from '../config/contracts';

interface VaultManagerProps {
  vaultAddress: `0x${string}`;
}

export function VaultManager({ vaultAddress }: VaultManagerProps) {
  const [tokenAmount, setTokenAmount] = useState('');
  const [tokenAddress, setTokenAddress] = useState('');
  const [topUpAmount, setTopUpAmount] = useState('');

  const { data: vault } = useReadContract({
    address: vaultAddress,
    abi: VAULT_ABI,
    functionName: 'getVaultSummary',
  });

  const { data: nftFee } = useReadContract({
    address: vaultAddress,
    abi: VAULT_ABI,
    functionName: 'calculateNFTFee',
  });

  const { data: feeDeposit } = useReadContract({
    address: vaultAddress,
    abi: VAULT_ABI,
    functionName: 'feeDeposit',
  });

  const { writeContract } = useWriteContract();
  const { isLoading: isConfirming } = useWaitForTransactionReceipt();

  const handleUpdateActivity = () => {
    writeContract({
      address: vaultAddress,
      abi: VAULT_ABI,
      functionName: 'updateActivity',
    });
  };

  const handleDepositERC20 = () => {
    if (!tokenAmount || !tokenAddress) return;
    
    writeContract({
      address: vaultAddress,
      abi: VAULT_ABI,
      functionName: 'depositERC20',
      args: [tokenAddress as `0x${string}`, parseEther(tokenAmount)],
    });
  };

  const handleTopUpFee = () => {
    if (!topUpAmount) return;
    
    writeContract({
      address: vaultAddress,
      abi: VAULT_ABI,
      functionName: 'topUpFeeDeposit',
      value: parseEther(topUpAmount),
    });
  };

  const handleEmergencyWithdraw = () => {
    if (confirm('⚠️ This will permanently close the vault. Continue?')) {
      writeContract({
        address: vaultAddress,
        abi: VAULT_ABI,
        functionName: 'emergencyWithdraw',
      });
    }
  };

  if (!vault) return <div>Loading vault...</div>;

  return (
    <div className="bg-white p-6 rounded-lg shadow-lg">
      <div className="mb-6">
        <h3 className="text-xl font-bold mb-4">Vault Details</h3>
        <div className="grid grid-cols-2 gap-4 text-sm">
          <div>
            <span className="font-medium">Owner:</span> {vault.owner}
          </div>
          <div>
            <span className="font-medium">Heir:</span> {vault.heir}
          </div>
          <div>
            <span className="font-medium">ETH Balance:</span> {formatEther(vault.ethBalance)} ETH
          </div>
          <div>
            <span className="font-medium">Status:</span>
            <span className={`ml-2 px-2 py-1 rounded text-xs ${
              vault.executed ? 'bg-red-100 text-red-800' : 
              vault.canDistribute ? 'bg-yellow-100 text-yellow-800' : 
              'bg-green-100 text-green-800'
            }`}>
              {vault.executed ? 'Distributed' : vault.canDistribute ? 'Ready' : 'Active'}
            </span>
          </div>
          <div>
            <span className="font-medium">Time Until Distribution:</span>
            <span className={`ml-2 ${
              vault.timeUntilDistribution === 0n ? 'text-red-600 font-bold' : ''
            }`}>
              {vault.timeUntilDistribution === 0n 
                ? 'READY NOW!' 
                : `${Number(vault.timeUntilDistribution) / (24 * 60 * 60)} days`
              }
            </span>
          </div>
        </div>
      </div>

      {/* NFT Fee Status */}
      {nftFee && feeDeposit && (
        <div className="mb-6 p-4 bg-yellow-50 rounded-lg">
          <h4 className="font-medium mb-2">NFT Fee Status</h4>
          <div className="text-sm space-y-1">
            <div>Required: {formatEther(nftFee)} ETH</div>
            <div>Deposited: {formatEther(feeDeposit)} ETH</div>
            <div className={`font-medium ${
              Number(feeDeposit) >= Number(nftFee) ? 'text-green-600' : 'text-red-600'
            }`}>
              Status: {Number(feeDeposit) >= Number(nftFee) ? '✅ Sufficient' : '⚠️ Insufficient'}
            </div>
          </div>
        </div>
      )}

      {/* Action Buttons */}
      <div className="space-y-3">
        <button
          onClick={handleUpdateActivity}
          disabled={isConfirming || vault.executed}
          className="w-full bg-green-600 text-white py-2 px-4 rounded hover:bg-green-700 disabled:opacity-50"
        >
          Update Activity (Reset Timer)
        </button>

        {/* ERC20 Deposit */}
        <div className="border p-4 rounded">
          <h4 className="font-medium mb-2">Deposit ERC20 Tokens</h4>
          <div className="space-y-2">
            <input
              type="text"
              value={tokenAddress}
              onChange={(e) => setTokenAddress(e.target.value)}
              placeholder="Token Address"
              className="w-full px-3 py-2 border rounded"
            />
            <input
              type="number"
              value={tokenAmount}
              onChange={(e) => setTokenAmount(e.target.value)}
              placeholder="Amount"
              className="w-full px-3 py-2 border rounded"
            />
            <button
              onClick={handleDepositERC20}
              disabled={isConfirming || vault.executed}
              className="w-full bg-blue-600 text-white py-2 px-4 rounded hover:bg-blue-700 disabled:opacity-50"
            >
              Deposit Tokens
            </button>
          </div>
        </div>

        {/* Fee Top-up */}
        <div className="border p-4 rounded">
          <h4 className="font-medium mb-2">Top Up NFT Fee Deposit</h4>
          <div className="space-y-2">
            <input
              type="number"
              value={topUpAmount}
              onChange={(e) => setTopUpAmount(e.target.value)}
              step="0.001"
              min="0"
              placeholder="ETH Amount"
              className="w-full px-3 py-2 border rounded"
            />
            <button
              onClick={handleTopUpFee}
              disabled={isConfirming || vault.executed}
              className="w-full bg-purple-600 text-white py-2 px-4 rounded hover:bg-purple-700 disabled:opacity-50"
            >
              Top Up Fee
            </button>
          </div>
        </div>

        {/* Emergency Withdraw */}
        <button
          onClick={handleEmergencyWithdraw}
          disabled={isConfirming || vault.executed}
          className="w-full bg-red-600 text-white py-2 px-4 rounded hover:bg-red-700 disabled:opacity-50"
        >
          Emergency Withdraw (Close Vault)
        </button>
      </div>
    </div>
  );
}
```

### 4. Vault List Component

```typescript
// components/VaultList.tsx
import { useVaults } from '../hooks/useVaults';
import { VaultManager } from './VaultManager';

export function VaultList() {
  const { vaults, isLoading } = useVaults();

  if (isLoading) return <div>Loading vaults...</div>;
  if (!vaults.length) return <div>No vaults found</div>;

  return (
    <div className="space-y-6">
      <h2 className="text-2xl font-bold">Your Inheritance Vaults</h2>
      {vaults.map((vault) => (
        <VaultManager key={vault} vaultAddress={vault} />
      ))}
    </div>
  );
}
```

---

## 🔄 Real-time Event Listening

```typescript
// hooks/useVaultEvents.ts
import { useWatchContractEvent } from 'wagmi';
import { CONTRACTS, FACTORY_ABI, VAULT_ABI } from '../contracts';

export function useVaultEvents() {
  // Listen for new vault creation
  useWatchContractEvent({
    address: CONTRACTS.FACTORY,
    abi: FACTORY_ABI,
    eventName: 'VaultCreated',
    onLogs: (logs) => {
      logs.forEach((log) => {
        console.log('New vault created:', {
          vault: log.args.vault,
          owner: log.args.owner,
          heir: log.args.heir,
          inactivityPeriod: log.args.inactivityPeriod,
        });
        // Update UI state, show notification, etc.
      });
    },
  });

  // Listen to specific vault events
  const watchVaultEvents = (vaultAddress: `0x${string}`) => {
    useWatchContractEvent({
      address: vaultAddress,
      abi: VAULT_ABI,
      eventName: 'AssetsDistributed',
      onLogs: (logs) => {
        logs.forEach((log) => {
          console.log('Vault distributed:', {
            heir: log.args.heir,
            timestamp: log.args.timestamp,
          });
          // Update vault status, notify heir, etc.
        });
      },
    });

    useWatchContractEvent({
      address: vaultAddress,
      abi: VAULT_ABI,
      eventName: 'ActivityUpdated',
      onLogs: (logs) => {
        logs.forEach((log) => {
          console.log('Activity updated:', log.args.newTimestamp);
          // Update countdown timer in UI
        });
      },
    });
  };

  return { watchVaultEvents };
}
```

---

## 🛡️ Error Handling

```typescript
// utils/errors.ts
export class VaultError extends Error {
  constructor(message: string, public code?: string) {
    super(message);
    this.name = 'VaultError';
  }
}

export const handleVaultError = (error: any): VaultError => {
  if (error.message?.includes('ZeroAddress')) {
    return new VaultError('Invalid address provided', 'ZERO_ADDRESS');
  }
  if (error.message?.includes('InvalidPeriod')) {
    return new VaultError('Invalid inactivity period. Must be between 1 day and 10 years', 'INVALID_PERIOD');
  }
  if (error.message?.includes('Unauthorized')) {
    return new VaultError('You are not authorized to perform this action', 'UNAUTHORIZED');
  }
  if (error.message?.includes('InsufficientFeeBalance')) {
    return new VaultError('Insufficient fee deposit for NFT distribution. Please top up your fee deposit.', 'INSUFFICIENT_FEE');
  }
  if (error.message?.includes('AlreadyExecuted')) {
    return new VaultError('This vault has already been distributed', 'ALREADY_EXECUTED');
  }
  
  return new VaultError(error.message || 'Unknown error occurred');
};

// Usage in components
const handleError = (error: any) => {
  const vaultError = handleVaultError(error);
  console.error('Vault operation failed:', vaultError);
  
  // Show user-friendly error message
  alert(vaultError.message);
  
  // Optionally track error analytics
  // trackError(vaultError);
};
```

---

## 📊 Data Fetching & Caching

```typescript
// hooks/useVaultData.ts
import { useReadContract } from 'wagmi';
import { keepPreviousData } from '@tanstack/react-query';
import { CONTRACTS, FACTORY_ABI, VAULT_ABI } from '../contracts';

export function useVaultData(vaultAddress?: `0x${string}`) {
  const vaultSummary = useReadContract({
    address: vaultAddress,
    abi: VAULT_ABI,
    functionName: 'getVaultSummary',
    query: {
      enabled: !!vaultAddress,
      staleTime: 30_000, // 30 seconds
      refetchInterval: 60_000, // Refetch every minute
    },
  });

  const nftFee = useReadContract({
    address: vaultAddress,
    abi: VAULT_ABI,
    functionName: 'calculateNFTFee',
    query: {
      enabled: !!vaultAddress,
      staleTime: 30_000,
    },
  });

  return {
    vaultSummary: vaultSummary.data,
    nftFee: nftFee.data,
    isLoading: vaultSummary.isLoading || nftFee.isLoading,
    error: vaultSummary.error || nftFee.error,
  };
}
```

---

## 🎨 UI Components Library

### Status Badge Component

```typescript
// components/StatusBadge.tsx
interface StatusBadgeProps {
  executed: boolean;
  canDistribute: boolean;
  timeUntil: bigint;
}

export function StatusBadge({ executed, canDistribute, timeUntil }: StatusBadgeProps) {
  const getStatus = () => {
    if (executed) {
      return { text: 'Distributed', color: 'bg-red-100 text-red-800' };
    }
    if (canDistribute) {
      return { text: 'Ready', color: 'bg-yellow-100 text-yellow-800' };
    }
    if (timeUntil < 7n * 24n * 60n * 60n) { // Less than 7 days
      return { text: 'Warning', color: 'bg-orange-100 text-orange-800' };
    }
    return { text: 'Active', color: 'bg-green-100 text-green-800' };
  };

  const status = getStatus();

  return (
    <span className={`px-2 py-1 rounded-full text-xs font-medium ${status.color}`}>
      {status.text}
    </span>
  );
}
```

### Countdown Timer Component

```typescript
// components/CountdownTimer.tsx
import { useState, useEffect } from 'react';

interface CountdownTimerProps {
  timeUntil: bigint;
  onExpire?: () => void;
}

export function CountdownTimer({ timeUntil, onExpire }: CountdownTimerProps) {
  const [timeLeft, setTimeLeft] = useState(Number(timeUntil));

  useEffect(() => {
    if (timeLeft <= 0) {
      onExpire?.();
      return;
    }

    const timer = setInterval(() => {
      setTimeLeft((prev) => {
        if (prev <= 1) {
          onExpire?.();
          return 0;
        }
        return prev - 1;
      });
    }, 1000);

    return () => clearInterval(timer);
  }, [timeUntil, onExpire]);

  const formatTime = (seconds: number) => {
    const days = Math.floor(seconds / (24 * 60 * 60));
    const hours = Math.floor((seconds % (24 * 60 * 60)) / (60 * 60));
    const minutes = Math.floor((seconds % (60 * 60)) / 60);
    const secs = seconds % 60;

    return `${days}d ${hours}h ${minutes}m ${secs}s`;
  };

  return (
    <div className={`font-mono text-lg ${
      timeLeft === 0 ? 'text-red-600 font-bold' : 
      timeLeft < 7 * 24 * 60 * 60 ? 'text-orange-600' : 'text-gray-600'
    }`}>
      {formatTime(timeLeft)}
    </div>
  );
}
```

---

## 🔧 Utility Functions

```typescript
// utils/vaultUtils.ts
import { formatEther } from 'viem';

export const formatVaultTime = (seconds: bigint): string => {
  const days = Number(seconds) / (24 * 60 * 60);
  return `${days.toFixed(1)} days`;
};

export const formatInactivityPeriod = (seconds: bigint): string => {
  const totalDays = Number(seconds) / (24 * 60 * 60);
  if (totalDays < 30) {
    return `${totalDays.toFixed(0)} days`;
  }
  if (totalDays < 365) {
    return `${(totalDays / 30).toFixed(1)} months`;
  }
  return `${(totalDays / 365).toFixed(1)} years`;
};

export const calculateFeeRequired = (amount: bigint, feePercent: number): bigint => {
  return (amount * BigInt(feePercent)) / 10000n;
};

export const validateHeirAddress = (address: string): boolean => {
  return /^0x[a-fA-F0-9]{40}$/.test(address) && address !== '0x0000000000000000000000000000000000000000';
};

export const validateInactivityPeriod = (days: number): boolean => {
  const seconds = days * 24 * 60 * 60;
  return seconds >= 86400 && seconds <= 315360000; // 1 day to 10 years
};
```

---

## 📱 Mobile Responsive Design

```typescript
// components/responsive/VaultCard.tsx
export function VaultCard({ vault }: { vault: any }) {
  return (
    <div className="bg-white rounded-lg shadow-md p-4 mb-4 w-full">
      {/* Mobile-first responsive grid */}
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
        <div className="col-span-full sm:col-span-2 lg:col-span-1">
          <h3 className="font-semibold text-lg truncate">{vault.address}</h3>
          <p className="text-sm text-gray-600 truncate">{vault.heir}</p>
        </div>
        
        <div className="text-center sm:text-right lg:text-left">
          <StatusBadge 
            executed={vault.executed}
            canDistribute={vault.canDistribute}
            timeUntil={vault.timeUntilDistribution}
          />
        </div>
        
        <div className="col-span-full sm:col-span-2 lg:col-span-1">
          <CountdownTimer 
            timeUntil={vault.timeUntilDistribution}
            onExpire={() => window.location.reload()}
          />
        </div>
      </div>
      
      {/* Action buttons responsive */}
      <div className="flex flex-col sm:flex-row gap-2 mt-4">
        <button className="flex-1 min-w-0">Update Activity</button>
        <button className="flex-1 min-w-0">Manage Assets</button>
        <button className="flex-1 min-w-0 text-red-600">Emergency</button>
      </div>
    </div>
  );
}
```

---

## 🚀 Deployment Tips

### Environment Configuration

```typescript
// config/environment.ts
export const ENV = {
  IS_MAINNET: import.meta.env.VITE_IS_MAINNET === 'true',
  RPC_URL: import.meta.env.VITE_RPC_URL,
  INFURA_ID: import.meta.env.VITE_INFURA_ID,
  ALCHEMY_ID: import.meta.env.VITE_ALCHEMY_ID,
} as const;

export const NETWORK_CONFIG = ENV.IS_MAINNET ? {
  chainId: 1,
  name: 'Ethereum',
  contracts: {
    FACTORY: '0x...', // Mainnet addresses
    REGISTRY: '0x...',
  }
} : {
  chainId: 11155111,
  name: 'Sepolia',
  contracts: CONTRACTS, // Testnet addresses
};
```

### Bundle Optimization

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  define: {
    'process.env': '{}',
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['wagmi', 'viem', '@tanstack/react-query'],
          contracts: ['./contracts/abis'],
        },
      },
    },
  },
});
```

---

## 🧪 Testing

### Component Testing with wagmi Testing

```typescript
// __tests__/CreateVault.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { WagmiProvider } from 'wagmi';
import { createConfig, http } from 'wagmi';
import { sepolia } from 'wagmi/chains';
import { CreateVault } from '../components/CreateVault';

const testConfig = createConfig({
  chains: [sepolia],
  transports: {
    [sepolia.id]: http(),
  },
});

const renderWithWagmi = (component: React.ReactElement) => {
  return render(
    <WagmiProvider config={testConfig}>
      {component}
    </WagmiProvider>
  );
};

describe('CreateVault', () => {
  it('should create vault with valid inputs', async () => {
    renderWithWagmi(<CreateVault />);
    
    fireEvent.change(screen.getByPlaceholderText('0x...'), {
      target: { value: '0x742d35Cc6635C0532925a3b844168675c8C44e7' }
    });
    
    fireEvent.click(screen.getByText('Create Vault'));
    
    await waitFor(() => {
      expect(screen.getByText('Creating...')).toBeInTheDocument();
    });
  });
});
```

---

## 📈 Performance Optimization

### Contract Call Optimization

```typescript
// hooks/useOptimizedVaults.ts
import { useReadContract } from 'wagmi';
import { useMemo } from 'react';

export function useOptimizedVaults(address?: `0x${string}`) {
  const { data: rawVaults } = useReadContract({
    address: CONTRACTS.FACTORY,
    abi: FACTORY_ABI,
    functionName: 'getVaultsByOwner',
    args: address ? [address] : undefined,
    query: {
      enabled: !!address,
      staleTime: 60_000, // Cache for 1 minute
    },
  });

  // Memoize processed vault data
  const processedVaults = useMemo(() => {
    if (!rawVaults) return [];
    
    return rawVaults.map((vault) => ({
      address: vault,
      shortAddress: `${vault.slice(0, 6)}...${vault.slice(-4)}`,
    }));
  }, [rawVaults]);

  return {
    vaults: processedVaults,
    rawVaults,
  };
}
```

---

## 🔗 Important Integration Notes

### 1. **NFT Fee Deposits** ⚠️
- Always check `calculateNFTFee()` after depositing NFTs
- Use `topUpFeeDeposit()` to ensure sufficient fees for distribution
- Fee deposit is separate from main ETH balance

### 2. **Gas Optimization**
- Use EIP-1167 clone pattern (already implemented in contracts)
- Batch operations when possible
- Cache read operations with appropriate staleTime

### 3. **Error Boundaries**
- Wrap components in error boundaries
- Handle transaction failures gracefully
- Provide retry mechanisms

### 4. **Mobile Considerations**
- Responsive design for all screen sizes
- Touch-friendly button sizes
- Simplified forms for mobile input

### 5. **Security**
- Never store private keys
- Validate all user inputs
- Use proper wallet connection methods

---

## 🎯 Quick Integration Checklist

- [ ] Configure wagmi with correct chain and RPC
- [ ] Set up contract ABIs and addresses
- [ ] Implement wallet connection
- [ ] Create vault creation flow
- [ ] Add vault management interface
- [ ] Implement NFT fee handling
- [ ] Add real-time event listeners
- [ ] Include error handling
- [ ] Make responsive design
- [ ] Add loading states
- [ ] Test on testnet first

---

**🎉 Your frontend is now ready to integrate with NATECIN smart contracts!**

For support and updates, visit the project repository or join the community Discord.
