# Frontend Integration Guide for NATECIN Vault System

> **Using existing wagmi config & ABI JSON imports to implement vault features**

> **Prerequisites**: wagmi configured with Lisk Sepolia • ABI JSON in `lib/abi/*` • UI themes ready

---

## 🎯 Integration Methods for Existing Infrastructure

### 1. **Using Existing ABIs in React Components**

Since you already have ABIs imported in `lib/abi/*`, here's how to use them:

```typescript
// Example: Access existing ABIs
import FACTORY_ABI from '../lib/abi/NatecinFactory.json';
import VAULT_ABI from '../lib/abi/NatecinVault.json';
import REGISTRY_ABI from '../lib/abi/VaultRegistry.json';

// Your existing wagmi hooks with these ABIs
import { useReadContract, useWriteContract } from 'wagmi';
import { FACTORY_ADDRESS, REGISTRY_ADDRESS } from '../config/contracts'; // Assuming you have this
```

### 2. **Factory Contract Integration Methods**

```typescript
// Method: Create Vault with Multi-Heir Support
export function useCreateVault() {
  const { writeContract } = useWriteContract();
  
  const createVault = (heirs: string[], inactivityPeriod: number, ethAmount: string) => {
    return writeContract({
      address: FACTORY_ADDRESS,
      abi: FACTORY_ABI.abi,
      functionName: 'createVault',
      args: [heirs[0] as `0x${string}`, BigInt(inactivityPeriod)], // Primary heir
      value: parseEther(ethAmount),
    });
  };
  
  return { createVault };
}

// Method: Get Vaults by Owner
export function useGetVaultsByOwner(ownerAddress: string) {
  return useReadContract({
    address: FACTORY_ADDRESS,
    abi: FACTORY_ABI.abi,
    functionName: 'getVaultsByOwner',
    args: [ownerAddress as `0x${string}`],
  });
}

// Method: Paginated Vault List
export function useGetVaults(offset: number = 0, limit: number = 10) {
  return useReadContract({
    address: FACTORY_ADDRESS,
    abi: FACTORY_ABI.abi,
    functionName: 'getVaults',
    args: [BigInt(offset), BigInt(limit)],
  });
}
```

### 3. **Vault Contract Integration Methods**

```typescript
// Method: Get Complete Vault Summary
export function useVaultSummary(vaultAddress: string) {
  return useReadContract({
    address: vaultAddress as `0x${string}`,
    abi: VAULT_ABI.abi,
    functionName: 'getVaultSummary',
  });
}

// Method: ETH Deposits with Fee Calculation
export function useDepositETH() {
  const { writeContract } = useWriteContract();
  
  const depositETH = (vaultAddress: string, amount: string) => {
    return writeContract({
      address: vaultAddress as `0x${string}`,
      abi: VAULT_ABI.abi,
      functionName: 'depositETH',
      value: parseEther(amount),
    });
  };
  
  return { depositETH };
}

// Method: NFT Deposits (ERC721/ERC1155)
export function useDepositNFT() {
  const { writeContract } = useWriteContract();
  
  const depositERC721 = (vaultAddress: string, contractAddress: string, tokenId: number) => {
    return writeContract({
      address: vaultAddress as `0x${string}`,
      abi: VAULT_ABI.abi,
      functionName: 'depositERC721',
      args: [contractAddress as `0x${string}`, BigInt(tokenId)],
    });
  };
  
  const depositERC1155 = (vaultAddress: string, contractAddress: string, tokenId: number, amount: number) => {
    return writeContract({
      address: vaultAddress as `0x${string}`,
      abi: VAULT_ABI.abi,
      functionName: 'depositERC1155',
      args: [contractAddress as `0x${string}`, BigInt(tokenId), BigInt(amount)],
    });
  };
  
  return { depositERC721, depositERC1155 };
}

// Method: Update Activity Timer
export function useUpdateActivity() {
  const { writeContract } = useWriteContract();
  
  const updateActivity = (vaultAddress: string) => {
    return writeContract({
      address: vaultAddress as `0x${string}`,
      abi: VAULT_ABI.abi,
      functionName: 'updateActivity',
    });
  };
  
  return { updateActivity };
}

// Method: Calculate NFT Distribution Fees
export function useCalculateNFTFee(vaultAddress: string) {
  return useReadContract({
    address: vaultAddress as `0x${string}`,
    abi: VAULT_ABI.abi,
    functionName: 'calculateNFTFee',
  });
}
```

### 4. **Vault Registry Integration Methods**

```typescript
// Method: Get Total Vaults Count
export function useGetTotalVaults() {
  return useReadContract({
    address: REGISTRY_ADDRESS,
    abi: REGISTRY_ABI.abi,
    functionName: 'getTotalVaults',
  });
}

// Method: Get Vaults Ready for Distribution
export function useGetDistributableVaults() {
  return useReadContract({
    address: REGISTRY_ADDRESS,
    abi: REGISTRY_ABI.abi,
    functionName: 'getDistributableVaults',
  });
}

// Method: Batch Distribution Operations
export function useExecuteBatch() {
  const { writeContract } = useWriteContract();
  
  const executeBatch = (vaultList: string[], nextIndex: number) => {
    return writeContract({
      address: REGISTRY_ADDRESS,
      abi: REGISTRY_ABI.abi,
      functionName: 'executeBatch',
      args: [vaultList as `0x${string}`[], BigInt(nextIndex)],
    });
  };
  
  return { executeBatch };
}
```

---

## 🎨 UI Integration Examples with Existing Theme

### 1. **Vault Creation Form Integration**

