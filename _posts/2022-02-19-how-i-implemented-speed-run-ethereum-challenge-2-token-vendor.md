---
title: "DRAFT - How I Implemented Speed Run Ethereum Challenge 2: Token Vendor"
description: "Explaining my accepted solution to Speed Run Ethereum Challenge 2"
toc: true
comments: true
layout: post
categories: [solidity, ethereum, web3, cryptocurrency]
image: images/tokenvendor.png
author: Leo Gau
---

# DRAFT

# What I'm Building

In this post, I'll show you how I implemented Challenge 2 of Speed Run Ethereum

## 🚩 Challenge 2: 🏵 Token Vendor 🤖

> 🏵 Create `YourToken.sol` smart contract that inherits the **ERC20** token standard from OpenZeppelin. Set your token to `_mint()` **1000** (* 10 ** 18) tokens to the `msg.sender`. Then create a `Vendor.sol` contract that sells your token using a payable `buyTokens()` function.

[Speed Run Ethereum](https://speedrunethereum.com/challenge/token-vendor)

# The Code

Deployed token contract: https://rinkeby.etherscan.io/address/0x2Bfd3d12bABf24711960afFA2518AfAbE401A440#code

```solidity
contract YourToken is ERC20 {
    constructor() ERC20("Gold", "GLD") {
        _mint(msg.sender, 1000 * 10 ** 18);
    }
}
```

Deployed vendor contract:

https://rinkeby.etherscan.io/address/0x759c77f4f268eAFf274C33b5047731CCA553cC94

```solidity
contract Vendor is Ownable {
  /// Reference to our ERC20 token contract
  YourToken public yourToken;

  /// Our token price
  uint256 public constant tokensPerEth = 100;

  // Events
  event BuyTokens(address buyer, uint256 amountOfETH, uint256 amountOfTokens);
  event SellTokens(address buyer, uint256 amountOfETH, uint256 amountOfTokens);

  constructor(address tokenAddress) {
    yourToken = YourToken(tokenAddress);
  }

  /// Allow users to buy tokens
  function buyTokens() public payable {
    // Validate the user sent eth
    uint256 amountOfEth = msg.value;
    require(amountOfEth > 0, "Send some ETH to buy tokens");

    // Validate the vendor has enough tokens
    uint256 amountOfTokens = amountOfEth * tokensPerEth;
    uint256 vendorBalance = yourToken.balanceOf(address(this));
    require(vendorBalance >= amountOfTokens, "Vendor does not have enough tokens");

    // Send the tokens
    address buyer = msg.sender;
    (bool sent) = yourToken.transfer(buyer, amountOfTokens);
    require(sent, "Failed to transfer token");

    // Emit buy event
    emit BuyTokens(buyer, amountOfEth, amountOfTokens);
  }

  /// Allow the owner to withdraw ETH
  function withdraw() public onlyOwner {
    // Validate the vendor has ETH to withdraw
    uint256 vendorBalance = address(this).balance;
    require(vendorBalance > 0, "Vendor does not have any ETH to withdraw");

    // Send ETH
    address owner = msg.sender;
    (bool sent, ) = owner.call{value: vendorBalance}("");
    require(sent, "Failed to withdraw");
  }

  /// Allow users to sell tokens back to the vendor
  function sellTokens(uint256 amount) public {
    // Validate token amount
    require(amount > 0, "Must sell a token amount greater than 0");

    // Validate the user has the tokens to sell
    address user = msg.sender;
    uint256 userBalance = yourToken.balanceOf(user);
    require(userBalance >= amount, "User does not have enough tokens");

    // Validate the vendor has enough ETH
    uint256 amountOfEth = amount / tokensPerEth;
    uint256 vendorEthBalance = address(this).balance;
    require(vendorEthBalance > amountOfEth, "Vendor does not have enough ETH");

    // Transfer tokens
    (bool sent) = yourToken.transferFrom(user, address(this), amount);
    require(sent, "Failed to transfer tokens");

    // Transfer ETH
    (bool ethSent, ) = user.call{value: amountOfEth }("");
    require(ethSent, "Failed to send back eth");

    // Emit sell event
    emit SellTokens(user, amountOfEth, amount);
  }
}
```

Deployed UI: [https://thundering-process.surge.sh/](https://thundering-process.surge.sh/)

# Checkpoint 2: 🏵Your Token 💵

The first thing we need to do is create our ERC20 token. We're using the popular OpenZeppelin library to create the token. 

```solidity
contract YourToken is ERC20 {
    constructor() ERC20("Gold", "GLD") {
        _mint(msg.sender, 1000 * 10 ** 18);
    }
}
```

What we need to do is update the address we're minting too.  Otherwise this is boilerplate for minting ERC20 tokens.



# Checkpoint 3: ⚖️ Vendor 🤖

For this checkpoint, I need to implement the ability to buy these new tokens

```solidity
  // Events
  event BuyTokens(address buyer, uint256 amountOfETH, uint256 amountOfTokens);

  /// Allow users to buy tokens
  function buyTokens() public payable {
    // Validate the user sent eth
    uint256 amountOfEth = msg.value;
    require(amountOfEth > 0, "Send some ETH to buy tokens");

    // Validate the vendor has enough tokens
    uint256 amountOfTokens = amountOfEth * tokensPerEth;
    uint256 vendorBalance = yourToken.balanceOf(address(this));
    require(vendorBalance >= amountOfTokens, "Vendor does not have enough tokens");

    // Send the tokens
    address buyer = msg.sender;
    (bool sent) = yourToken.transfer(buyer, amountOfTokens);
    require(sent, "Failed to transfer token");

    // Emit buy event
    emit BuyTokens(buyer, amountOfEth, amountOfTokens);
  }
```

After all the validation, we transfer tokens with

```solidity
address buyer = msg.sender;
(bool sent) = yourToken.transfer(buyer, amountOfTokens);
```

and then emit the buy event so the UI can update

```
emit BuyTokens(buyer, amountOfEth, amountOfTokens);
```

Then we add a withdraw function so the owner can withdraw ETH from the contract

# Checkpoint 4: 🤔 Vendor Buyback 🤯

In order to buy tokens back, we need to implement the "Approve" pattern for ERC20s



ERC20 tokens will come with an Approve function. Update the UI to make the call. After the user approves, we can move the tokens out of their wallet back to the vendor. Then the vendor sends ETH to the caller.



```solidity
  /// Allow users to sell tokens back to the vendor
  function sellTokens(uint256 amount) public {
    // Validate token amount
    require(amount > 0, "Must sell a token amount greater than 0");

    // Validate the user has the tokens to sell
    address user = msg.sender;
    uint256 userBalance = yourToken.balanceOf(user);
    require(userBalance >= amount, "User does not have enough tokens");

    // Validate the vendor has enough ETH
    uint256 amountOfEth = amount / tokensPerEth;
    uint256 vendorEthBalance = address(this).balance;
    require(vendorEthBalance > amountOfEth, "Vendor does not have enough ETH");

    // Transfer tokens
    (bool sent) = yourToken.transferFrom(user, address(this), amount);
    require(sent, "Failed to transfer tokens");

    // Transfer ETH
    (bool ethSent, ) = user.call{value: amountOfEth }("");
    require(ethSent, "Failed to send back eth");

    // Emit sell event
    emit SellTokens(user, amountOfEth, amount);
  }
```



# So What Did I Do?

I've now implemented a token vendor which can buy and sell the token that I created.



At this point, I updated the code to deploy to Rinkeby and deployed the UI to surge.



On to the next challenge.



See you then!


