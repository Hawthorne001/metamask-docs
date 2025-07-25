---
description: Get started quickly with the MetaMask Delegation Toolkit.
sidebar_position: 3
sidebar_label: Delegation quickstart
---

# Delegation Toolkit quickstart

This page demonstrates how to get started quickly with the MetaMask Delegation Toolkit, by creating a delegator account and completing the delegation lifecycle (creating, signing, and redeeming a delegation).

## Prerequisites

[Install and set up the Delegation Toolkit.](install.md)

## Steps

### 1. Set up a Public Client

Set up a [Viem Public Client](https://viem.sh/docs/clients/public) using Viem's `createPublicClient` function. This client will let the delegator account query the signer's account state and interact with smart contracts.

```typescript
import { createPublicClient, http } from 'viem'
import { sepolia as chain } from 'viem/chains'

const publicClient = createPublicClient({
  chain,
  transport: http(),
})
```

### 2. Set up a Bundler Client

Set up a [Viem Bundler Client](https://viem.sh/account-abstraction/clients/bundler) using Viem's `createBundlerClient` function. This lets you use the bundler service to estimate gas for user operations and submit transactions to the network.

```typescript
import { createBundlerClient } from 'viem/account-abstraction'

const bundlerClient = createBundlerClient({
  client: publicClient,
  transport: http('https://your-bundler-rpc.com'),
})
```

### 3. Create a delegator account

[Create a delegator smart account](../how-to/create-smart-account/index.md) to set up a delegation.

This example configures a [Hybrid](../how-to/create-smart-account/configure-accounts-signers.md#configure-a-hybrid-smart-account) delegator account:

```typescript
import { Implementation, toMetaMaskSmartAccount } from '@metamask/delegation-toolkit'
import { privateKeyToAccount } from 'viem/accounts'

const delegatorAccount = privateKeyToAccount('0x...')

const delegatorSmartAccount = await toMetaMaskSmartAccount({
  client: publicClient,
  implementation: Implementation.Hybrid,
  deployParams: [delegatorAccount.address, [], [], []],
  deploySalt: '0x',
  signatory: { account: delegatorAccount },
})
```

### 4. Create a delegate account

Create a delegate account to receive the delegation. The delegate can be either a smart account or an externally owned account (EOA).

This example uses a smart account:

```typescript
import { Implementation, toMetaMaskSmartAccount } from '@metamask/delegation-toolkit'
import { privateKeyToAccount } from 'viem/accounts'

const delegateAccount = privateKeyToAccount('0x...')

const delegateSmartAccount = await toMetaMaskSmartAccount({
  client: publicClient,
  implementation: Implementation.Hybrid,
  deployParams: [delegateAccount.address, [], [], []],
  deploySalt: '0x',
  signatory: { account: delegateAccount },
})
```

### 5. Create a delegation

[Create a root delegation](../how-to/create-delegation/index.md#create-a-root-delegation) from the delegator account to the delegate account.

This example passes an empty `caveats` array, which means the delegate can perform any action on the delegator's behalf. We recommend [restricting the delegation](../how-to/create-delegation/restrict-delegation.md) by adding caveat enforcers.

:::warning Important

Before creating a delegation, ensure that the delegator account has been deployed. If the account is not deployed, redeeming the delegation will fail.

:::

```typescript
import { createDelegation } from '@metamask/delegation-toolkit'

const delegation = createDelegation({
  to: delegateSmartAccount.address,
  from: delegatorSmartAccount.address,
  caveats: [], // Empty caveats array - we recommend adding appropriate restrictions.
})
```

### 6. Sign the delegation

[Sign the delegation](../how-to/create-delegation/index.md#sign-a-delegation) using the [`signDelegation`](../reference/api/smart-account.md#signdelegation) method from `MetaMaskSmartAccount`. Alternatively, you can use the Delegation Toolkit's [`signDelegation`](../reference/api/delegation.md#signdelegation) utility. The signed delegation will be used later to perform actions on behalf of the delegator.

```typescript
const signature = await delegatorSmartAccount.signDelegation({
  delegation,
})

const signedDelegation = {
  ...delegation,
  signature,
}
```

### 7. Redeem the delegation

The delegate account can now [redeem the delegation](../how-to/redeem-delegation.md). The redeem transaction is sent to the `DelegationManager` contract, which validates the delegation and executes actions on the delegator's behalf.

To prepare the calldata for the redeem transaction, use the [`redeemDelegation`](../reference/api/delegation.md#redeemdelegation) utility function from the Delegation Toolkit.

```typescript
import { createExecution } from '@metamask/delegation-toolkit'
import { DelegationManager } from '@metamask/delegation-toolkit/contracts'
import { SINGLE_DEFAULT_MODE } from '@metamask/delegation-toolkit/utils'
import { zeroAddress } from 'viem'

const delegations = [signedDelegation]

const executions = createExecution({ target: zeroAddress })

const redeemDelegationCalldata = DelegationManager.encode.redeemDelegations({
  delegations: [delegations],
  modes: [SINGLE_DEFAULT_MODE],
  executions: [executions],
})

const userOperationHash = await bundlerClient.sendUserOperation({
  account: delegateSmartAccount,
  calls: [
    {
      to: delegateSmartAccount.address,
      data: redeemDelegationCalldata,
    },
  ],
  maxFeePerGas: 1n,
  maxPriorityFeePerGas: 1n,
})
```