```typescript
// components/VaultCreationForm.tsx
import React, { useState } from 'react';
import { useAccount } from 'wagmi';
import { useCreateVault } from '../hooks/useVaultCreation'; // Import from above
import { parseEther } from 'viem';

export function VaultCreationForm() {
  const { address, isConnected } = useAccount();
  const { createVault, isPending, isConfirming } = useCreateVault();
  
  const [formData, setFormData] = useState({
    heirs: [''], // Multi-heir support
    inactivityDays: 30,
    ethAmount: '0.01',
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const inactivityPeriod = formData.inactivityDays * 24 * 60 * 60; // Convert to seconds
    
    try {
      await createVault(formData.heirs.filter(h => h), inactivityPeriod, formData.ethAmount);
      // Handle success - your existing UI theme will show notifications
    } catch (error) {
      // Your existing error handling
      console.error('Vault creation failed:', error);
    }
  };

  const addHeir = () => {
    setFormData(prev => ({ ...prev, heirs: [...prev.heirs, ''] }));
  };

  const updateHeir = (index: number, value: string) => {
    setFormData(prev => ({
      ...prev,
      heirs: prev.heirs.map((h, i) => i === index ? value : h)
    }));
  };

  return (
    <div className="vault-creation-form">
      <h3 className="form-title">Create New Vault</h3>
      
      <div className="form-section">
        <label className="form-label">Heirs</label>
        {formData.heirs.map((heir, index) => (
          <input
            key={index}
            className="form-input"
            type="text"
            value={heir}
            onChange={(e) => updateHeir(index, e.target.value)}
            placeholder={`Heir ${index + 1} address`}
          />
        ))}
        <button 
          type="button" 
          onClick={addHeir}
          className="add-heir-btn"
        >
          + Add Heir
        </button>
      </div>

      <div className="form-section">
        <label className="form-label">Inactivity Period (days)</label>
        <input
          className="form-input"
          type="number"
          value={formData.inactivityDays}
          onChange={(e) => setFormData(prev => ({ ...prev, inactivityDays: parseInt(e.target.value) }))}
          min="1"
          max="3650"
        />
      </div>

      <div className="form-section">
        <label className="form-label">Initial ETH Deposit</label>
        <input
          className="form-input"
          type="number"
          step="0.001"
          value={formData.ethAmount}
          onChange={(e) => setFormData(prev => ({ ...prev, ethAmount: e.target.value }))}
          min="0.001"
        />
      </div>

      <button 
        type="submit" 
        onClick={handleSubmit}
        disabled={isPending || isConfirming}
        className="submit-btn" // Using your existing theme classes
      >
        {isPending ? 'Creating...' : isConfirming ? 'Confirming...' : 'Create Vault'}
      </button>
    </div>
  );
}
```

### 2. **Vault Dashboard Integration**

```typescript
// components/VaultDashboard.tsx
import React, { useState, useEffect } from 'react';
import { useAccount } from 'wagmi';
import { useVaultSummary, useUpdateActivity } from '../hooks/useVaultManagement';
import { useDepositETH, useDepositNFT } from '../hooks/useAssetDeposits';
import { formatEther } from 'viem';
import { formatDistanceToNow } from 'date-fns';

export function VaultDashboard({ vaultAddress }: { vaultAddress: string }) {
  const { address } = useAccount();
  const { vaultSummary, isLoading, refetch } = useVaultSummary(vaultAddress);
  const { updateActivity } = useUpdateActivity();
  const { depositETH } = useDepositETH();
  const { depositERC721 } = useDepositNFT();
  
  const [timeRemaining, setTimeRemaining] = useState<string>('');
  const [depositAmount, setDepositAmount] = useState('0.1');

  // Time remaining calculation
  useEffect(() => {
    if (!vaultSummary) return;
    
    const interval = setInterval(() => {
      const now = Math.floor(Date.now() / 1000);
      const lastActivity = Number(vaultSummary.lastActivity);
      const inactivityPeriod = Number(vaultSummary.inactivityPeriod);
      const timeLeft = inactivityPeriod - (now - lastActivity);
      
      if (timeLeft <= 0) {
        setTimeRemaining('Ready for distribution');
      } else {
        const date = new Date((now + timeLeft) * 1000);
        setTimeRemaining(formatDistanceToNow(date, { addSuffix: true }));
      }
    }, 1000);
    
    return () => clearInterval(interval);
  }, [vaultSummary]);

  const handleDepositETH = async () => {
    try {
      await depositETH(vaultAddress, depositAmount);
      await refetch(); // Refresh vault data
      setDepositAmount('0.1'); // Reset form
    } catch (error) {
      console.error('ETH deposit failed:', error);
    }
  };

  const handleUpdateActivity = async () => {
    try {
      await updateActivity(vaultAddress);
      await refetch();
    } catch (error) {
      console.error('Activity update failed:', error);
    }
  };

  if (isLoading) {
    return <div className="loading-spinner">Loading vault details...</div>;
  }

  if (!vaultSummary) {
    return <div className="error-message">Vault not found</div>;
  }

  return (
    <div className="vault-dashboard">
      <div className="vault-header">
        <h2 className="vault-title">Vault Details</h2>
        <div className={`status-badge ${vaultSummary.isActive ? 'active' : 'inactive'}`}>
          {vaultSummary.isActive ? 'Active' : 'Inactive'}
        </div>
      </div>

      <div className="vault-cards">
        <div className="info-card">
          <h4>Owner</h4>
          <p className="address">{vaultSummary.owner}</p>
        </div>
        
        <div className="info-card">
          <h4>Heir</h4>
          <p className="address">{vaultSummary.heir}</p>
        </div>
        
        <div className="info-card">
          <h4>Balance</h4>
          <p className="balance">{formatEther(vaultSummary.balance)} ETH</p>
        </div>
        
        <div className="info-card">
          <h4>Time Remaining</h4>
          <p className="timer">{timeRemaining}</p>
        </div>
      </div>

      {address?.toLowerCase() === vaultSummary.owner.toLowerCase() && (
        <div className="vault-actions">
          <div className="action-section">
            <h4>Quick Actions</h4>
            <button 
              onClick={handleUpdateActivity}
              className="action-btn update-activity"
            >
              Update Activity
            </button>
          </div>

          <div className="action-section">
            <h4>Deposit ETH</h4>
            <div className="deposit-form">
              <input
                type="number"
                step="0.001"
                value={depositAmount}
                onChange={(e) => setDepositAmount(e.target.value)}
                className="deposit-input"
                placeholder="Amount"
              />
              <button 
                onClick={handleDepositETH}
                className="deposit-btn"
              >
                Deposit
              </button>
            </div>
          </div>

          <div className="action-section">
            <h4>Deposit NFT</h4>
            <NFTDepositForm vaultAddress={vaultAddress} />
          </div>
        </div>
      )}
    </div>
  );
}

// NFT Deposit Component
function NFTDepositForm({ vaultAddress }: { vaultAddress: string }) {
  const { depositERC721, depositERC1155 } = useDepositNFT();
  const [formData, setFormData] = useState({
    contractAddress: '',
    tokenId: '',
    tokenType: 'ERC721',
    amount: '1', // For ERC1155
  });

  const handleDeposit = async () => {
    try {
      if (formData.tokenType === 'ERC721') {
        await depositERC721(vaultAddress, formData.contractAddress, parseInt(formData.tokenId));
      } else {
        await depositERC1155(
          vaultAddress, 
          formData.contractAddress, 
          parseInt(formData.tokenId), 
          parseInt(formData.amount)
        );
      }
    } catch (error) {
      console.error('NFT deposit failed:', error);
    }
  };

  return (
    <div className="nft-deposit-form">
      <select
        value={formData.tokenType}
        onChange={(e) => setFormData(prev => ({ ...prev, tokenType: e.target.value }))}
        className="token-type-select"
      >
        <option value="ERC721">ERC721</option>
        <option value="ERC1155">ERC1155</option>
      </select>
      
      <input
        type="text"
        value={formData.contractAddress}
        onChange={(e) => setFormData(prev => ({ ...prev, contractAddress: e.target.value }))}
        placeholder="NFT Contract Address"
        className="contract-input"
      />
      
      <input
        type="number"
        value={formData.tokenId}
        onChange={(e) => setFormData(prev => ({ ...prev, tokenId: e.target.value }))}
        placeholder="Token ID"
        className="token-id-input"
      />
      
      {formData.tokenType === 'ERC1155' && (
        <input
          type="number"
          value={formData.amount}
          onChange={(e) => setFormData(prev => ({ ...prev, amount: e.target.value }))}
          placeholder="Amount"
          className="amount-input"
          min="1"
        />
      )}
      
      <button onClick={handleDeposit} className="deposit-nft-btn">
        Deposit NFT
      </button>
    </div>
  );
}
```

