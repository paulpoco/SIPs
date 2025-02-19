---
sip:  31
title: New fee framwork
description: No need to duplicate contract codes in the chain
author:   jjos, frank_the_tank
status: Final
type: Standard
category: Core
requiers : SIP-3, SIP-20, SIP-23, SIP-29
created: 2021-09-18
---
## Introduction
Signum introduced the so-called “slot-fee” mechanism on [SIP-3](sip-3.md) and its protocol enforcement with [SIP-23](sip-23.md).

The experience has shown that these constraints lead to unnecessary delays in transaction confirmation. The motivation for these past SIPs was mainly spam concerns, which we will cover with a different approach in the present proposal.

## Motivation
Currently with the “slot-fee” mechanism we end up with unconfirmed transactions just waiting for blocks without much advantage. Users have a bad experience waiting even though blocks have plenty of space to be filled. All this is done to prevent spam, but keeping transactions in the memory of all nodes can be even more damaging than simply confirming them all and letting them be stored in the database. The idea was also to create more rewards for miners, but that is already handled by the minimum rewards that are now part of the protocol [SIP-29](sip-29.md).

## Spam concerns
We had some spam tests on testnet around block height 4k and 10k. They consisted in filling all blocks with multi-outs using all the space available. The effect we see is that those blocks are harder to process. This makes the sync from empty slower during those blocks. These blocks slow down the process because they all send to new accounts, so the accounts table needs to be extended and accounts created. This is not an issue for the operation of the chain or pools, as a fraction of a second per block would not be an issue. But it can make the sync from empty slower as the time goes.

Another problem is the total blockchain size, transactions can use up to 1k bytes as attachment, so you can greatly reduce the maximum number of transactions by sending a lot of transactions with attachments. Plus the transactions are stored forever and that makes the blockchain size to increase.

## Proposal

The present proposal consists basically in:
 - Remove the fee enforcement from the protocol, leaving only the FEE_QUANT = 0.00735 Signa limit per transaction in the protocol level
 - Introduce a new minimum fee as a function of transaction bytes:
    `minFee = FEE_QUANT * floor (transactionBytes / 176)`
 - Adjust the fee for alias assignment to 20 * FEE_QUANT
 - Adjust the fee for asset (token) issuance to 15000 * FEE_QUANT

Note that the above computation uses the `floor` function and so any transaction with less bytes than 176*2=352 would pay the FEE_QUANT. Larger transactions, with a lot of bytes attached, multi-outs with many recipients, etc. would pay a bit more fee. With this we avoid bloating the database at a very low cost. 

Examples of resulting fees:
 - Ordinary TX send: 176 bytes, pays FEE_QUANT
 - Add commitment: 184 bytes, pays FEE_QUANT
 - Deposit to an exchange (with the memo): 245 bytes, pays FEE_QUANT
 - Place token order: 201 bytes, pays FEE_QUANT
 - TX send to extended address (with public key attached): 209 bytes, pays FEE_QUANT
 - Multi-out with the max receivers: over 1200 bytes, pays 6*FEE_QUANT 
 - Message with max attachment: over 1200 bytes, pays 6*FEE_QUANT 
 - Create subscription: 181 bytes, pays FEE_QUANT

These changes would also solve the *faucet draining* issue.
If this is implemented, faucets must always send only FEE_QUANT so any attempt to dry out faucets will be pointless, since you end up with zero coins.


## Specification

In this CIP the following changes are suggested:
The fee enforcement will be removed from the protocol
A fee per 176 bytes will be introduced with: 
minFee = FEE_QUANT * (transactionBytes / 176)
A minFeeFactor will be introduced for each transaction type

Transactions will be multiples of the FEE_QUANT, depending on how many bytes they take, as follows:
![image](./assets/sip-31/New_Fee_Framwork.png)

As can be observed, most of the transactions will stay with the current FEE_QUANT, but longer messages need to pay a higher fee.  As can be seen in the table above, Alias and Token creation also are being adjusted.
## Backwards Compatibility
This is a hard forking change, thus breaks compatibility with old fully-validating node. It should not be deployed without widespread consensus.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
