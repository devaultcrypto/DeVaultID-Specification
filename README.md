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

**Part** | **Example** | **Description**
--- | --- | ---
Alias | James | A human readable name, as an UTF-8 encoded string
Blockheight | 574998 | A numerical blockheight reference the block that stored the register transaction
Transaction ID | 0d86[...]bc18 | A hash of the transaction that registered the alias


## Alias Naming Restrictions