### 3. **Asset Management Integration**

```typescript
// components/AssetManagement.tsx
import React, { useState } from 'react';
import { useAccount } from 'wagmi';
import { useGetDistributableVaults, useExecuteBatch } from '../hooks/useVaultRegistry';
import { useCalculateNFTFee } from '../hooks/useVaultManagement';

export function AssetManagement() {
  const { address } = useAccount();
  const { distributableVaults, refetch } = useGetDistributableVaults();
  const { executeBatch, isPending } = useExecuteBatch();
  
  const [selectedVaults, setSelectedVaults] = useState<string[]>([]);

  const toggleVaultSelection = (vaultAddress: string) => {
    setSelectedVaults(prev => 
      prev.includes(vaultAddress) 
        ? prev.filter(v => v !== vaultAddress)
        : [...prev, vaultAddress]
    );
  };

  const handleBatchDistribution = async () => {
    if (selectedVaults.length === 0) return;
    
    try {
      await executeBatch(selectedVaults, 0);
      await refetch();
      setSelectedVaults([]);
    } catch (error) {
      console.error('Batch distribution failed:', error);
    }
  };

  return (
    <div className="asset-management">
      <h3>Asset Distribution</h3>
      
      {distributableVaults && distributableVaults.length > 0 ? (
        <div className="distributable-vaults">
          <div className="vault-list">
            {distributableVaults.map((vault, index) => (
              <div key={vault} className="vault-item">
                <input
                  type="checkbox"
                  checked={selectedVaults.includes(vault)}
                  onChange={() => toggleVaultSelection(vault)}
                  className="vault-checkbox"
                />
                <div className="vault-info">
                  <span className="vault-index">#{index + 1}</span>
                  <span className="vault-address">{vault}</span>
                </div>
                <VaultFeeInfo vaultAddress={vault} />
              </div>
            ))}
          </div>
          
          <div className="batch-actions">
            <button 
              onClick={handleBatchDistribution}
              disabled={selectedVaults.length === 0 || isPending}
              className="batch-distribute-btn"
            >
              Distribute {selectedVaults.length} Vaults
            </button>
          </div>
        </div>
      ) : (
        <div className="no-vaults">
          <p>No vaults ready for distribution</p>
        </div>
      )}
    </div>
  );
}

function VaultFeeInfo({ vaultAddress }: { vaultAddress: string }) {
  const { data: feeAmount } = useCalculateNFTFee(vaultAddress);
  
  return (
    <div className="fee-info">
      <span className="fee-label">NFT Fee:</span>
      <span className="fee-amount">
        {feeAmount ? formatEther(feeAmount) : '0'} ETH
      </span>
    </div>
  );
}
```

---

## 🔄 Transaction Management & Event Handling

### 1. **Real-time Event Integration**

```typescript
// hooks/useVaultEvents.ts
import { useWatchContractEvent } from 'wagmi';
import { FACTORY_ABI, VAULT_ABI, REGISTRY_ABI } from '../lib/abi';

export function useVaultEvents(callback: (event: any) => void) {
  // Listen to vault creation events
  useWatchContractEvent({
    address: undefined, // Factory address from config
    abi: FACTORY_ABI.abi,
    eventName: 'VaultCreated',
    onLogs: (logs) => {
      logs.forEach(log => callback({
        type: 'VaultCreated',
        data: log,
        timestamp: Date.now(),
      }));
    },
  });

  // Listen to asset deposit events
  useWatchContractEvent({
    address: undefined, // Will be set dynamically
    abi: VAULT_ABI.abi,
    eventName: 'AssetDeposited',
    onLogs: (logs) => {
      logs.forEach(log => callback({
        type: 'AssetDeposited',
        data: log,
        timestamp: Date.now(),
      }));
    },
  });

  // Listen to vault distribution events
  useWatchContractEvent({
    address: undefined,
    abi: VAULT_ABI.abi,
    eventName: 'VaultDistributed',
    onLogs: (logs) => {
      logs.forEach(log => callback({
        type: 'VaultDistributed',
        data: log,
        timestamp: Date.now(),
      }));
    },
  });
}

// Usage in component
export function EventNotifier() {
  const [events, setEvents] = useState<any[]>([]);
  
  useVaultEvents((event) => {
    setEvents(prev => [event, ...prev.slice(0, 9)]); // Keep last 10 events
    
    // Your existing notification system
    showNotification({
      type: event.type,
      message: getEventMessage(event),
      timestamp: event.timestamp,
    });
  });

  return (
    <div className="event-notifier">
      <h4>Recent Events</h4>
      {events.map((event, index) => (
        <div key={index} className="event-item">
          <span className="event-type">{event.type}</span>
          <span className="event-time">
            {new Date(event.timestamp).toLocaleTimeString()}
          </span>
        </div>
      ))}
    </div>
  );
}

function getEventMessage(event: any): string {
  switch (event.type) {
    case 'VaultCreated':
      return 'New vault created successfully';
    case 'AssetDeposited':
      return 'Assets deposited to vault';
    case 'VaultDistributed':
      return 'Vault distributed to heir';
    default:
      return 'Vault event occurred';
  }
}
```

