=====
# CTTP - Cross-Chain Token Transfer Protocol
**By:** Jared Rice Sr.
**Published:** April 16th, 2020
**Last Updated:** April 16th, 2020

## Abstract
CTTP (Cross-Chain Token Transfer Protocol) is a protocol designed to allow for the autonomous transfer of cryptographic tokens between one blockchain and a wholly related blockchain. CTTP enables "cross-chain" token transfers by burning a specified amount of tokens on one blockchain, where the burnt amount, along with an identifier of the receiving blockchain, the account address of the receiving user and an identifier of the token burning event are stored in the sending blockchain's ledger, so that the receiving blockchain can verify:

- 1. It is actually intended to be the receiving blockchain
- 2. That the receiving account exists on its network; and
- 3. How many tokens to mint on the receiving blockchain

The protocol then sends the minted tokens to the verified account address and stores the minted amount, the transaction identifier (TXID) related to the minting event (POM - Proof Of Minting), along with the corresponding ID of the original burn event on the sending blockchain's ledger, in order to prevent attempts at "double minting" tokens. All of the protocol's operations are designed to take place within statically-typed smart contracts, where the blockchains involved in a two-way token transfer are able to utilize each other as oracles (trusted sources) in the transfer process, in order to eliminate the need for any centralized clearing or management systems. This also ensures that the CTTP protocol remains blockchain agnostic, as long as the blockchains involved utilize smart contracts and statically-typed smart contract programming languages.

## Background
There are many use-cases for a cryptocurrency to have functionality on more than just a single network. Many blockchain users prefer specific blockchain platforms over others and in-turn developers are building applications on these platforms in order to reach more users. Several new development frameworks now allow developers to easily develop applications that function alongside multiple blockchain platforms at once (ex. a decentralized social network known as dSocial, that functions on its own native blockchain called SocialX, yet is also integrated with Ethereum and EOS, so that Ethereum and EOS users can utilize the dSocial application without a SocialX blockchain account.). Although, if their application uses its own token (ex. dSocial uses the LIKE token), other blockchain platform users (i.e. users of Ethereum or EOS) would be unable to utilize the application if the token existed on only one of the blockchain platforms that the application was integrated with (i.e. if LIKE only existed on the SocialX blockchain and not on the Ethereum or EOS blockchains). This would require developers of such an application to issue the token or digital asset on all of the blockchain platforms it was built upon and as a result, centralizing the transfer of tokens or assets between blockchains. 

This is because there isn't a protocol for transferring tokens between multiple blockchains in a completely decentralized and autonomous way, where the total supply and inflation related features of a token are unaffected in the process. A protocol of this type would also have to prevent the "double minting" issue, where two or more minting events take place for a single burning event. Without a protocol like CTTP, clear functionality issues exist where UserA of BlockchainA would be unable to send an application's token (ex. LIKE), to UserB of BlockchainB, even if the token existed on both BlockchainA and BlockchainB, without centralizing the transfer between blockchains in order to avoid obvious exploits. Developers more than likely would also want to do so without affecting the token's overall total supply on both blockchain platforms, in order to legitimize the token's market value. CTTP fixes these issues by bringing forth a protocol for two different blockchain networks to safely communicate the burning (removal of a token from a blockchain's total supply) and the minting (issuing of a token into a blockchain's total supply) of a token, autonomously, without a single point of control or failure, ultimately allowing developers to completely integrate their application and application-specific token with any blockchain platform.

## 1. Burning (Via The Sending Blockchain)
The transferring of a cryptographic token from one blockchain platform to another blockchain platform; per the CTTP protocol design, begins with the "burning" process on the sending blockchain. The process is initiated with a transaction of a specified amount of tokens that the initiator intends to burn. On a blockchain platform like Ethereum , this initiating function would be known as a ```payable``` function, that in-turn would require the initiator to send a specified amount of tokens that they want to burn to the Ethereum-based smart contract itself. Being that CTTP is blockchain agnostic and different blockchains will certainly handle this initiating function a bit differently, CTTP requires that:

