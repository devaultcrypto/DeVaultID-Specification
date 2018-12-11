**This is a draft specification - please do not use or share until finalized.**


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

From time to time though, the same name will be registered more than once at the same blockheight, and a part of the **Transaction ID** will be needed. This **Collision Avoidance Part** is only as long as required to resolve the **Name Collision** and is expected to stay within a few characters, creating a **Short Identifier**.

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

The **Account Name** is an UTF-8 encoded string with a character length between 1 and 99, and a byte length small enough to allow for the desired **Payment Data**. Furthermore it is recommended that the name matches a strict **Regular Expression** of ```/\w{1,99}/``` to retain the human accessibility trait.

Presentation of **Account Names** should always be in the case that they are stored in while collision checks must always be done in lower case.


### Payment Data Types

**Payment Data** consists of a **Payment Identifier** and **Payment Information**.

**Name** | **Payment Identifier** | **Payment Information** | **Notes**
--- | --- | --- | ---
Key Hash | 0x01 | Key Hash (20) | [Address reuse](https://en.bitcoin.it/wiki/Address_reuse) undermines the security and privacy of the users.
Script Hash | 0x02 | Script Hash (20) | [Address reuse](https://en.bitcoin.it/wiki/Address_reuse) undermines the security and privacy of the users.
Payment Code | 0x03 | Payment Code (80) | 
Stealth Keys | 0x04 | Spend Key (33) View Key (33) | *(The keys are compressed, No formal specification exist, needs a reference and peer review)*


## Security and Scalability

While it is technically possible for a client to download the referenced block, scan all transactions and look for matching transactions to parse the payment data, doing so at scale might be too costly for light clients. To optimize this process it is expected that 3rd parties provide indexing services.


### Indexing Services

A service can be made that continously scans the blockchain to create a database of valid **Cash Accounts**, indexed by their **Account Names** and **Block Heights**. To query such a service, the wallet/client should send a request for only the **Account Name** and **Block Height**, even if it is aware of a collision:

*By always omitting any **Collision Avoidance Parts** the indexing services cannot know which account is being looked for which increases privacy, and while an indexing service can always lie by omission, doing so without knowing which entry is being looked for adds a detection risk.*

```
{
    "name": "<lookup_name>",
    "block": <block_height>
}
```

The service should reply with a list of matches including all information necessary for an SPV wallet to independently verify each entry, such as the **Register Transaction**, **SPV Inclusion Proof** and **Block Information**. The client may then locally resolve any naming collosions or present a list of the resulting **Cash Accounts** with as much context information as reasonable, such as **Account Age**, **Collision Avoidance Part** and **Previous Interaction** for this account.

```
{
    "name": "<lookup_name",
    "block": <block_height>,
    "result":
    {
        "payment_data":
        {
            "key_hash": "",
            "payment_code": ""
        },
        "transaction":
        {
            "id": "967d009ee103340d6762819ebf452107561423fe97b04fbf501594f231e212c4",
            "data": ""
        },
        "inclusion_proof": 
        {
            "blockheight": 369863,
            "position": 2917,
            "merkle": 
             [
                "d844420b0f01398953b809b844bc9a5987f41d1373dab3180b6b4fe4de8633c4",
                "3c1f63dae13e84aaba94d6ee12c7b48fa7b470eacfbe6e183ba008b7cbc3725c",
                "0c5141f62f1ad58a6ebcee98868451ff392bc49bfb6daa5e4f492b6d175812cf",
                "e414fd7ee39a83b7bd524f98ed09efa279a97fecbeeca68087de384eda71fc31",
                "47fbf181989f2549d0d7a9fee2337ea60295059df0c0267cca173fd43b8e8596",
                "bc441c955a6d9bc629e3a8e428aa7670d2bf631a9d11887bb4ad6b534fcf4817",
                "1985800e12a150e6196ac07119278ea84ad296f1d8b2c88b2c85f61035f6a7cd",
                "fc02dfab3083ca9185bce4e8b9357b5aacacce2172c969ac543da04eaa1c0c1d",
                "6a36f10f34da6d66fdf75d4d97e0796106e7594409d354b8419a248231ab6935",
                "cbc02af67b294fe7bad08a69f9435b520f8f89fa221583e0a84c8d9c9dc469a0",
                "1043c7a6b9b4c42243e84a49a1f40476692b5d36d2f64dfb14409490ca3dfad2",
                "fbbaec1ca9d29ba9ee6a89214428556827b4e5a254de59de5adabbabb2621f54"
            ]
        }
    }
}
```


### Payment Type Preference

When a sending client looks up a **Cash Account** and finds more than one compatible **Payment Data**, it should make a best-effort choice to maximize the **security** and then **privacy** of the recepient, regardless of which order the **Payment Data** was stored in.

The reasoning for this is that the data is immutable and software is not, so the senders wallet may be aware of security and privacy issues that the recipient has not yet become aware of.


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