### 2. **Gas Estimation Integration**

```typescript
// hooks/useGasEstimation.ts
import { useEstimateGas, useGasPrice } from 'wagmi';
import { parseEther } from 'viem';

export function useGasEstimation() {
  const { data: gasPrice } = useGasPrice();
  
  const estimateVaultCreation = (heirs: string[], inactivityPeriod: number, ethAmount: string) => {
    const { data: estimatedGas } = useEstimateGas({
      address: FACTORY_ADDRESS,
      abi: FACTORY_ABI.abi,
      functionName: 'createVault',
      args: [heirs[0] as `0x${string}`, BigInt(inactivityPeriod)],
      value: parseEther(ethAmount),
    });

    const gasCostETH = estimatedGas && gasPrice 
      ? formatEther(estimatedGas * gasPrice) 
      : '0';

    return {
      estimatedGas,
      gasCostETH,
    };
  };

  const estimateDepositETH = (vaultAddress: string, amount: string) => {
    const { data: estimatedGas } = useEstimateGas({
      address: vaultAddress as `0x${string}`,
      abi: VAULT_ABI.abi,
      functionName: 'depositETH',
      value: parseEther(amount),
    });

    const gasCostETH = estimatedGas && gasPrice 
      ? formatEther(estimatedGas * gasPrice) 
      : '0';

    return {
      estimatedGas,
      gasCostETH,
    };
  };

  return {
    estimateVaultCreation,
    estimateDepositETH,
    gasPrice: gasPrice ? formatEther(gasPrice) : '0',
  };
}
```

---

## 🎯 Integration Checklist

### ✅ **Required Imports**
```typescript
// Add these to your existing component files
import { useReadContract, useWriteContract, useWatchContractEvent } from 'wagmi';
import { parseEther, formatEther } from 'viem';
import FACTORY_ABI from '../lib/abi/NatecinFactory.json';
import VAULT_ABI from '../lib/abi/NatecinVault.json';
import REGISTRY_ABI from '../lib/abi/VaultRegistry.json';
```

### ✅ **Contract Addresses**
```typescript
// Add to your config file if not already present
export const CONTRACTS = {
  FACTORY: process.env.NEXT_PUBLIC_FACTORY_ADDRESS || '0x65ac91c0f205653e7387B20c8392dbac5A48Da3B',
  REGISTRY: process.env.NEXT_PUBLIC_REGISTRY_ADDRESS || '0xcE5DCDd570a8AE19479c02fe368d92778a2AC4E8',
};
```

### ✅ **Theme Integration**
- All components use your existing theme classes
- Loading states use your existing spinner components
- Error handling uses your existing notification system
- Forms use your existing input styles and validation

### ✅ **Integration Steps**
1. Copy the hook implementations to your hooks directory
2. Add the components to your components directory
3. Import the hooks in your page components
4. Integrate with your existing routing and navigation
5. Test with your existing wallet connection flow

---

## 🔧 Customization Options

### **Multi-Heir Support**
```typescript
// For future multi-heir implementation
const createMultiHeirVault = (heirs: string[], percentages: number[], inactivityPeriod: number) => {
  return writeContract({
    address: FACTORY_ADDRESS,
    abi: FACTORY_ABI.abi,
    functionName: 'createMultiHeirVault', // Future function
    args: [heirs, percentages, BigInt(inactivityPeriod)],
  });
};
```

### **Advanced Asset Management**
```typescript
// Add support for additional asset types
const depositERC20 = (vaultAddress: string, tokenAddress: string, amount: string) => {
  return writeContract({
    address: vaultAddress as `0x${string}`,
    abi: VAULT_ABI.abi,
    functionName: 'depositERC20',
    args: [tokenAddress as `0x${string}`, parseEther(amount)],
  });
};
```

---

This guide provides direct integration methods for your existing frontend infrastructure, leveraging your wagmi config, ABI imports, and UI themes to implement NATECIN vault features efficiently.

### 1. useVaultCreation Hook

```typescript
// hooks/useVaultCreation.ts
import { useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
import { parseEther } from 'viem';
import { ABIS, CONTRACTS } from '../lib/abi';

interface CreateVaultParams {
  heir: string;
  inactivityPeriod: number; // in seconds
  value: string; // ETH amount in wei
}

export function useVaultCreation() {
  const { writeContract, data: hash, isPending, error } = useWriteContract();
  
  const { 
    data: receipt, 
    isLoading: isConfirming, 
    isSuccess: isConfirmed 
  } = useWaitForTransactionReceipt({ hash });

  const createVault = ({ heir, inactivityPeriod, value }: CreateVaultParams) => {
    writeContract({
      address: CONTRACTS.FACTORY,
      abi: ABIS.FACTORY.abi,
      functionName: 'createVault',
      args: [heir as `0x${string}`, BigInt(inactivityPeriod)],
      value: parseEther(value),
    });
  };

  return {
    createVault,
    isPending,
    isConfirming,
    isConfirmed,
    hash,
    receipt,
    error,
  };
}
```

### 2. useVaultManagement Hook

```typescript
// hooks/useVaultManagement.ts
import { useReadContract, useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
import { ABIS } from '../lib/abi';
import { formatEther, parseEther } from 'viem';

interface VaultSummary {
  owner: string;
  heir: string;
  inactivityPeriod: bigint;
  lastActivity: bigint;
  balance: bigint;
  status: number;
  isActive: boolean;
  canDistribute: boolean;
}

export function useVaultSummary(vaultAddress: string) {
  const { data, error, isLoading, refetch } = useReadContract({
    address: vaultAddress as `0x${string}`,
    abi: ABIS.VAULT.abi,
    functionName: 'getVaultSummary',
  });

  const vaultSummary: VaultSummary | undefined = data ? {
    owner: data[0],
    heir: data[1],
    inactivityPeriod: data[2],
    lastActivity: data[3],
    balance: data[4],
    status: data[5],
    isActive: data[5] === 0n,
    canDistribute: data[6],
  } : undefined;

  return {
    vaultSummary,
    error,
    isLoading,
    refetch,
  };
}

export function useVaultOperations(vaultAddress: string) {
  const { writeContract, data: hash, isPending, error } = useWriteContract();
  
  const { 
    data: receipt, 
    isLoading: isConfirming, 
    isSuccess: isConfirmed 
  } = useWaitForTransactionReceipt({ hash });

  const updateActivity = () => {
    writeContract({
      address: vaultAddress as `0x${string}`,
      abi: ABIS.VAULT.abi,
      functionName: 'updateActivity',
    });
  };

  const setHeir = (newHeir: string) => {
    writeContract({
      address: vaultAddress as `0x${string}`,
      abi: ABIS.VAULT.abi,
      functionName: 'setHeir',
      args: [newHeir as `0x${string}`],
    });
  };

  return {
    updateActivity,
    setHeir,
    isPending,
    isConfirming,
    isConfirmed,
    hash,
    receipt,
    error,
  };
}
```