A. The initiator chooses how many tokens are being burnt through the "payment" or "self-destruction" (burning) of the tokens themselves.
B. This burn amount (```burnAmount```) must be determined through the payment of the initiator. In other words, it should not be determined through outside data sources or data that is submitted to the contract itself. 

As an example, the burning process can only start through a transaction where the initiator loses custody of a specified amount of tokens, whether those tokens are directly "burned" from the initiator's account by the initiator or the initiator awards custody of a specified amount of tokens to a smart contract following the CTTP protocol design.

By determining the burn amount through the initiator's transfer of a specified amount of tokens, we can ensure later in the transfer process that only this same specified amount can be minted by the minting() function on the receiving blockchain. Besides the burn amount, the CTTP protocol design depends on several key properties and identifiers so that the minting process on the receiving blockchain can take place. The following sub-sections cover those key properties. Cross-chain token transfers can happen on many different blockchain platforms, as long as the following properties and identifiers are retrieved, stored and queries properly via the sending blockchain:

### 1.1 Burn Event Identifier (```burnId```) [bId]
The cross-chain token transfer process is initiated through what is called a "burn event", where the initiator transfers custody of a specified amount of tokens to the custody or control of the sending blockchain, where the tokens are burnt by the initiator or by the smart contract itself, only after the smart contract receives custody of a specified amount of tokens. This burn event must have a randomly generated identifier that separates this burn event from all other burn events. The ```burnId``` should be generated by the contract itself as an unsigned 256-bit integer for purposes of uniformity between blockchain platforms.

### 1.2 Burn Amount (```burnAmount```) [bAm]
This property must be determined through the action of the initiator (ex. if the process is initiated through the custody transfer of 10 RIX tokens, then the burn amount must be 10 RIX). The ```burnAmount``` must derive from the amount of tokens used to initiate the process and cannot be written to the contract in any other way. **This is a crucial requirement of the CTTP protocol design**. 

