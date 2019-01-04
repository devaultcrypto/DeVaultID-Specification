* Status: Released, version 1.0
* Activation blockheight: 563720
* Block Modification Value: 563620


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
Jonathan#100.5876958390;
```

**Part** | **Example** | **Description**
--- | --- | ---
Account Name | Jonathan | Human readable account name
Account Number | #100 | Number separating accounts with the same name in different blocks.
Collision Hash | .5876958390 | Number separating accounts with the same name in the same block.

#### Account Name

The **Account Name** is chosen by the user at registration time and can be at most 99 characters long.


#### Account Number

The **Account Number** is calculated by taking the **Block Height** of the block that mined the **Registration Transaction** and substracting a **Block Modification Value**.

The **Block Modification Value** is equal to the **Activation Block Height** minus 100 and was choosen such that new accounts would create **Account Numbers** starting with 100. 

Registrations in blocks before the **Activation Block Height** are automatically invalid and must not be used.

Digits | Range | Expected availability
--- | --- | ---
3 | 100~999 | ~7 days
4 | 1000~9999 | ~2 months
5 | 10000~99999 | ~2 years
6 | 100000~999999 | ~19 years
7 | 1000000~9999999 | ~190 years


#### Collision Hash

The **Collision Hash** is used to resolve naming collisions within the same block and is calculated as follows:

* **Block Hash**: `000000000000000002abbeff5f6fb22a0b3b5c2685c6ef4ed2d2257ed54e9dcb`
* **Transaction Hash**: `590d1fdf7e04af0ee08f9194bb9e8d1971bdcbf55d29303d5bf32d4eae5e7136`

```
Step 1: Concatenate the block hash with the transaction hash
=> 000000000000000002abbeff5f6fb22a0b3b5c2685c6ef4ed2d2257ed54e9dcb590d1fdf7e04af0ee08f9194bb9e8d1971bdcbf55d29303d5bf32d4eae5e7136

Step 2: Hash the results of the concatenation with sha256
=> 37f1d9b19e25be4d310f3d4a798091ad38a2fb4a3fb66a82ff1d12e2e29a6230

Step 3: Take the first four bytes and discard the rest
=> 37f1d9b1

Step 4: Convert to decimal notation and store as a string
=> 938596785

Step 5: Reverse the the string so the last number is first
=> 587695839