### 3. useAssetDeposits Hook

```typescript
// hooks/useAssetDeposits.ts
import { useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
import { parseEther } from 'viem';
import { ABIS } from '../lib/abi';

interface DepositParams {
  vaultAddress: string;
  value?: string; // for ETH deposits
  tokenAddress?: string; // for ERC20 deposits
  amount?: string; // for ERC20 deposits
  tokenId?: bigint; // for NFT deposits
}

export function useAssetDeposits() {
  const { writeContract, data: hash, isPending, error } = useWriteContract();
  
  const { 
    data: receipt, 
    isLoading: isConfirming, 
    isSuccess: isConfirmed 
  } = useWaitForTransactionReceipt({ hash });

  const depositETH = ({ vaultAddress, value }: DepositParams) => {
    writeContract({
      address: vaultAddress as `0x${string}`,
      abi: ABIS.VAULT.abi,
      functionName: 'depositETH',
      value: parseEther(value || '0'),
    });
  };

  const depositERC20 = ({ vaultAddress, tokenAddress, amount }: DepositParams) => {
    writeContract({
      address: vaultAddress as `0x${string}`,
      abi: ABIS.VAULT.abi,
      functionName: 'depositERC20',
      args: [tokenAddress as `0x${string}`, parseEther(amount || '0')],
    });
  };

  const depositERC721 = ({ vaultAddress, tokenAddress, tokenId }: DepositParams) => {
    writeContract({
      address: vaultAddress as `0x${string}`,
      abi: ABIS.VAULT.abi,
      functionName: 'depositERC721',
      args: [tokenAddress as `0x${string}`, tokenId!],
    });
  };

  return {
    depositETH,
    depositERC20,
    depositERC721,
    isPending,
    isConfirming,
    isConfirmed,
    hash,
    receipt,
    error,
  };
}
```

### 4. useVaultRegistry Hook

```typescript
// hooks/useVaultRegistry.ts
import { useReadContract, useWriteContract } from 'wagmi';
import { ABIS, CONTRACTS } from '../lib/abi';

export function useVaultRegistry() {
  const { data: totalVaults, refetch: refetchTotal } = useReadContract({
    address: CONTRACTS.REGISTRY,
    abi: ABIS.REGISTRY.abi,
    functionName: 'getTotalVaults',
  });

  const { data: vaults, refetch: refetchVaults } = useReadContract({
    address: CONTRACTS.REGISTRY,
    abi: ABIS.REGISTRY.abi,
    functionName: 'getVaults',
    args: [0n, 100n], // offset, limit
  });

  const { data: distributableVaults, refetch: refetchDistributable } = useReadContract({
    address: CONTRACTS.REGISTRY,
    abi: ABIS.REGISTRY.abi,
    functionName: 'getDistributableVaults',
  });

  const { writeContract, data: hash, isPending, error } = useWriteContract();

  const executeBatch = (vaultList: string[], nextIndex: number) => {
    writeContract({
      address: CONTRACTS.REGISTRY,
      abi: ABIS.REGISTRY.abi,
      functionName: 'executeBatch',
      args: [vaultList as `0x${string}`[], BigInt(nextIndex)],
    });
  };

  return {
    totalVaults,
    vaults,
    distributableVaults,
    executeBatch,
    isPending,
    hash,
    error,
    refetch: {
      total: refetchTotal,
      vaults: refetchVaults,
      distributable: refetchDistributable,
    },
  };
}
```

---

## 🎨 React Components

### 1. Vault Creation Form

```typescript
// components/VaultCreationForm.tsx
import React, { useState } from 'react';
import { useAccount, useBalance } from 'wagmi';
import { useVaultCreation } from '../hooks/useVaultCreation';
import { formatEther, parseEther } from 'viem';

interface VaultCreationFormProps {
  onSuccess?: (vaultAddress: string) => void;
}

export function VaultCreationForm({ onSuccess }: VaultCreationFormProps) {
  const { address, isConnected } = useAccount();
  const { data: balance } = useBalance({ address });
  const { createVault, isPending, isConfirming, isConfirmed, error } = useVaultCreation();
  
  const [formData, setFormData] = useState({
    heir: '',
    inactivityPeriod: 2592000, // 30 days default
    depositAmount: '0.01',
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!isConnected || !address) return;
    
    try {
      await createVault({
        heir: formData.heir as `0x${string}`,
        inactivityPeriod: formData.inactivityPeriod,
        value: formData.depositAmount,
      });
      
      if (onSuccess && isConfirmed) {
        // Handle success - vault address will be in receipt logs
        onSuccess(''); // Get vault address from transaction receipt
      }
    } catch (err) {
      console.error('Vault creation failed:', err);
    }
  };

  if (!isConnected) {
    return <div>Connect wallet to create vault</div>;
  }

  return (
    <form onSubmit={handleSubmit} className="vault-creation-form">
      <div className="form-group">
        <label htmlFor="heir">Heir Address</label>
        <input
          id="heir"
          type="text"
          value={formData.heir}
          onChange={(e) => setFormData(prev => ({ ...prev, heir: e.target.value }))}
          placeholder="0x..."
          required
        />
      </div>

      <div className="form-group">
        <label htmlFor="inactivityPeriod">Inactivity Period (days)</label>
        <input
          id="inactivityPeriod"
          type="number"
          value={formData.inactivityPeriod / 86400}
          onChange={(e) => setFormData(prev => ({ 
            ...prev, 
            inactivityPeriod: parseInt(e.target.value) * 86400 
          }))}
          min="1"
          required
        />
      </div>

      <div className="form-group">
        <label htmlFor="depositAmount">Initial Deposit (ETH)</label>
        <input
          id="depositAmount"
          type="number"
          step="0.001"
          value={formData.depositAmount}
          onChange={(e) => setFormData(prev => ({ ...prev, depositAmount: e.target.value }))}
          min="0.001"
          required
        />
        <small>Balance: {balance?.formatted} ETH</small>
      </div>

      {error && <div className="error">{error.message}</div>}

      <button 
        type="submit" 
        disabled={isPending || isConfirming}
        className="submit-button"
      >
        {isPending ? 'Creating...' : isConfirming ? 'Confirming...' : 'Create Vault'}
      </button>
    </form>
  );
}
```

