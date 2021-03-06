---
id: Deposit
title: 3. Deposit
sidebar_label: Deposit
---

If you successfully deposited some tokens, they will be locked into DepositContract, and the contract will mint the corresponding, equivalent amount of tokens that will be used in Plasma.

## 3-1. Implement deposit

You can call the `deposit` function from the plasma light client.

The users can simply deposit tokens into Plasma just by invoking this function.

[Plasma Light Client API reference | deposit](/docs/api/Plasma_Light_Client#deposit)

```javascript
const DEPOSIT_CONTRACT_ADDRESS = config.payoutContracts.DepositContract;

async function deposit(client, amount) {
  console.log("deposit:", amount);
  await client.deposit(amount, DEPOSIT_CONTRACT_ADDRESS);
}
```

## 3-2. Add deposit function to the CUI

In order to enable invoking `deposit` function from the CUI Wallet, write a process to execute the `deposit` function when the deposit is entered in ReadLine.

```javascript
function cuiWalletReadLine(client) {
  rl.question(">> ", async (input) => {
    const args = input.split(/\s+/);
    const command = args.shift();
    switch (command) {
      case "deposit":
        await deposit(client, args[0]);
        cuiWalletReadLine(client);
        break;
      default:
        console.log(`${command} is not found`);
        cuiWalletReadLine(client);
    }
  });
}

async function main() {
  const client = await startLightClient();
  cuiWalletReadLine(client);
}

main();
```

## 3-3. Deposit your ether

Launch the CUI Wallet and deposit your Ether into Plasma!

Start the app with the node command and try `deposit <amount>`.

```
$ node app.js
>> deposit 100
```

\* At this time, the unit is wei.

## Current source code

<details>
<summary>Click here</summary>

```javascript
const readline = require("readline");
const ethers = require("ethers");
const { Bytes } = require("@cryptoeconomicslab/primitives");
const { LevelKeyValueStore } = require("@cryptoeconomicslab/level-kvs");
const initializeLightClient = require("@cryptoeconomicslab/eth-plasma-light-client")
  .default;

// TODO: enter your private key
const PRIVATE_KEY = "ENTER YOUR PRIVATE KEY";
const config = require("./config.local.json");
const DEPOSIT_CONTRACT_ADDRESS = config.payoutContracts.DepositContract;

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

async function deposit(client, amount) {
  console.log("deposit:", amount);
  await client.deposit(amount, DEPOSIT_CONTRACT_ADDRESS);
}

async function startLightClient() {
  const kvs = new LevelKeyValueStore(Bytes.fromString("plasma_light_client"));
  const wallet = new ethers.Wallet(
    PRIVATE_KEY,
    new ethers.providers.JsonRpcProvider("http://127.0.0.1:8545")
  );
  const lightClient = await initializeLightClient({
    wallet,
    kvs,
    config,
    aggregatorEndpoint: "http://127.0.0.1:3000",
  });
  await lightClient.start();
  return lightClient;
}

function cuiWalletReadLine(client) {
  rl.question(">> ", async (input) => {
    const args = input.split(/\s+/);
    const command = args.shift();
    switch (command) {
      case "deposit":
        await deposit(client, args[0]);
        cuiWalletReadLine(client);
        break;
      case "quit":
        console.log("Bye.");
        rl.close();
        process.exit();
      default:
        console.log(`${command} is not found`);
        cuiWalletReadLine(client);
    }
  });
}

async function main() {
  const client = await startLightClient();
  cuiWalletReadLine(client);
}

main();
```

</details>

## Go to the next step!

You have now deposited your Ether to Plasma.

Let's see if your deposit was successfully completed, by checking your balance.

Move on to the [4. Show balance](tutorial/cui-wallet/show-balance.md) step.
