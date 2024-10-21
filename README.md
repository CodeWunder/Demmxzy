# **ETH to ICP Bridge with ckETH**

This project is to show you how to deposit ETH from the Ethereum blockchain to the Internet Computer Protocol (ICP) using ckETH, and how to withdraw it back. It includes:

- A basic **frontend** built with React that allows users to deposit and withdraw ETH.
- **Backend canister** logic using Motoko to track ckETH deposits and withdrawals.

---

## **Table of Contents**

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Project Setup](#project-setup)
4. [Step-by-Step Guide](#step-by-step-guide)
   - Install DFINITY SDK
   - Set Up Canister
   - Frontend Development
5. [Running the Application](#running-the-application)
6. [Acknowledgements](#acknowledgements)

---

## **Introduction**

The aim of this project is to create a bridge that allows depositing ETH from the Ethereum network to ICP using ckETH (a wrapped version of ETH on ICP). It also enables users to view their ckETH balance and withdraw ckETH back to Ethereum.

---

## **Prerequisites**

- **Node.js** and **npm** installed.
- **DFINITY SDK** installed.
- **Metamask** browser extension.
- Basic knowledge of **React**, **Ethereum**, and **ICP**.

---

## **Project Setup**

### Step 1: Install and Set Up the DFINITY SDK

If you haven’t installed the DFINITY SDK, do so using the following command:

```bash
sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"
```

### Step 2: Install Metamask

- Download and install the [Metamask extension](https://metamask.io/download.html) for your browser.
- Set up a wallet and ensure it contains ETH for testing.

---

## **Step-by-Step Guide**

### **Backend: Create and Deploy Canister on ICP**

#### Step 1: Create a New DFINITY Project

Create a new DFINITY project that will contain the canister code:

```bash
dfx new eth2icp
cd eth2icp
```

#### Step 2: Modify Canister Code to Support ckETH Deposits

Open `src/eth2icp/main.mo` and implement basic functions to deposit and withdraw ckETH:

```motoko
actor {

  var ckETH_balance : Nat = 0;

  public query func get_balance() : async Nat {
    return ckETH_balance;
  };

  public func deposit_ckETH(amount : Nat) : async Text {
    ckETH_balance += amount;
    return "Deposit Successful!";
  };

  public func withdraw_ckETH(amount : Nat) : async Text {
    if (ckETH_balance >= amount) {
      ckETH_balance -= amount;
      return "Withdrawal Successful!";
    } else {
      return "Insufficient balance!";
    }
  };
}
```

#### Step 3: Deploy the Canister

Start the Internet Computer and deploy the canister:

```bash
dfx start
dfx deploy
```

Copy the **canister ID** after deployment. You’ll need this to interact with the canister from the frontend.

---

### **Frontend: Create React App for Depositing/Withdrawing ETH**

#### Step 1: Set up React Project

Create a React project to serve as the frontend interface:

```bash
npx create-react-app eth2icp-frontend
cd eth2icp-frontend
```

#### Step 2: Install Dependencies

Install the necessary packages to interact with Ethereum and ICP:

```bash
npm install ethers @dfinity/agent
```

#### Step 3: Implement Deposit/Withdrawal Logic

Edit `src/App.js` to include functionality for interacting with Ethereum and the deployed ICP canister:

```javascript
import React, { useState } from "react";
import { ethers } from "ethers";
import { Actor, HttpAgent } from "@dfinity/agent";
import { idlFactory, canisterId } from "../../src/declarations/eth2icp";

function App() {
  const [ethAmount, setEthAmount] = useState("");
  const [ckEthBalance, setCkEthBalance] = useState(0);

  // Connect to Ethereum via Metamask
  const connectMetamask = async () => {
    if (window.ethereum) {
      await window.ethereum.request({ method: "eth_requestAccounts" });
      const provider = new ethers.providers.Web3Provider(window.ethereum);
      const signer = provider.getSigner();
      return signer;
    }
  };

  // Deposit ETH into ICP via ckETH
  const depositEth = async () => {
    const signer = await connectMetamask();
    const amount = ethers.utils.parseEther(ethAmount);
    console.log("ETH deposit to ICP initiated: ", amount);

    // Interact with ICP canister to register ckETH deposit
    const agent = new HttpAgent();
    const ethCanister = Actor.createActor(idlFactory, {
      agent,
      canisterId,
    });

    await ethCanister.deposit_ckETH(amount.toString());
    fetchCkEthBalance();
  };

  // Fetch ckETH balance from ICP
  const fetchCkEthBalance = async () => {
    const agent = new HttpAgent();
    const ethCanister = Actor.createActor(idlFactory, {
      agent,
      canisterId,
    });

    const balance = await ethCanister.get_balance();
    setCkEthBalance(balance);
  };

  return (
    <div className="App">
      <h1>Deposit ETH to ICP using ckETH</h1>
      <input
        type="text"
        value={ethAmount}
        onChange={(e) => setEthAmount(e.target.value)}
        placeholder="Enter ETH amount"
      />
      <button onClick={depositEth}>Deposit ETH</button>

      <h2>ckETH Balance: {ckEthBalance}</h2>
      <button onClick={fetchCkEthBalance}>Refresh Balance</button>
    </div>
  );
}

export default App;
```

---

## **Running the Application**

### Backend (ICP)

1. Start the Internet Computer:

```bash
dfx start
```

2. Deploy the canister:

```bash
dfx deploy
```

### Frontend

1. Navigate to the frontend directory and start the development server:

```bash
npm start
```

2. Open your browser and interact with the frontend at `http://localhost:3000`. Use Metamask to deposit ETH and check your ckETH balance.

---

## **Acknowledgements**

This project is inspired by the need to bridge assets between Ethereum and ICP, with a focus on decentralized applications and token interoperability using ckETH.