### 2. Vault Dashboard

```typescript
// components/VaultDashboard.tsx
import React, { useState, useEffect } from 'react';
import { useAccount } from 'wagmi';
import { useVaultSummary, useVaultOperations } from '../hooks';
import { formatEther } from 'viem';
import { formatDistanceToNow } from 'date-fns';

interface VaultDashboardProps {
  vaultAddress: string;
}

export function VaultDashboard({ vaultAddress }: VaultDashboardProps) {
  const { address } = useAccount();
  const { vaultSummary, isLoading, refetch } = useVaultSummary(vaultAddress);
  const { updateActivity, setHeir, isPending } = useVaultOperations(vaultAddress);
  
  const [timeRemaining, setTimeRemaining] = useState<string>('');
  const [newHeir, setNewHeir] = useState<string>('');

  useEffect(() => {
    if (!vaultSummary) return;
    
    const calculateTimeRemaining = () => {
      const now = Math.floor(Date.now() / 1000);
      const lastActivity = Number(vaultSummary.lastActivity);
      const inactivityPeriod = Number(vaultSummary.inactivityPeriod);
      const timeElapsed = now - lastActivity;
      const timeLeft = inactivityPeriod - timeElapsed;
      
      if (timeLeft <= 0) {
        setTimeRemaining('Ready for distribution');
      } else {
        setTimeRemaining(
          formatDistanceToNow(new Date((now + timeLeft) * 1000), { addSuffix: true })
        );
      }
    };

    calculateTimeRemaining();
    const interval = setInterval(calculateTimeRemaining, 1000);
    
    return () => clearInterval(interval);
  }, [vaultSummary]);

  if (isLoading) return <div>Loading vault details...</div>;
  if (!vaultSummary) return <div>Vault not found</div>;

  const handleUpdateActivity = async () => {
    try {
      await updateActivity();
      await refetch();
    } catch (error) {
      console.error('Failed to update activity:', error);
    }
  };

  const handleUpdateHeir = async () => {
    if (!newHeir) return;
    
    try {
      await setHeir(newHeir as `0x${string}`);
      await refetch();
      setNewHeir('');
    } catch (error) {
      console.error('Failed to update heir:', error);
    }
  };

  return (
    <div className="vault-dashboard">
      <div className="vault-header">
        <h2>Vault Dashboard</h2>
        <div className={`status-badge ${vaultSummary.isActive ? 'active' : 'inactive'}`}>
          {vaultSummary.isActive ? 'Active' : 'Inactive'}
        </div>
      </div>

      <div className="vault-details">
        <div className="detail-item">
          <label>Owner:</label>
          <span>{vaultSummary.owner}</span>
        </div>
        
        <div className="detail-item">
          <label>Heir:</label>
          <span>{vaultSummary.heir}</span>
        </div>
        
        <div className="detail-item">
          <label>Balance:</label>
          <span>{formatEther(vaultSummary.balance)} ETH</span>
        </div>
        
        <div className="detail-item">
          <label>Time Remaining:</label>
          <span>{timeRemaining}</span>
        </div>
        
        <div className="detail-item">
          <label>Status:</label>
          <span>{vaultSummary.canDistribute ? 'Ready for distribution' : 'Active'}</span>
        </div>
      </div>

      {address?.toLowerCase() === vaultSummary.owner.toLowerCase() && (
        <div className="vault-actions">
          <h3>Vault Actions</h3>
          
          <button 
            onClick={handleUpdateActivity}
            disabled={isPending}
            className="action-button update-activity"
          >
            Update Activity
          </button>

          <div className="heir-update">
            <input
              type="text"
              value={newHeir}
              onChange={(e) => setNewHeir(e.target.value)}
              placeholder="New heir address"
              className="heir-input"
            />
            <button 
              onClick={handleUpdateHeir}
              disabled={isPending || !newHeir}
              className="action-button update-heir"
            >
              Update Heir
            </button>
          </div>
        </div>
      )}
    </div>
  );
}
```

### 3. Asset Deposit Interface

```typescript
// components/AssetDeposit.tsx
import React, { useState } from 'react';
import { useAccount, useBalance, useReadContract } from 'wagmi';
import { useAssetDeposits } from '../hooks/useAssetDeposits';
import { ABIS } from '../lib/abi';
import { parseEther, formatUnits } from 'viem';

interface AssetDepositProps {
  vaultAddress: string;
}

export function AssetDeposit({ vaultAddress }: AssetDepositProps) {
  const { address, isConnected } = useAccount();
  const { depositETH, depositERC20, depositERC721, isPending, isConfirming } = useAssetDeposits();
  
  const [depositType, setDepositType] = useState<'ETH' | 'ERC20' | 'ERC721'>('ETH');
  const [formData, setFormData] = useState({
    amount: '0.1',
    tokenAddress: '',
    tokenId: '',
  });

  const handleDeposit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!isConnected) return;
    
    try {
      switch (depositType) {
        case 'ETH':
          await depositETH({
            vaultAddress,
            value: formData.amount,
          });
          break;
          
        case 'ERC20':
          await depositERC20({
            vaultAddress,
            tokenAddress: formData.tokenAddress,
            amount: formData.amount,
          });
          break;
          
        case 'ERC721':
          await depositERC721({
            vaultAddress,
            tokenAddress: formData.tokenAddress,
            tokenId: BigInt(formData.tokenId),
          });
          break;
      }
    } catch (error) {
      console.error('Deposit failed:', error);
    }
  };

  if (!isConnected) {
    return <div>Connect wallet to deposit assets</div>;
  }

  return (
    <div className="asset-deposit">
      <h3>Deposit Assets</h3>
      
      <div className="deposit-type-selector">
        {(['ETH', 'ERC20', 'ERC721'] as const).map((type) => (
          <button
            key={type}
            onClick={() => setDepositType(type)}
            className={`type-button ${depositType === type ? 'active' : ''}`}
          >
            {type}
          </button>
        ))}
      </div>

      <form onSubmit={handleDeposit} className="deposit-form">
        {depositType === 'ETH' && (
          <div className="form-group">
            <label htmlFor="amount">Amount (ETH)</label>
            <input
              id="amount"
              type="number"
              step="0.001"
              value={formData.amount}
              onChange={(e) => setFormData(prev => ({ ...prev, amount: e.target.value }))}
              min="0.001"
              required
            />
          </div>
        )}

        {depositType === 'ERC20' && (
          <>
            <div className="form-group">
              <label htmlFor="tokenAddress">Token Address</label>
              <input
                id="tokenAddress"
                type="text"
                value={formData.tokenAddress}
                onChange={(e) => setFormData(prev => ({ ...prev, tokenAddress: e.target.value }))}
                placeholder="0x..."
                required
              />
            </div>
            <div className="form-group">
              <label htmlFor="amount">Amount</label>
              <input
                id="amount"
                type="number"
                step="0.001"
                value={formData.amount}
                onChange={(e) => setFormData(prev => ({ ...prev, amount: e.target.value }))}
                min="0.001"
                required
              />
            </div>
          </>
        )}

        {depositType === 'ERC721' && (
          <>
            <div className="form-group">
              <label htmlFor="tokenAddress">NFT Contract Address</label>
              <input
                id="tokenAddress"
                type="text"
                value={formData.tokenAddress}
                onChange={(e) => setFormData(prev => ({ ...prev, tokenAddress: e.target.value }))}
                placeholder="0x..."
                required
              />
            </div>
            <div className="form-group">
              <label htmlFor="tokenId">Token ID</label>
              <input
                id="tokenId"
                type="number"
                value={formData.tokenId}
                onChange={(e) => setFormData(prev => ({ ...prev, tokenId: e.target.value }))}
                min="0"
                required
              />
            </div>
          </>
        )}

        <button 
          type="submit" 
          disabled={isPending || isConfirming}
          className="deposit-button"
        >
          {isPending ? 'Depositing...' : isConfirming ? 'Confirming...' : 'Deposit'}
        </button>
      </form>
    </div>
  );
}
```

