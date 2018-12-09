# Abstract

Cash Accounts is a naming system that can be used alongside regular bitcoin addresses and payment codes to simplify the process of sharing payment information.


# Motivation

The Bitcoin address system based on hashing data creates complex and difficult to share identifiers. While these identifiers have proper checksums and misspellings are rare, they are still very cumbersome to transfer over the telephone, in a regular chat or similar. Many attempts have been made to obfuscate the addresses by transferring them as QR codes or NFC tags, but the need for a human accessible format remains.


# Specification

## Introduction

When a user wants to, they can create a **Cash Account** by selecting a suitable name and publish a transaction on-chain to aquire an identifier. Once the transaction is included in a block the user is informed of their name and identifier and can now share this information in a convenient manner with others.

When a user receives a **Cash Account Identifier** their wallet looks up the **Payment Information** that corresponds to the alias.


### Cash Account Identifiers

### Complete Idenfitiers

A **Complete Identifier** consists of an **Name**, a **Blockheight** and a **Transaction ID**.

```
James#574998:0d8648cbb1725cc5bbe59c47fa4f6268fe8879ad6fe2b094a3e934e80f3abc18;
```

**Part** | **Example** | **Description**
--- | --- | ---
Name | James | A human readable name, as an UTF-8 encoded string
Blockheight | #574998 | A numerical blockheight reference the block that stored the register transaction
Transaction ID | :0d86[...]bc18 | A Hex-encoded hash of the transaction that registered the alias

#### Minimal Identifiers

Most of the time it is expected that **Account Names** are uniquely registered in their block heights. This allows the shortest identifier to consist of only the **Name** and **Blockheight** to form a simple human-accessible **Minimal Identifier**.

```
James#574998;
```

#### Short Identifiers

From time to time though, the same name will be registered more than once at the same blockheight, and a part of the **Transaction ID** will be needed. This additional part is only as long as required to resolve the **Name Collision** and is expected to stay within a few characters, creating a **Short Identifier**.

```
James#574998:A;
James#574998:5;
```

* *When a wallet detects a naming collisions during registration, it may opt to re-create the account in a later block to get a simpler identifier.*




## Protocol 

To register a **Cash Account** you broadcast a **Bitcoin Cash** transaction with an OP_RETURN output cointaining a **Protocol Identifier**, an **Account Name** and one or more **Payment Data**.

```
OP_RETURN (0x6a)
    OP_PUSH (0x04) <PROTOCOL> (0x01010101)
    OP_PUSH (0x4c) <LENGTH> <ACCOUNT_NAME>
    OP_PUSH (0x4c) <LENGTH> <PAYMENT_DATA> [...OP_PUSH (0x4c) <LENGTH> <PAYMENT_DATA>]
```

### Protocol Identifier

This protocol adheres to the [OP_RETURN Prefix Guidelines](https://github.com/Lokad/Terab/blob/master/spec/opreturn-prefix-guideline.md) and uses the 0x01010101 protocol identifier.

### Account Name

The **Account Name** is an UTF-8 encoded string with a character length between 1 and 99, and a byte length small enough to allow for the desired **Payment Data**. Furthermore it also need to match a strict **Regular Expression** of ```/\w+/``` (equivalent to ```/[a-zA-Z0-9_]+/```) to be valid.

Presentation of **Account Names** should always be in the case that they are stored in while collision checks must always be done in lower case.


### Payment Data Types

**Payment Data** consists of a **Payment Identifier** and **Payment Data**. The initial version of this protocol supports three types of payment data: **Key Hashes**, **Script Hashes** and **Payment Codes**.

**Name** | **Payment Identifier** | **Payment Data** | **Notes**
--- | --- | --- | ---
Key Hash | 0x01 | Key Hash (20) | [Address reuse](https://en.bitcoin.it/wiki/Address_reuse) undermines the security and privacy of the users.
Script Hash | 0x02 | Script Hash (20) | [Address reuse](https://en.bitcoin.it/wiki/Address_reuse) undermines the security and privacy of the users.
Payment Code | 0x03 | Payment Code (80) | Published payment codes [might undermine the privacy](https://github.com/bitcoin/bips/wiki/Comments:BIP-0047) of the users.


## Security and Scalability

While it is technically possible for a client to download the referenced block, scan all transactions and look for matching transactions to parse the payment data, doing so at scale might be too costly for light clients. To optimize this process it is expected that 3rd parties provide indexing services.


### Indexing Services

A service can be made that continously scans the blockchain to create a database of valid **Cash Accounts**, indexed by their shortest **Account Idenfitiers**.

Such a service should allow for convenient lookup of **Cash Account** information, but must return all data needed to independently verify the information, such as the **Full Transaction**, **SPV-proof** and **Block Information**.

An indexing service should validate the **Cash Account Identifier** according to ```/\w+#\d+(:[0-9a-fA-F]+)?;/``` to ensure that the lookup was not sent prematurely.

If a user requests lookup for a **Minimal Identifier** that cannot be uniquely idenfitied, an indexing service must return all matching transactions and should clearly indicate that the lookup was unsuccessful.


### Known Attacks

#### Index Collusion

When **Alice** and **Bob** uses the same **Indexing Service** there is a risk that the **Indexing Service** could trick **Bob** into accepting an incorrect lookup. The attack starts when **Alice** tries to register an account name. The **Indexing Service** could detect the attempt in the mempool and create an intentional collision, but report to **Alice** that her name was created without any collisions. When **Bob** requests a lookup for **Alices** account, the **Indexing Service** could then provide its own account with a valid **SPV-proof** and transaction reference.

To mitigate this attack **Alice** should poll random indexing services after creation, or download the full block to locally detect any collisions. If any honest service report a collision or there is a locally detect a collision, **Alice** now knows that her chosen indexing service cannot be trusted.


# References

**Name** | **BIP** | **Link**
--- | --- | ---
Bitcoin Script | N/A | https://en.bitcoin.it/wiki/Script
Reusable Payment Codes | BIP-47 | https://github.com/OpenBitcoinPrivacyProject/bips/blob/master/bip-0047.mediawiki
OP_RETURN | N/A | https://en.bitcoin.it/wiki/OP_RETURN
OP_RETURN Guidelines | N/A | https://github.com/Lokad/Terab/blob/master/spec/opreturn-prefix-guideline.md