Step 6: Right pad the string with zeroes up to a string length of 10.
=> 5876958390
```


### Minimal Identifiers

Most of the time it is expected that **Account Names** are uniquely registered in their block heights. This allows the shortest identifier to consist of only the **Account Name** and **Account Number** to form a simple human-accessible **Minimal Identifier**.

```
Jonathan#100;
```

#### Short Identifiers

It is possible that two or more users register the same name in the same block. To uniquely identify such accounts we need to extend the **Minimal Identifier** with a **Collision Avoidance Part** consisting of as many of the initial digits of the **Collision Hash** as required to resolve the naming collision, creating a **Short Identifier**.

```
Jonathan#100.56;
Jonathan#100.51;
```

* *Wallets should ideally poll an indexing server or lookup names in their local mempool to avoid naming collisions when possible*
* *When a wallet detects naming collisions during registration, it may opt to re-create the account in a later block to get a simpler identifier.*


#### Account Identicons

As an optional feature clients may calculate and show an **Account Identicon** for each **Cash Account** from the following unicode emoji list:

*If the locally available fonts do not support all emojis, consider using the [Noto Color Emoji](https://www.google.com/get/noto/help/emoji/) font.*

 &#128123;&#128018;&#128021;&#128008;&#128014;&#128004;&#128022;&#128016;&#128042;&#128024;  &#128000;&#128007;&#128063;&#129415;&#128019;&#128039;&#129414;&#129417;&#128034;&#128013;
 
 &#128031;&#128025;&#128012;&#129419;&#128029;&#128030;&#128375;&#127803;&#127794;&#127796;  &#127797;&#127809;&#127808;&#127815;&#127817;&#127819;&#127820;&#127822;&#127826;&#127827;
 
 &#129373;&#129381;&#129365;&#127805;&#127798;&#127812;&#129472;&#129370;&#129408;&#127850;  &#127874;&#127853;&#127968;&#128663;&#128690;&#9973;&#9992;&#128641;&#128640;&#8986;
 
 &#9728;&#11088;&#127752;&#9730;&#127880;&#127872;&#9917;&#9824;&#9829;&#9830;  &#9827;&#128083;&#128081;&#127913;&#128276;&#127925;&#127908;&#127911;&#127928;&#127930;
 
 &#129345;&#128269;&#128367;&#128161;&#128214;&#9993;&#128230;&#9999;&#128188;&#128203;  &#9986;&#128273;&#128274;&#128296;&#128295;&#9878;&#9775;&#128681;&#128099;&#127838;

The identicon acts like a checksum which helps prevent against typing mistakes during user entry, but only has a probabilistic benefit in the sense that it is technically possible for a user to mistype their entry into another valid account, which then has a 1% chance of having the same identicon.


##### Account Identicon Calculation

To calculate which emoji to use for the **Account Identicon** perform the following steps:

* **Block Hash**: `000000000000000002abbeff5f6fb22a0b3b5c2685c6ef4ed2d2257ed54e9dcb`
* **Transaction Hash**: `590d1fdf7e04af0ee08f9194bb9e8d1971bdcbf55d29303d5bf32d4eae5e7136`
* **Emoji List**: `[ 128123, 128018, 128021, 128008, 128014, 128004, 128022, 128016, 128042, 128024, 128000, 128007, 128063, 129415, 128019, 128039, 129414, 129417, 128034, 128013, 128031, 128025, 128012, 129419, 128029, 128030, 128375, 127803, 127794, 127796, 127797, 127809, 127808, 127815, 127817, 127819, 127820, 127822, 127826, 127827, 129373, 129381, 129365, 127805, 127798, 127812, 129472, 129370, 129408, 127850, 127874, 127853, 127968, 128663, 128690, 9973, 9992, 128641, 128640, 8986, 9728, 11088, 127752, 9730, 127880, 127872, 9917, 9824, 9829, 9830, 9827, 128083, 128081, 127913, 128276, 127925, 127908, 127911, 127928, 127930, 129345, 128269, 128367, 128161, 128214, 9993, 128230, 9999, 128188, 128203, 9986, 128273, 128274, 128296, 128295, 9878, 9775, 128681, 128099, 127838 ]`

```
Step 1: Concatenate the block hash with the transaction hash
=> 000000000000000002abbeff5f6fb22a0b3b5c2685c6ef4ed2d2257ed54e9dcb590d1fdf7e04af0ee08f9194bb9e8d1971bdcbf55d29303d5bf32d4eae5e7136

Step 2: Hash the results of the concatenation with sha256
=> 37f1d9b19e25be4d310f3d4a798091ad38a2fb4a3fb66a82ff1d12e2e29a6230

Step 3: Take the last four bytes and discard the rest
=> e29a6230

Step 4: Convert to decimal notation
=> 3801768496

Step 5: Modulus by 100.
=> 96

Step 6: Take the emoji at the given position in the emoji list.
=> 9775
```


## Protocol 

To register a **Cash Account** you broadcast a **Bitcoin Cash** transaction with a single OP_RETURN output in any position, cointaining a **Protocol Identifier**, an **Account Name** and one or more **Payment Data**.

```
OP_RETURN (0x6a)
    OP_PUSH (0x04) <PROTOCOL> (0x01010101)
    OP_PUSH <ACCOUNT_NAME>
    OP_PUSH <PAYMENT_DATA> [...OP_PUSH <PAYMENT_DATA>]