---

## 🎯 Event Handling & Real-time Updates

### 1. Event Listeners

```typescript
// hooks/useVaultEvents.ts
import { useWatchContractEvent } from 'wagmi';
import { ABIS } from '../lib/abi';

export function useVaultEvents(vaultAddress: string, onEvent?: (event: any) => void) {
  // Listen to VaultCreated events
  useWatchContractEvent({
    address: undefined, // Factory address
    abi: ABIS.FACTORY.abi,
    eventName: 'VaultCreated',
    onLogs: (logs) => {
      logs.forEach(log => {
        if (onEvent) {
          onEvent({
            type: 'VaultCreated',
            data: log,
          });
        }
      });
    },
  });

  // Listen to vault-specific events
  useWatchContractEvent({
    address: vaultAddress as `0x${string}`,
    abi: ABIS.VAULT.abi,
    eventName: 'ActivityUpdated',
    onLogs: (logs) => {
      logs.forEach(log => {
        if (onEvent) {
          onEvent({
            type: 'ActivityUpdated',
            data: log,
          });
        }
      });
    },
  });

  useWatchContractEvent({
    address: vaultAddress as `0x${string}`,
    abi: ABIS.VAULT.abi,
    eventName: 'AssetDeposited',
    onLogs: (logs) => {
      logs.forEach(log => {
        if (onEvent) {
          onEvent({
            type: 'AssetDeposited',
            data: log,
          });
        }
      });
    },
  });

  useWatchContractEvent({
    address: vaultAddress as `0x${string}`,
    abi: ABIS.VAULT.abi,
    eventName: 'VaultDistributed',
    onLogs: (logs) => {
      logs.forEach(log => {
        if (onEvent) {
          onEvent({
            type: 'VaultDistributed',
            data: log,
          });
        }
      });
    },
  });
}
```

---

## 🛡️ Security & Error Handling

### 1. Transaction Error Handler

```typescript
// utils/transactionErrorHandler.ts
export function handleTransactionError(error: any): string {
  if (error.code === 4001) {
    return 'Transaction rejected by user';
  }
  
  if (error.code === -32603) {
    return 'Insufficient funds for gas';
  }
  
  if (error.message?.includes('reverted')) {
    if (error.message?.includes('Insufficient balance')) {
      return 'Insufficient balance for this operation';
    }
    if (error.message?.includes('Invalid heir address')) {
      return 'Invalid heir address provided';
    }
    if (error.message?.includes('Inactivity period too short')) {
      return 'Inactivity period must be at least 1 day';
    }
  }
  
  return error.message || 'Transaction failed';
}
```

### 2. Input Validation

```typescript
// utils/validation.ts
export const validateAddress = (address: string): boolean => {
  return /^0x[a-fA-F0-9]{40}$/.test(address);
};

export const validateInactivityPeriod = (days: number): boolean => {
  return days >= 1 && days <= 365 * 10; // Max 10 years
};

export const validateAmount = (amount: string): boolean => {
  const num = parseFloat(amount);
  return !isNaN(num) && num > 0 && num <= 1000000; // Reasonable limits
};
```

---

## 📊 Gas Estimation

### 1. Gas Estimation Hook

```typescript
// hooks/useGasEstimation.ts
import { useEstimateGas } from 'wagmi';
import { parseEther } from 'viem';
import { ABIS, CONTRACTS } from '../lib/abi';

export function useGasEstimation() {
  const estimateVaultCreation = (heir: string, inactivityPeriod: number, value: string) => {
    return useEstimateGas({
      address: CONTRACTS.FACTORY,
      abi: ABIS.FACTORY.abi,
      functionName: 'createVault',
      args: [heir as `0x${string}`, BigInt(inactivityPeriod)],
      value: parseEther(value),
    });
  };

  const estimateDepositETH = (vaultAddress: string, value: string) => {
    return useEstimateGas({
      address: vaultAddress as `0x${string}`,
      abi: ABIS.VAULT.abi,
      functionName: 'depositETH',
      value: parseEther(value),
    });
  };

  return {
    estimateVaultCreation,
    estimateDepositETH,
  };
}
```

---

## 🎨 UI Components Library

### 1. Base Components

