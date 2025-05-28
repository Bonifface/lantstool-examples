---
id: clear-state
title: "Contract State Cleaner"
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import {Github} from "@site/src/components/codetabs";


In NEAR blockchain, the contract state is persisted across redeployments.
If the structure of the contract has changed — for example, if a variable type is modified or new fields are added — the old version of the data may become incompatible.
This can lead to deserialization errors, such as:

```bash
Smart contract panicked: Cannot deserialize the contract state.
````

Additionally, if the contract uses an initialization function (#[init]), it cannot be called again unless the state is cleared.
This makes it impossible to reinitialize a contract without removing its existing state.

### Why Manual Clearing Often Doesn’t Work
Another challenge arises with large contracts that store a significant number of keys or data entries.
Every operation that deletes a storage key consumes gas, and there is a strict limit on the maximum gas that can be used in a single transaction.

As a result, fully clearing the state manually is often not feasible, especially for contracts with complex or large data sets.
This limitation makes it impractical to rely on internal cleanup logic or multi-call scripts.

That’s why specialized tools such as Lantstool's Account Cleaner or the CLI-based method are recommended for clearing contract state — especially when dealing with contracts that need to be reinitialized, redeployed cleanly, or reset to a clean state.

---

## Methods for cleaning contract state
There are two available methods for clearing the state of a smart contract in NEAR using:
- Account Cleaner utility in the [Lantstool](https://app.lantstool.dev/) application, which is available only for mainnet
- NEAR CLI, which can be used across all networks, including mainnet, testnet, and local environments

## Account Cleaner

To simplify the process of removing large contract state from a NEAR smart contract, the Lantstool application offers a convenient tool called Account Cleaner:

1. Open the [Lantstool](https://app.lantstool.dev/) app
2. Navigate to the "Utils" section in the sidebar
3. Select "Account Cleaner"
4. Choose "Clear Contract State"
5. Click the "Clear Contract State" button

![lantstool](/docs/assets/lantstool/lantstool-near_protocol-utils-clear_contract_state.png)

After the contract state is successfully cleared, you will see a confirmation message in the logs:
<p>"Operation completed successfully"</p>

<details>
<summary> Example logs </summary>

![lantstool](/docs/assets/lantstool/lantstool-near_protocol-utils-clear_contract_state-logs.png)

</details>

:::tip Want to see it in action?

Watch a short demo video here: [Account Cleaner](https://www.youtube.com/watch?v=84OlJric7lk&t=9s)

:::

:::note
The Clear State feature in [Lantstool](https://app.lantstool.dev/) is currently available only on NEAR Mainnet.
:::

---

## Using CLI to clean contract state

This JavaScript CLI tool deploys a [`state-cleanup.wasm`](https://github.com/near-examples/near-clear-state/blob/main/contractWasm/state_cleanup.wasm) contract replacing the current one, and then uses the new contract to clean up the account's state, so you can easily redeploy a new contract or use the account in any other way.

Here's a quick snippet of the contract's main code:

<Github language="rust" url="https://github.com/near-examples/near-clear-state/blob/main/state-cleanup/src/lib.rs" start="21" end="24" />

:::tip Want to check the smart contract?

Check the GitHub repository and learn more about the [State Cleanup tool](https://github.com/near-examples/near-clear-state).

<!-- https://github.com/nameskyteam/state-cleanup -->

:::


---
### How to use

### Requirements

You'll need [NEAR CLI](cli.md). You can install it by running:

```bash
npm install -g near-cli-rs@latest
```

### Clear your Account State

To clear your Account state, follow these steps.

#### 1. Login with NEAR CLI

This will store a full access key locally on your machine.
Select the account you wish to clear the state.

```bash
near login
```

:::warning Legacy keychain
Be sure to select `Store the access key in my legacy keychain (compatible with the old near CLI)` to store the access key on the legacy keychain.
:::

#### 2. Clone the `near-clear-state` Repository

```sh
git clone https://github.com/near-examples/near-clear-state.git
```

#### 3. Install dependencies

```bash
cd near-clear-state && npm i
```

#### 4. Clear your State

```bash
npx near-clear-state clear-state --account <account-name.testnet>
```

:::tip mainnet

If you want to clean the state of a `mainnet` account, use the `--network` option:

```sh
npx near-clear-state clear-state --account <account-name.near> --network mainnet
```

:::

#### (Optional) Check your results

You can view the all the state keys have been erased in your account with:

```bash
near view-state <account-name.testnet>
```

---

### Troubleshooting

If your contract state is large, depending on the RPC node, you may get the error:

```
State of contract example.near is too large to be viewed.
```

This is an RPC issue, as the RPC node has a limited contract state view.

You can prevent the error `State of contract example.near is too large to be viewed` when calling view-state via the JSON RPC API if you select an alternative RPC provider. You can find different providers in [this RPC list](../api/rpc/providers.md).

<Github language="javascript" url="https://github.com/near-examples/near-clear-state/blob/main/commands/clearState.js" start="22" end="30" />

For example, you could replace the default RPC node in [`commands/clearState.js`](https://github.com/near-examples/near-clear-state/blob/main/commands/clearState.js) with another RPC server:

```js
config = {
  networkId: netId,
  keyStore,
  nodeUrl: "https://endpoints.omniatech.io/v1/near/"+ netId +"/public",
  ...
```
