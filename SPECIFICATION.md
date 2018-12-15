**This is a draft specification - please do not use or share until finalized.**


# Abstract

Cash Accounts is a naming system that can be used alongside regular bitcoin addresses and payment codes to simplify the process of sharing payment information.


# Motivation

The Bitcoin address system based on hashing data creates complex and difficult to share identifiers. While these identifiers have proper checksums and misspellings are rare, they are still very cumbersome to transfer over the telephone, in a regular chat or similar. Many attempts have been made to obfuscate the addresses by transferring them as QR codes or NFC tags, but the need for a human accessible format remains.


# Specification

## Introduction

When a user wants to, they can create a **Cash Account** by selecting a suitable name and publish a transaction on-chain to aquire an identifier. Once the transaction is included in a block the user is informed of their name and identifier and can now share this information in a convenient manner with others.

When a user receives a **Cash Account Identifier** their wallet looks up the **Payment Information** that corresponds to the account.


### Cash Account Identifiers

### Complete Idenfitiers

A **Complete Identifier** consists of an **Account Name**, a **Account Number** and a **Collision Hash**.

```
Alice#662.338697598;
```

**Part** | **Example** | **Description**
--- | --- | ---
Account Name | Alice | Human readable account name
Account Number | #662 | Number separating accounts with the same name in different blocks.
Collision Hash | .338697598 | Number separating accounts with the same name in the same block.

#### Account Name

The **Account Name** is chosen by the user at registration time and can be at most 99 characters long.


#### Account Number

The **Account Number** calculated by taking the **Block Height** of the block that mined the **Registration Transaction** and substracting an arbitrary value. This value will be determined when the specification is finalized and publicly released and will be chosen such that an account registered in the first block of 2019 will have a **Account Number** of 100.

For examples in this draft specification, the number is **560000**.

Digits | Range | Expected availability
--- | --- | ---
3 | 100~999 | ~7 days
4 | 1000~9999 | ~2 months
5 | 10000~99999 | ~2 years
6 | 100000~999999 | ~19 years
7 | 1000000~9999999 | ~190 years


#### Collision Hash

The **Collision Hash** is used to resolve naming collisions within the same block and is calculated as follows:

```
Step 1: Concatenate the block hash with the transaction hash
Step 2: Hash the results of the concatenation with sha256
Step 3: Take the first four bytes and discard the rest
Step 4: Convert to decimal notation and store as a string
Step 5: Reverse the the string so the last number is first
```

* Note: the process might be able to be simpler by utilizing a HMAC to combine the purposes of step 1+2


### Minimal Identifiers

Most of the time it is expected that **Account Names** are uniquely registered in their block heights. This allows the shortest identifier to consist of only the **Account Name** and **Account Number** to form a simple human-accessible **Minimal Identifier**.

```
Alice#662;
```

#### Short Identifiers

It is possible that two or more users register the same name in the same block. To uniquely identify such accounts we need to extend the **Minimal Identifier** with a **Collision Avoidance Part** consisting of as many of the initial digits of the **Collision Hash** as required to resolve the naming collision, creating a **Short Identifier**.

```
Alice#662.33;
Alice#662.37;
```

* *Wallets should ideally poll an indexing server or lookup names in their local mempool to avoid naming collisions when possible*
* *When a wallet detects naming collisions during registration, it may opt to re-create the account in a later block to get a simpler identifier.*


## Protocol 

To register a **Cash Account** you broadcast a **Bitcoin Cash** transaction with an OP_RETURN output cointaining a **Protocol Identifier**, an **Account Name** and one or more **Payment Data**.

```
OP_RETURN (0x6a)
    OP_PUSH (0x04) <PROTOCOL> (0x01010101)
    OP_PUSH <ACCOUNT_NAME>
    OP_PUSH <PAYMENT_DATA> [...OP_PUSH <PAYMENT_DATA>]
```

### Protocol Identifier

