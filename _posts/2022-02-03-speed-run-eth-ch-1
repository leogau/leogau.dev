---
title: "How I Implemented Speed Run ETH Challenge 1"
description: How I Implemented Speed Run ETH Challenge 1
toc: true
comments: true
layout: post
categories: [solidity, ethereum]
image: images/fastpages_posts/actions/actions_logo.png
author: Leo Gau
---

# DRAFT

# How I Implemented Speed Run ETH Challenge 1

# What I'm Building

In this post, I'll show you how I implemented Challenge 1 of Speed Run ETH.

Here's the description from the challenge page: https://speedrunethereum.com/challenge/decentralized-staking

> 🏦 Build a `Staker.sol` contract that collects **ETH** from numerous addresses using a payable `stake()` function and keeps track of `balances`. After some `deadline` if it has at least some `threshold` of ETH, it sends it to an `ExampleExternalContract` and triggers the `complete()` action sending the full balance. If not enough **ETH** is collected, allow users to `withdraw()`.

# The Code

Deployed contract address: 

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.4;

import "hardhat/console.sol";
import "./ExampleExternalContract.sol";

contract Staker {
  // VARIABLES
  /// External contract that will hold staked funds if threshold reached
  ExampleExternalContract public exampleExternalContract;

  /// Balances of user's staked funds
  mapping (address => uint256) public balances;

  /// Staking threshold
  uint256 public constant threshold = 1 ether;

  /// Staking deadline
  uint256 public deadline = block.timestamp + 72 hours;

  /// Boolean set if threshold is not reached by the deadline
  bool public openForWithdraw;

  // EVENTS
  /// Emit when stake() is called
  event Stake(address sender, uint256 value);

  // MODIFIERS
  /// Modifier that checks whether the required deadline has passed
  modifier deadlinePassed(bool requireDeadlinePassed) {
    uint256 timeRemaining = timeLeft();
    if (requireDeadlinePassed) {
      require(timeRemaining <= 0, "Deadline has not been passed yet");
    } else {
      require(timeRemaining > 0, "Deadline is already passed");
    }
    _;
  }

  /// Modifier that checks whether the external contract is completed
  modifier stakingNotCompleted() {
    bool completed = exampleExternalContract.completed();
    require(!completed, "Staking period has completed");
    _;
  }

  constructor(address exampleExternalContractAddress) public {
      exampleExternalContract = ExampleExternalContract(exampleExternalContractAddress);
  }

  // Collect funds in a payable `stake()` function and track individual `balances` with a mapping:
  //  ( make sure to add a `Stake(address,uint256)` event and emit it for the frontend <List/> display )
  function stake() public payable deadlinePassed(false) stakingNotCompleted {
    // update the sender's balance
    balances[msg.sender] += msg.value;

    // emit Stake event to notify the UI
    emit Stake(msg.sender, msg.value);
  }


  // After some `deadline` allow anyone to call an `execute()` function
  //  It should either call `exampleExternalContract.complete{value: address(this).balance}()` to send all the value
  function execute() public stakingNotCompleted {
    uint256 contractBalance = address(this).balance;
    if (contractBalance >= threshold) {
      // if the `threshold` is met, send the balance to the externalContract
      exampleExternalContract.complete{value: contractBalance}();
    } else {
      // if the `threshold` was not met, allow everyone to call a `withdraw()` function
      openForWithdraw = true;
    }
  }



  // Add a `withdraw(address payable)` function lets users withdraw their balance
  function withdraw(address payable _to) public deadlinePassed(true) stakingNotCompleted {
      // check the amount staked did not reach the threshold by the deadline
      require(openForWithdraw, "Not open for withdraw");

      // get the sender balance
      uint256 userBalance = balances[msg.sender];

      // check if the sender has a balance to withdraw
      require(userBalance > 0, "userBalance is 0");

      // reset the sender's balance
      balances[msg.sender] = 0;

      // transfer sender's balance to the `_to` address
      (bool sent, ) = _to.call{value: userBalance}("");

      // check transfer was successful
      require(sent, "Failed to send to address");
  }

  /// Add a `timeLeft()` view function that returns the time left before the deadline for the frontend
  function timeLeft() public view returns (uint256) {
      if (block.timestamp >= deadline) {
          return 0;
      } else {
          return deadline - block.timestamp;
      }
  }


  // Add the `receive()` special function that receives eth and calls stake()
  receive() external payable {
      stake();
  }
}
```

# Checkpoint 2: Staking

The first challenge after getting my environment set up is to implement staking.

I add the 2 variables needed and implement the stake function below

```
mapping ( address => uint256 ) public balances;
uint256 public constant threshold = 1 ether;
```

Then i implement the staking function

```
/// Emit when stake() is called
event Stake(address sender, uint256 value);