```

### Protocol Identifier

This protocol adheres to the [OP_RETURN Prefix Guidelines](https://github.com/Lokad/Terab/blob/master/spec/opreturn-prefix-guideline.md) and uses the 0x01010101 protocol identifier and has to be pushed with the 0x04 opcode.

### Account Name

The **Account Name** is an ASCII encoded string with a character length between 1 and 99, and a byte length small enough to allow for the desired **Payment Data**. The string must adhere to a strict **Regular Expression** of `/[a-zA-Z0-9_]{1,99}/` to retain the human accessibility trait.

Presentation of **Account Names** should always be in the case that they are stored in while collision checks must always be done in lower case.


### Payment Data Types

**Payment Data** consists of a **Payment Identifier** and **Payment Information**.

**Name** | **Payment Identifier** | **Payment Information** | **Notes**
--- | --- | --- | ---
Key Hash | 0x01 | Key Hash (20) | [Address reuse](https://en.bitcoin.it/wiki/Address_reuse) undermines the security and privacy of the users.
Script Hash | 0x02 | Script Hash (20) | [Address reuse](https://en.bitcoin.it/wiki/Address_reuse) undermines the security and privacy of the users.
Payment Code | 0x03 | Payment Code (80) | 
Stealth Keys | 0x04 | Spend Key (33) View Key (33) | <reserved for stealth keys, pending a formal specification>


## Security and Scalability

While it is technically possible for a client to download the referenced block, scan all transactions and look for matching transactions to parse the payment data, doing so at scale might be too costly for light clients. To optimize this process it is expected that 3rd parties provide indexing services.


### Indexing Services

A service can be made that continously scans the blockchain to create a database of valid **Cash Accounts**, indexed by their **Account Names** and **Account Numbers**. An indexing server should thus validate the **Account Name** with the same strict **Regular Expression** (`/[a-zA-Z0-9_]{1,99}/`) as is applied to the registration. To query such a service, the wallet/client should send a request for only the **Account Name** and **Account Number**, even if it is aware of a collision:

*By always omitting any **Collision Avoidance Parts** the indexing services cannot know which account is being looked for which increases privacy, and while an indexing service can always lie by omission, doing so without knowing which entry is being looked for adds a detection risk.*

```
{
    "name": "Jonathan",
    "block": 563720
}
```

The service should reply with a list of matches using the same case folding transformation as is done for collision detection, including the full **Register Transactions** and **Inclusion Proofs** necessary for an SPV wallet to independently verify data. The client may then locally resolve any naming collosions or present a list of the resulting **Cash Accounts** with as much context information as reasonable, such as **Account Age**, **Collision Avoidance Part** and **Previous Interaction** for this account and let the user choose which to use.

```
{
    "name": "Jonathan",
    "block": 563720,
    "results":
    [
        {
            "transaction": "590d1fdf7e04af0ee08f9194bb9e8d1971bdcbf55d29303d5bf32d4eae5e7136",
            "inclusion_proof": "<???>"
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
attack_transactions = 10^(d-1)
bch_per_transaction = 0.00000255
attack_bch = attack_transactions * bch_per_transaction
```

| Forced digits `d` | Attack transactions | Attack BCH |
|------------------:|--------------------:|-----------:|
|                 1 |                   1 |  0.0000023 |
|                 2 |                  10 |  0.0000225 |
|                 3 |                 100 |  0.0002250 |
|                 4 |               1,000 |  0.0022500 |
|                 5 |              10,000 |  0.0225000 |
|                 6 |             100,000 |  0.2250000 |
|                 7 |           1,000,000 |  2.2500000 |

A user might try to mitigate attacks by broadcasting `r` total registrations and then choosing one after confirmation in a block. However, `r-1` of the registrations act effectively as attack registrations for the purpose of increasing collision digits.

If the attacker intends to disrupt the protocol as a whole, the attack becomes more expensive with increased usage:

Registrations | Attack BCH @ Forced digits `d = 1` | `d = 2` | `d = 3` | `d = 4` | `d = 5`
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

If **Alice** properly verifies that her identifier has the required length for her **Collision Avoidance Part**, then **Bob** will either fail to look up her **Cash Account**, or will be able to positively prove that her **Payment Data** corresponds to a valid **Registration Transaction**.


# References

**Name** | **BIP** | **Link**
--- | --- | ---
Bitcoin Script | N/A | https://en.bitcoin.it/wiki/Script
Reusable Payment Codes | BIP-47 | https://github.com/OpenBitcoinPrivacyProject/bips/blob/master/bip-0047.mediawiki
OP_RETURN | N/A | https://en.bitcoin.it/wiki/OP_RETURN
OP_RETURN Guidelines | N/A | https://github.com/Lokad/Terab/blob/master/spec/opreturn-prefix-guideline.md
