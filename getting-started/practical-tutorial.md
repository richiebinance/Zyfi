# Practical Tutorial

## ERC20 Paymaster

This guide will help you get started with the Zyfi API to transform a normal transaction in a paymaster one.

For advanced usage and detailed explanation of each parameter and feature, please explore our detailed [API documentation](https://api.zyfi.org/api-doc).

The basic flow consists of the following steps:

1. Collect the desired transaction payload and the ERC20 the user desires to pay with
2. Send an API request
3. Receive back the quote and transaction payload for the user to sign

### Step 1: Send the API request <a href="#step-1-send-the-api-request" id="step-1-send-the-api-request"></a>

Send the required data to the `erc20_paymaster` endpoint. Below an example implementation using Javascript [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch\_API/Using\_Fetch) (natively supported in most environments)

Copy

```
// Define the payload
const payload = {
  chainId: 324, // Optional, defaults to zkSync Era Mainnet; valid options are 324, 300 (ZkSync-Sepolia), 388 (Cronos zkEVM mainnet), 282 (Cronos zkEVM Tesnet) and 11124 (Abstract Testnet).
  feeTokenAddress: // ERC20 the user desires to use as gas token
  txData: {
    from: "0x...",
    to: "0x...",
    data: "0x.."
  }
};

// Define the function to perform the POST request
async function postTransactionData() {
  try {
    const response = await fetch('https://api.zyfi.org/api/erc20_paymaster/v1', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(payload),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    console.log(data); // Process the response data
  } catch (error) {
    console.error('Error during the API call:', error);
  }
}
```

The returned object does not change the original transaction calldata, but add the paymaster fields necessary to process the transaction.

### Step 2: Show the quote to the user <a href="#step-2-show-the-quote-to-the-user" id="step-2-show-the-quote-to-the-user"></a>

The response, on top of the transaction payload, returns a series of helper values that can be used by the UI to better inform the user. In particular:

* `tokenAddress` : ERC20 token address used to pay for the gas fee (feeToken)
* `tokenPrice` : the estimated cost of the feeToken
* `feeTokenAmount` : Max amount of the ERC20 token that the user will pay as fee (before refunds). The user need to have this balance for the transaction to not fail
* `feeUSD` : Equivalent value of feeTokenAmount in USD
* `markup` : Markup or discount applied on the gas fee
* `expirationTime` : `block.timestamp` of the quote expiration. Currently it's one hour

Transactions on zkSync overestimate the gas needed and receive a refund at the end, which is difficult to know in advance.

The unspent gas is refunded to the user in the feeToken.

We suggest to each protocol to implement their own estimation for the final cost by observing the actual gas being used on their transaction. We also provide a very rough estimation with `estimatedFinalFeeTokenAmount` and `estimatedFinalFeeUSD`

### Step 3: Execute the transaction <a href="#step-3-execute-the-transaction" id="step-3-execute-the-transaction"></a>

EthersViem

Ensure you are using [zksync-ethers](https://docs.zksync.io/build/sdks/js/getting-started.html#getting-zksync-ethers) V5 or V6.

Below an example implementation with

Copy

```
import { Signer, Web3Provider } from "zksync-ethers";
import * as ethers from "ethers";

const signer = Web3Provider.getSigner

rawTx = Apiresponse.txData

//since the API returns the transaction payload in the ethers format, we can use it as is

txHash = await signer.sendTransaction(rawTx);
```

## Sponsored Paymaster

This guide will help you get started with Zyfi API sponsored endpoint to allow custom business logic.

In addition to the standard ERC20 Paymaster flow, it allows a protocol to decide how much of the transaction it wishes to sponsor for an user with their off-chain business logic by setting the sponsorshipRatio. What is not sponsored is paid by the user with the selected feeToken

For advanced usage and detailed explanation of each parameter and feature, please explore our detailed [API documentation](https://api.zyfi.org/api-doc).

The basic flow consists of the following steps:

1.  Deposit the ETH that will be used to sponsor in the `vault` contract: on ZkSync, the vault's address is

    Copy

    ```
    0x32faBA244AB815A5cb3E09D55c941464DBe31496
    ```
2. Get an `API-Key` connected to the vault balance (Ask to the [Zyfi team](https://t.me/Paul\_Web3))
3. Collect the transaction payload (similar to the ERC20 Paymaster)
4. Send an API request with the sponsorship information
5. Receive back the quote and transaction payload for the user to sign

[Here's a quick demo video](https://youtu.be/AukF\_sqCMSc?feature=shared) on how to integrate the Zyfi API on the Abstract Chain.

### Step 1: Send the API request <a href="#step-1-send-the-api-request" id="step-1-send-the-api-request"></a>

Send the required data to the `erc20_sponsored_paymaster` endpoint. Below an example implementation using Javascript [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch\_API/Using\_Fetch) (natively supported in most environments)

Copy

```
// Define the payload
const payload = {
  chainId: 324, // Optional, defaults to zkSync Era Mainnet; valid options are 324, 300 (ZkSync-Sepolia), 388 (Cronos zkEVM Mainnet), 282 (Cronos zkEVM Testnet) and 11124 (Abstract Testnet).
  feeTokenAddress: // ERC20 the user desires to use as gas token
  sponsorshipRatio: // [0-100] which % of the transaction is sponsored by the protocol 
  replayLimit: // Optional (default of 5), how many times the user can execute the transaction
  txData: {
    from: "0x...",
    to: "0x...",
    data: "0x.."
  }
};

// Define the function to perform the POST request
async function postTransactionData() {
  try {
    const response = await fetch('https://api.zyfi.org/api/erc20_sponsored_paymaster/v1', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': '<API Key>'
      },
      body: JSON.stringify(payload),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    console.log(data); // Process the response data
  } catch (error) {
    console.error('Error during the API call:', error);
  }
}
```

The returned object does not change the original transaction calldata, but add the paymaster fields necessary to process the transaction.

### Step 2: Show the quote to the user <a href="#step-2-show-the-quote-to-the-user" id="step-2-show-the-quote-to-the-user"></a>

The response, on top of the transaction payload, returns a series of helper values that can be used by the UI to better inform the user. In particular:

* `tokenAddress` : ERC20 token address used to pay for the gas fee (feeToken)
* `tokenPrice` : the estimated cost of the feeToken
* `feeTokenAmount` : Max amount of the ERC20 token that the user will pay as fee (before refunds). The user need to have this balance for the transaction to not fail
* `feeUSD` : Equivalent value of feeTokenAmount in USD
* `markup` : Markup or discount applied on the gas fee
* `protocolAddress` : The account in the vault being used to pay for the sponsored part
* `sponsorshipRatio` : which % of the transaction is sponsored by the protocol
* `expirationTime` : `block.timestamp` of the quote expiration. Currently it's one hour

Transactions on zkSync overestimate the gas needed and receive a refund at the end, which is difficult to know in advance.

The unspent gas is refunded to the user in the feeToken.

We suggest to to each protocol to implement their own estimation for the final cost by observing the actual gas being used on their transaction. We also provide a very rough estimation with `estimatedFinalFeeTokenAmount` and `estimatedFinalFeeUSD`

### Step 3: Execute the transaction <a href="#step-3-execute-the-transaction" id="step-3-execute-the-transaction"></a>

EthersViem

Ensure you are using [zksync-ethers](https://docs.zksync.io/build/sdks/js/getting-started.html#getting-zksync-ethers) V5 or V6.

Below an example implementation with

Copy

```
import { Signer, Web3Provider } from "zksync-ethers";
import * as ethers from "ethers";

const signer = Web3Provider.getSigner

rawTx = Apiresponse.txData

//since the API returns the transaction payload in the ethers format, we can use it as is

txHash = await signer.sendTransaction(rawTx);
```