```typescript
// components/ui/Button.tsx
import React from 'react';

interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
}

export function Button({ 
  children, 
  onClick, 
  disabled = false, 
  variant = 'primary',
  size = 'md',
  loading = false
}: ButtonProps) {
  const baseClasses = 'font-medium rounded-lg transition-all duration-200';
  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
    danger: 'bg-red-600 text-white hover:bg-red-700',
  };
  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
  };

  return (
    <button
      onClick={onClick}
      disabled={disabled || loading}
      className={`${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]} ${
        disabled ? 'opacity-50 cursor-not-allowed' : ''
      }`}
    >
      {loading ? (
        <span className="inline-flex items-center">
          <svg className="animate-spin -ml-1 mr-2 h-4 w-4" fill="none" viewBox="0 0 24 24">
            <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
            <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
          </svg>
          Processing...
        </span>
      ) : (
        children
      )}
    </button>
  );
}
```

---

## 🚀 Deployment & Performance

### 1. Performance Optimization

```typescript
// utils/performance.ts
export const debounce = <T extends (...args: any[]) => any>(
  func: T,
  wait: number
): ((...args: Parameters<T>) => void) => {
  let timeout: NodeJS.Timeout;
  return (...args: Parameters<T>) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => func(...args), wait);
  };
};

export const throttle = <T extends (...args: any[]) => any>(
  func: T,
  limit: number
): ((...args: Parameters<T>) => void) => {
  let inThrottle: boolean;
  return (...args: Parameters<T>) => {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
};
```

### 2. Bundle Optimization

```typescript
// next.config.js (if using Next.js)
/** @type {import('next').NextConfig} */
const nextConfig = {
  webpack: (config) => {
    config.resolve.fallback = {
      ...config.resolve.fallback,
      fs: false,
      net: false,
      tls: false,
    };
    return config;
  },
  experimental: {
    optimizePackageImports: ['wagmi', 'viem', '@tanstack/react-query'],
  },
};

module.exports = nextConfig;
```

---

## 📱 Responsive Design

### 1. Mobile-First CSS

```css
/* styles/globals.css */
.vault-dashboard {
  @apply w-full max-w-4xl mx-auto p-4 sm:p-6 lg:p-8;
}

.vault-creation-form {
  @apply space-y-4;
}

.form-group {
  @apply space-y-2;
}

.form-group label {
  @apply block text-sm font-medium text-gray-700;
}

.form-group input {
  @apply w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500;
}

.action-button {
  @apply px-4 py-2 text-sm font-medium text-white bg-blue-600 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500;
}

.status-badge {
  @apply px-3 py-1 rounded-full text-xs font-semibold;
}

.status-badge.active {
  @apply bg-green-100 text-green-800;
}

.status-badge.inactive {
  @apply bg-red-100 text-red-800;
}
```

---

## 🔧 Testing

### 1. Component Testing

```typescript
// __tests__/VaultCreationForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { WagmiProvider } from 'wagmi';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { VaultCreationForm } from '../components/VaultCreationForm';
import { config } from '../wagmi.config';

const queryClient = new QueryClient();

const renderWithProviders = (component: React.ReactElement) => {
  return render(
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        {component}
      </QueryClientProvider>
    </WagmiProvider>
  );
};

describe('VaultCreationForm', () => {
  it('renders form fields correctly', () => {
    renderWithProviders(<VaultCreationForm />);
    
    expect(screen.getByLabelText(/heir address/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/inactivity period/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/initial deposit/i)).toBeInTheDocument();
  });

  it('validates form inputs', async () => {
    renderWithProviders(<VaultCreationForm />);
    
    const submitButton = screen.getByRole('button', { name: /create vault/i });
    fireEvent.click(submitButton);
    
    await waitFor(() => {
      expect(screen.getByText(/heir address is required/i)).toBeInTheDocument();
    });
  });
});
```

---

## 📚 Complete Example Integration

```typescript
// pages/vaults/index.tsx
import React, { useState } from 'react';
import { useAccount } from 'wagmi';
import { VaultCreationForm } from '../../components/VaultCreationForm';
import { VaultDashboard } from '../../components/VaultDashboard';
import { AssetDeposit } from '../../components/AssetDeposit';

export default function VaultsPage() {
  const { address, isConnected } = useAccount();
  const [activeTab, setActiveTab] = useState<'create' | 'manage' | 'deposit'>('create');
  const [selectedVault, setSelectedVault] = useState<string>('');

  if (!isConnected) {
    return <div>Connect your wallet to manage vaults</div>;
  }

  return (
    <div className="vaults-page">
      <h1>Vault Management</h1>
      
      <div className="tab-navigation">
        <button 
          onClick={() => setActiveTab('create')}
          className={activeTab === 'create' ? 'active' : ''}
        >
          Create Vault
        </button>
        <button 
          onClick={() => setActiveTab('manage')}
          className={activeTab === 'manage' ? 'active' : ''}
        >
          Manage Vault
        </button>
        <button 
          onClick={() => setActiveTab('deposit')}
          className={activeTab === 'deposit' ? 'active' : ''}
        >
          Deposit Assets
        </button>
      </div>

      {activeTab === 'create' && (
        <VaultCreationForm 
          onSuccess={(vaultAddress) => {
            setSelectedVault(vaultAddress);
            setActiveTab('manage');
          }} 
        />
      )}

      {activeTab === 'manage' && selectedVault && (
        <VaultDashboard vaultAddress={selectedVault} />
      )}

      {activeTab === 'deposit' && selectedVault && (
        <AssetDeposit vaultAddress={selectedVault} />
      )}
    </div>
  );
}
```

---

## 🎯 Best Practices

1. **Always validate inputs before transactions**
2. **Use proper error handling and user feedback**
3. **Implement loading states for better UX**
4. **Cache data and use react-query for efficient data management**
5. **Use TypeScript for type safety**
6. **Test all user interactions and edge cases**
7. **Implement proper wallet connection handling**
8. **Use environment variables for sensitive configuration**
9. **Optimize bundle size with tree-shaking**
10. **Implement proper responsive design**

---

## 🔍 Troubleshooting

### Common Issues & Solutions

1. **"Contract not deployed"**: Verify contract addresses are correct for the target network
2. **"Insufficient gas"**: Increase gas limit or check wallet balance
3. **"User rejected transaction"**: Handle user rejection gracefully
4. **"Network mismatch"**: Ensure wallet is connected to Lisk Sepolia
5. **"ABI mismatch"**: Verify ABI files are up-to-date with deployed contracts

---

This comprehensive guide provides everything needed to integrate the NATECIN Vault System with modern web3 technologies using wagmi, viem, and ABI JSON imports. The modular architecture allows for easy customization and extension based on specific application requirements.
