# Abstract

Cash Address is a naming system that can be used alongside regular bitcoin addresses and payment codes to simplify the process of sharing payment information.


# Motivation

The Bitcoin address system based on hashing public keys creates complex and difficult to share identifiers. While these identifiers have proper checksums and misspellings are rare, they are still very cumbersome to transfer over the telephone, in a regular chat or similar. Many attempts have been made to obfuscate the addresses by transferring them as QR codes or NFC tags, but the need for a human accessible format remains.


# Specification

## Introduction

When a user wants to, they can create a **Cash Address** by selecting a suitable name and publish a transaction on-chain to aquire an identifier. Once the transaction is included in a block the user is informed of their name and identifier and can now share this information in a convenient manner with others.

When a user receives a **Cash Address** name (with or without identifier), their wallet looks up the **Payment Code** that corresponds to the alias.

## Cash Address Names

A full **Cash Address Name** consists of an **Alias**, a **Blockheight** and a **Transaction ID**.

```
James#574998:0d8648cbb1725cc5bbe59c47fa4f6268fe8879ad6fe2b094a3e934e80f3abc18;
```

**Part** | **Example** | **Description**
--- | --- | ---
Alias | James | A human readable name, as an UTF-8 encoded string
Blockheight | 574998 | A numerical blockheight reference the block that stored the register transaction
Transaction ID | 0d86[...]bc18 | A hash of the transaction that registered the alias

**Note:** *Unless the same name is registered twice in the same block, the transaction identifier is redundant and can be dropped. If a wallet detects that a transaction ID will be necessary, it may opt to re-create the alias in a later block to get a simpler identifier.*


## Alias Naming Restrictions

While the protocol does not enforce any naming restrictions other than length, there are some considerations for implementors. For example, non-spoken bytes such as tabs, spaces, carriage return, null for string termination or characters that can be used in database injection attacks might be best to disallow.

It is ultimately up to the application to determine what rules to apply given the langage and user base.


## Protocol 

A **Cash Address Transaction** consists of a protocol **Identifier**, **Version**, **Action** and **Payload**.

**Part** | **Length** | **Description**
--- | --- | ---
Identifier | 4 bytes | A static identifier for the Cash Address protocol: 0x01010101.
Version | 1 byte | A version byte determines how to parse actions and payloads.
Action | 1 byte | An action byte determines how to parse the payload.
Payload | dynamic | Payload based on the version and action.

## Actions

### Version 1

**Name** | **Action Identifier** | **Structure** | **Notes**
--- | --- | --- | ---
Register | 0x01 | Payment Code (80), Alias Name (1~99) | Creates a new Cash Address using the Alias Name, blockheight and TXID.


# References

**Name** | **BIP** | **Link**
--- | --- | ---
Reusable Payment Codes | BIP-47 | https://github.com/OpenBitcoinPrivacyProject/bips/blob/master/bip-0047.mediawiki
OP_RETURN | N/A | https://en.bitcoin.it/wiki/OP_RETURN
