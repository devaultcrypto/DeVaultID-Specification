# Abstract

Cash Accounts is a naming system that can be used alongside regular bitcoin addresses and payment codes to simplify the process of sharing payment information.


# Motivation

The Bitcoins address system based on hashing data creates complex and difficult to share identifiers. While these identifiers have proper checksums and misspellings are rare, they are still very cumbersome to transfer over the telephone, in a regular chat or similar. Many attempts have been made to obfuscate the addresses by transferring them as QR codes or NFC tags, but the need for a human accessible format remains.


# Specification

## Introduction

When a user wants to, they can create a **Cash Account** by selecting a suitable name and publish a transaction on-chain to aquire an identifier. Once the transaction is included in a block the user is informed of their name and identifier and can now share this information in a convenient manner with others.

When a user receives a **Cash Account** name (with or without identifier), their wallet looks up the **Payment Information** that corresponds to the alias.


## Cash Address Names

A full **Cash Account** consists of an **Name**, a **Blockheight** and a **Transaction ID**.

```
James#574998:0d8648cbb1725cc5bbe59c47fa4f6268fe8879ad6fe2b094a3e934e80f3abc18;
```

**Part** | **Example** | **Description**
--- | --- | ---
Name | James | A human readable name, as an UTF-8 encoded string
Blockheight | #574998 | A numerical blockheight reference the block that stored the register transaction
Transaction ID | :0d86[...]bc18 | A hash of the transaction that registered the alias

* *When a name registration is unique in a given block, the TXID is redundant and not used in the account identifier.*
* *When the same name is registered by multiple parties, only as much as is needed from the TXID is used in the account identifier.*
* *When a wallet detects that a naming collision, it may opt to re-create the alias in a later block to get a simpler identifier.*
* *The **:** separator between the blockheight and the TXID is optional.*


## Protocol 

To register a **Cash Account** you broadcast a **Bitcoin Cash** transaction with an OP_RETURN output cointaining a protocol **Identifier**, one or more **Payment Data** and an **Account Name**.

```
OP_RETURN
    OP_PUSH 01010101
    OP_PUSH <PAYMENT_DATA> [...OP_PUSH <PAYMENT_DATA>]
    OP_PUSH <ACCOUNT_NAME>
```

### Protocol Prefix

This protocol adheres to the [OP_RETURN Prefix Guidelines](https://github.com/Lokad/Terab/blob/master/spec/opreturn-prefix-guideline.md) and uses the 0x01010101 protocol identifier.

### Payment Data Types

The initial version of this protocol supports three types of payment data: **Key Hashes**, **Script Hashes** and **Payment Codes**

**Name** | **Identifier** | **Structure**
--- | --- | ---
Key Hash | 0x01 | Key Hash (20)
Script Hash | 0x02 | Script Hash (20)
Payment Code | 0x03 | Payment Code (80)


## Account Naming Restrictions

While the protocol does not enforce any naming restrictions other than length, there are some considerations for implementors. For example, non-spoken bytes such as tabs, spaces, carriage return, null for string termination or characters that can be used in database injection attacks might be best to disallow.

It is ultimately up to the application to determine what rules to apply given the langage and user base.


# References

**Name** | **BIP** | **Link**
--- | --- | ---
Reusable Payment Codes | BIP-47 | https://github.com/OpenBitcoinPrivacyProject/bips/blob/master/bip-0047.mediawiki
OP_RETURN | N/A | https://en.bitcoin.it/wiki/OP_RETURN
OP_RETURN Guidelines | N/A | https://github.com/Lokad/Terab/blob/master/spec/opreturn-prefix-guideline.md