### 1.3 Receiving Network Identifier (```toNet```) [tN]
```toNet``` is a numeric identifier used to identify the receiving blockchain that the ```burnAmount``` should be minted upon. For example, the following ```toNet``` identifiers are used in the reference implementation for CTTP, known as [aExchange](https://github.com/arisenio/aexchange):

```
IF Arisen - toNet=0
IF Ethereum - toNet=1
IF EOS - toNet=2
IF TRON - ToNet=3
IF BitShares - ToNet=4
```

This value is queried by the receiving blockchain's smart contract to verify it is in-fact the intended receiver (or in this case, minter).

### 1.4 Receiving Address (```toAddress```) [toAdd]
```toAddress``` is a ```string literal``` of either an account address or an account username on the receiving blockchain. This address should be submitted to the sending blockchain's smart contract by the implementation itself. aEx, the reference implementation of CTTP, gathers this particular property by asking the initiator which network they're transferring tokens to. If they choose EOS, as an example, Scatter prompts the initiator to login to Scatter and choose which EOS account they're transferring tokens to. This account information is sent by the sending blockchain's smart contract and stored in the "Burn" user-defined data container (explained in sub-section 1.8).

### 1.5 Proof Of Burn (```txIdOfBurn```) [txB]
Proof of the burn event on the sending blockchain, which is simply a ```string literal``` of the transaction ID related to the burning of a specified amount of tokens in [1.2](#1-2), must also be stored in the "Burn" user-defined data container. The sending blockchain's smart contract must determine the transaction ID for the burning event and this data **must not** derive from any outside data source.

### 1.6 Burning Timestamp (```timeBurned```) [tB]
The timestamp, intended to be stored in ```unixtime``` format, of the burn event must also be stored on the sending blockchain as another differentiator for the burn event itself. This should be stored as an ```integer```, should also be determined by the contract itself and **must not** derive from any outside data source.

### 1.7 Initiator (```burner```) [b]
The initiator of the burning process' account address or username should be determined through the initiating transaction when the ```burnAmount``` is determined and stored on the sending blockchain. This should be stored as a ```string```, should also be determined by the contract itself and **must not** derive from any outside data source.

### 1.8 Burn Data
All-in-all, it is crucial to the protocol design, that the burn data properties and identifiers are stored on the sending blockchain in on-chain data container or database called ```Burnt``` like so:

Burnt {
     string burner; 
     int burnId;
     double burnAmount;
     int toNet;
     string toAddress;
     string txIdForBurn;
     int timeBurned;
}

The sending blockchain acts as an oracle (trusted source) for the receiving blockchain's smart contract, so that the minting process can happen authentically and autonomously. 

## 2 Querying The Sending Blockchain
After the sending blockchain has stored the burning-related data (explained in [1.8](#1-8), the receiving blockchain is able to query the sending blockchain's burn-related data via the receiving blockchain's token contract, in order to initiate the minting of tokens. 

### 2.1 Sending Network Identifier (```fromNet```) [fN]
When the minting function within the receiving blockchain's token contract is initiated, it requires what is called a ```fromNet``` identifier, which like the ```toNet``` identifier is a numeric network identifier. The ```fromNet``` identifier must identify the sending blockchain, so that the receiving network's token contract knows the sending network to query with a specific ```burnId```.

### 2.2 Burn Identifier (```burnId```) [bId]
Besides the ```fromNet``` identifier, the minting function within the receiving blockchain's token contract requires a burnId so that the sending blockchain related to the ```fromNet``` identifier can be correctly queried. 

Querying the sending blockchain would follow this model:
```fromNet(burnId)```

## 3. Minting (Via The Receiving Blockchain)
The minting process on the receiving blockchain is initiated once the contract receives a ```fromNet``` identifier and the ```burnId``` that the minting event is based upon. Using this information, the sending blockchain, as identified by the ```fromNet``` identifier, is queried using the specified ```burnId``, where the receiving blockchain's token contract will pull the following attributes from the query's response:

```
{
burnAmount  //this will be stored as the ```mintAmount``` variable in the receiving blockchain's token contract.
toNet  // this will be stored as the ```mintingNetwork``` variable in the receiving blockchain's token contract.
toAddress  // this will be stored as the ```mintingAddress``` variable in the receiving blockchain's token contract.
}
```

### 3.1 Minting Amount (```mintingAmount```) [mA]
A variable known as ```mintAmount``` should be stored in the receiving blockchain's token contract and should be equivalent to the ```burnAmount``` key/value response received from the sending blockchain. The ```mintAmount``` should be used by the receiving blockchain's token contract as the amount of tokens to be minted. This ensures that the amount of tokens burnt on the sending blockchain, are the same amount of tokens minted on the receiving blockchain.

### 3.2 Minting Network (```mintingNetwork```) [mNet]
A variable known as ```mintingNetwork``` should be stored in the receiving blockchain's token contract and should be equivalent to the ```toNet``` key/value response received from the sending blockchain. The ```mintingNetwork``` should be used by the receiving network to ensure that it is in-fact intended to be the receiving network.

### 3.3 Minting Address (```mintingAddress```) [mAdd]
A variable known as ```mintingAddress``` should be stored in the receiving blockchain's token contract and should be the equivalent to the ```toAddress``` key/value response received from the sending blockchain. The receiving blockchain uses the ```mintingAddress``` to:

- A. Query its own network for the existence of the ```mintingAddress```; and 
- B. As the destination address for the newly minted tokens.

### 3.4 Account Existence [isRealAccount()]
isRealAccount() should be a Boolean style function designed to return a true or false if the ```mintingAddress``` exists on the receiving blockchain. Minting should not take place if the ```mintingAddress``` doesn't exist. That is why it's important in CTTP-based implementations that a receiving address is verified as being existent on the receiving network, prior to storing this data on the sending blockchain. It's also important to remember that this is not verified during the burning part of the protocol design and is only verified with the isRealAccount() smart contract function on the receiving blockchain.

### 3.5 Previous Mint Event Checking [beenMinted()]
To prevent double-minting, the sending blockchain must be queried to see if a particular ```burnId``` has been minted. As explained in [Section 3.7](#), after minting takes place on the receiving blockchain, the receiving blockchain's token contract should write to the "Minting" data container or database via the sending blockchain's token contract, to inform the sending blockchain that minting has taken place on a particular receiving blockchain, along with proof of the minting itself.

The ```beenMinted()``` function queries the sending blockchain (the ```fromNet``` identifier) to see if a particular ```burnId``` has been minted on any particular receiving network. The receiving network the minting took place on isn't important, as one burning event should only have one minting event, regardless of the network it took place on. The ```beenMinted()``` function should return a true or false on whether or not a particular ```burnId``` has been minted previously. 

### 3.6 Minting Of Tokens
Before minting tokens on the receiving blockchain:

1. The ```mintingNetwork``` must be equivalent to the receiving blockchain that is processing the minting request;
2. The ```beenMinted()``` function must return ```false```; and 
3. The ```isRealAccount()``` function must return ```true```.

If all 3 requirements pass, the receiving blockchain should mint the ```mintAmount``` and should send the newly minted tokens to the ```mintingAddress```.

This completes the transfer of a specified amount of tokens from one blockchain to a completely unrelated blockchain. Although, there is still a crucial step left in the protocol design, as explained in [Section 3.7](#).

### 3.7 Minted Data
The following data should be written to an on-chain data container or database  called ```Minted``` on the sending blockchain like so:

Minted {
     burnId // The burnId that is now minted 
     mintNet // The network the tokens were minted on
     amount // The amount that was minted
     mintTxId // Proof of Minting (POM)
}

By writing this data to the sending blockchain, we ensure that double-minting is unable to take place and tokens are autonomously transferred between blockchain platforms (cross-chain).

## 4. Oracles
Oracles are an important element of the protocol design, where the sending and receiving of blockchain platforms use each other as oracles (trusted sources), retrieving and storing information specific to the minting and burning of tokens. This way, outside applications can integrate with CTTP-based token contracts, knowing that the data between blockchain platforms that the token is being exchanged on, are truly "trustless", as long as both platforms follow standard decentralized principles.

## 5. Reference Implementations
There are several reference implementations covered in this section, built on several different blockchain platforms, including a dApp-based implementation where you can test out the CTTP protocol for yourself.

### 5.1 CTTP-Based Ethereum Implementation
An Ethereum-based smart contract implementation for the Rise (RIX) token, that follows the CTTP protocol design, can be found [here](https://github.com/arisenio/aexchange).

### 5.2 CTTP-Based  EOS  Implementation
An EOS-based smart contract implementation for the Rise (RIX) token, that follows the CTTP protocol design, can be found [here](https://github.com/arisenio/aexchange).

### 5.3 CTTP-Based TRON Implementation
A TRON-based smart contract implementation for the Rise (RIX) token, that follows the CTTP protocol design, can be found [here](https://github.com/arisenio/aexchange).

### 5.4 CTTP-Based Arisen Implementation
An Arisen-based smart contract implementation for the Rise (RIX) token, that follows the CTTP protocol design, can be found [here](https://github.com/arisenio/aexchange).

### 5.5 aExchange dApp
The aExchange dApp utilizes all of the above smart contract implementations allowing the RISE (RIX) token, to be sent between the EOS, Arisen, Ethereum, TRON and BitShares blockchains. It is also integrated with Scatter, aBank, MetaMask and TRONLink to make the signing of transactions much more simpler and efficient.

## Copyright
Copyright 2020 Peeps Labs. All rights reserved.