This protocol adheres to the [OP_RETURN Prefix Guidelines](https://github.com/Lokad/Terab/blob/master/spec/opreturn-prefix-guideline.md) and uses the 0x01010101 protocol identifier.

### Account Name

The **Account Name** is an UTF-8 encoded string with a character length between 1 and 99, and a byte length small enough to allow for the desired **Payment Data**. Furthermore it is recommended that clients enforce a strict **Regular Expression** of ```/\w{1,99}/``` to the name to retain the human accessibility trait.

Presentation of **Account Names** should always be in the case that they are stored in while collision checks must always be done in lower case.

**TODO:** *The same transformations has to be used on both the client and server side, and across indexing servers. Furthemore, it might be worthwhile to look into: https://en.wikipedia.org/wiki/Unicode_equivalence*


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

A service can be made that continously scans the blockchain to create a database of valid **Cash Accounts**, indexed by their **Account Names** and **Account Numbers**. To query such a service, the wallet/client should send a request for only the **Account Name** and **Account Number**, even if it is aware of a collision:

*By always omitting any **Collision Avoidance Parts** the indexing services cannot know which account is being looked for which increases privacy, and while an indexing service can always lie by omission, doing so without knowing which entry is being looked for adds a detection risk.*

```
{
    "name": "Alice",
    "block": 560662
}
```

The service should reply with a list of matches including the full **Register Transactions** and **Inclusion Proofs** necessary for an SPV wallet to independently verify data. The client may then locally resolve any naming collosions or present a list of the resulting **Cash Accounts** with as much context information as reasonable, such as **Account Age**, **Collision Avoidance Part** and **Previous Interaction** for this account and let the user choose which to use.

```
{
    "name": "Alice",
    "block": 560662,
    "results":
    [
        {
            "transaction": "0100000001243fb2a33149dc9755949870e919c5b07317f9eeb279eff910daaa6a0ce6400d010000006a47304402203678134dff42e170125e97217df08a9eb177be4aa48208f72ad138eb463a995402204a2aef94e894eff7c54a38e3717571d1a5be9957b8e79e3ae9b76a040b815bd7412103466c5a4c9f64754f5023006b29839d1c2a5f2e063f58d5ddc84185a606172dfffeffffff020000000000000000226a040101010105416c6963651501ebdeb6430f3d16a9c6758d6c0d7a400c8e6bbee4c80d0800000000001976a9141fb7c820560a49d54c2e841ebc0b929639b91f7f88ac158e0800",
            "inclusion_proof": "0000c0207b65934755a2df17e7348a7e58c149ae5dfbdeb1faa3ea0200000000000000005767d9bcbbf3237b1db097ee7d3dc353722b8524bc03c131bf4455cea941f22ec293125ca030071869405157180200000be9a17b095ec090301730c6f9741f71e9e3fafb8a5c9004b34b0252b93f7cf6ee7705b5f132f690b7012f454afbf5929ef98c929eeeedd37ec0aa7fa07f647b851e1aa419ae5ce0f83e8c1cbf3ebe58865aef8b7afa99f70af4c0aad0418ccf00ec4982787de8e63c618842885fd23af3e71a4c4dc72f0a1dd45c8560f9b508014fd1ef4a33fc2f70dd583c8543635b858ba22b31af752daa3947df9a37e3ed6a0a4e90681594bcd2e7dd07facb5c37959f0aac2763584a0980169e85b760b10f3a510715955b79238b53aa0e3bbdf7a381606c496280d7750cc584bd8bf175ff1fe1cd26f962a8033f867d5cb2bf31a36bdd1bae3637e2fdc222238c7d1705614d6a8c68e12ff6015b17cf293488de51a379070c71321bb2acaa1d603a279e108caf09e663a52efa794149c91856ede90d150bd52add1b3d145e68f84b5e041fac33026e3b27f44eb6ae8907aeea1d4cd470d1198a1c693aea9e7e83273d02bc03ff2a00"
        }
    ]
}
```


### Payment Type Preference

When a sending client looks up a **Cash Account** and finds more than one compatible **Payment Data**, it should make a best-effort choice to maximize the **security** and then **privacy** of the recepient, regardless of which order the **Payment Data** was stored in.

The reasoning for this is that the data is immutable and software is not, so the senders wallet may be aware of security and privacy issues that the recipient has not yet become aware of.

### Pre-Confirmation

For new wallet users, waiting for a block confirmation is unreasanable before using their wallet. During the pre-confirmation time period the wallet should either fall back on other means of transferring the payment information, or should digitally share either the full registration transaction or the hash of the transaction so that the other party can look it up in their mempool.

Sharing the hash of the transaction allowed the other party to set up and store the finalized **Cash Account Identifier** in a local registry once it confirms in a block.


### Known Attacks

#### Reactive Collisions

When someone broadcasts a registration transaction, an attacker can inspect the transaction for the account name and then broadcast additional registration transactions with the same name for inclusion in the same block. This forces use of the **Collision Avoidance Part** and prevents the original user from acquiring a minimal identifier.

One attack objective would be to disrupt usability by increasing the length of the **Collision Avoidance Part**. For example:

```
Original registration (after confirmation in a block)
Alice#100.1234567890

Disruptive registrations : Required Short ID for original registration
Alice#100.__________     : Alice#100.1    (No same prefix digits ==> 1 digit Short ID)
Alice#100.1_________     : Alice#100.12   (Same 1 prefix digit   ==> 2 digit Short ID)
Alice#100.12________     : Alice#100.123  (Same 2 prefix digits  ==> 3 digit Short ID)
Alice#100.123_______     : Alice#100.1234 (Same 3 prefix digits  ==> 4 digit Short ID)
...
```

However, because the **Collision Hash** depends on the block hash, the attacker cannot predict it and must submit many attack registrations to expect matching prefix digits.

Given 1 original registration, on average the attacker can force `d` digits of the **Collision Avoidance Part** with 10<sup>(`d`-1)</sup> attack registrations. That is, exactly 1 transaction is required to force `d = 1`, and then on average 10 transactions are required to force `d = 2`, 100 for `d = 3`, etc. The cost for the attack scales directly with the number of attack registrations.

```
attack_tx = 10^(d-1)
bch_per_tx = 0.00000255
attack_bch = attack_tx * bch_per_tx
```
| Collision length `d` | Attack tx (avg) | Attack BCH (avg) |
|---------------------:|----------------:|-----------------:|
|                    1 |               1 |       0.00000225 |
|                    2 |              10 |        0.0000225 |
|                    3 |             100 |         0.000225 |
|                    4 |           1,000 |          0.00225 |
|                    5 |          10,000 |           0.0225 |
|                    6 |         100,000 |            0.225 |
|                    7 |       1,000,000 |             2.25 |

Additionally, a user can mitigate attacks as well as increase privacy by broadcasting `r` registrations and choosing any one of them after block confirmation. The attacker cannot know which registration will be used and therefore must attack all registrations. In other words, the user can greatly increase the cost for the attacker with a small number of mitigation transactions.

Note: Mitigation registrations interact with themselves and with attack registrations, reducing the actual attack cost. A more formal treatment of the problem should be made but the fundamental resistance to the reactive collision attack remains. Below is a table of approximate attack cost when the user makes mitigation registrations:

User registrations `r` | Attack BCH @Collision length `d = 1` | `d = 2` | `d = 3` | `d = 4` | `d = 5`
---: | --- | --- | --- | --- | ---
1 | 0.00000225 | 0.0000225 | 0.000225 | 0.00225 | 0.0225
10 | 0.0000225 | 0.000225 | 0.00225 | 0.0225 | 0.225
100 | 0.000225 | 0.00225 | 0.0225 | 0.225 | 2.25
1,000 | 0.00225 | 0.0225 | 0.225 | 2.25 | 22.5
10,000 | 0.0225 | 0.225 | 2.25 | 22.5 | 225
100,000 | 0.225 | 2.25 | 22.5 | 225 | 2,250
1,000,000 | 2.25 | 22.5 | 225 | 2,250 | 22,500


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