// Collect funds in a payable `stake()` function and track individual `balances` with a mapping:
//  ( make sure to add a `Stake(address,uint256)` event and emit it for the frontend <List/> display )
function stake() public payable {
   balances[msg.sender] += msg.value;

   emit Stake(msg.sender, msg.value);
}
```

Since he stake function is marked as `payable`, the function is able to recieve ether and has access to the sender's address -`msg.sender`-  and the amount of ether sent - `msg.value`.

Using these variables, we can update the contract's `balances` mapping with the total amount of ether that address has sent.

You'll notice I didn't have to initialize `balances[msg.sender]` to 0. Solidity will automatically initialize variables to 0 values.

After updating, the function emit's a Stake event. This is how our UI will know something has happened on the blockchain.

# Checkpoint 3

The next checkout is the meat of the contract. I need to keeps track of what state the contract is in. Is it open for staking? Does the contract balance exceed the threshold? Have we passed the deadline?



Let's talk about each function separately.



First we have execute

```
// Staking deadline
uint256 public deadline = block.timestamp + 30 seconds;

/// Boolean set if threshold is not reached by the deadline
bool public openForWithdraw;

// After some `deadline` allow anyone to call an `execute()` function
// It should either call `exampleExternalContract.complete{value: address(this).balance}()` to send all the value
function execute() public stakingNotCompleted {
  uint256 contractBalance = address(this).balance;
  if (contractBalance >= threshold) {
    // if the `threshold` is met, send the balance to the externalContract
    exampleExternalContract.complete{value: contractBalance}();
  } else {
    // if the `threshold` was not met, allow everyone to call a `withdraw()` function
    openForWithdraw = true;
  }
}

```



Next we have withdraw

```
// Add a `withdraw(address payable)` function lets users withdraw their balance
function withdraw(address payable _to) public deadlinePassed(true) stakingNotCompleted {
    // check the amount staked did not reach the threshold by the deadline
    require(openForWithdraw, "Not open for withdraw");

    // get the sender balance
    uint256 userBalance = balances[msg.sender];

    // check if the sender has a balance to withdraw
    require(userBalance > 0, "userBalance is 0");

    // reset the sender's balance
    balances[msg.sender] = 0;

    // transfer sender's balance to the `_to` address
    (bool sent, ) = _to.call{value: userBalance}("");

    // check transfer was successful
    require(sent, "Failed to send to address");
}
```



Finally the helper function timeLeft

```
/// Add a `timeLeft()` view function that returns the time left before the deadline for the frontend
function timeLeft() public view returns (uint256) {
    if (block.timestamp >= deadline) {
        return 0;
    } else {
        return deadline - block.timestamp;
    }
}
```





* requires

* timestamp

* address(this).balance

* calling other contracts

* sending ether to an address

* view

* returns

# Checkpoint 4

For this checkpoint, we implement a `receive` function to automatically stake eth which has been sent to the contract address.

```
// Add the `receive()` special function that receives eth and calls stake()
receive() external payable {
    stake();
}
```



We also create some modifiers to share logic across functions

```
  /// Modifier that checks whether the required deadline has passed
  modifier deadlinePassed(bool requireDeadlinePassed) {
    uint256 timeRemaining = timeLeft();
    if (requireDeadlinePassed) {
      require(timeRemaining <= 0, "Deadline has not been passed yet");
    } else {
      require(timeRemaining > 0, "Deadline is already passed");
    }
    _;
  }

  /// Modifier that checks whether the external contract is completed
  modifier stakingNotCompleted() {
    bool completed = exampleExternalContract.completed();
    require(!completed, "Staking period has completed");
    _;
  }
```



# So What Did We Do?

I updated the code to point at the `Rinkeby` ethereum test net and deployed the frontend to surge. 

You can see the deployed UI here althrough the staking period is over by now.

Now on to Challenge 2. 

See you then!
