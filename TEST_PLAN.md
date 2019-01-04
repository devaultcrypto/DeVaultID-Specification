# Test Plan

## Overview

### Data Entries

* 1: `Jonathan#100;`
* 2: `ABC#124.9;`
* 3: `Timothy#200;`
* 4: `Monsterbitar#123;`
* 5: `Alice#76;`
* 6: `خرجمن#102`
* 7: `?#?`
* 8: `?#?`
* 9: `?#?`

### Test Vectors

#### Account Validity

Account | Validity | Notes
---|---|---
1 | Valid | Key hash
2 | Valid | Payment code
3 | Valid | Multiple payment entries
4 | Valid | Complete naming collision
5 | Valid | Naming collision without case folding
6 | Valid | OP_RETURN last in the transaction
7 | Invalid | Invalid account name
8 | Invalid | Invalid payment data
9 | Invalid | Registered before release

#### Account Transaction

Entry | Blockheight | Transaction ID | Registration HEX
--- | --- | --- | ---
1 | 

#### Account Identity

Entry | Identicon | Account Name | Account Number | Collision Hash
--- | --- | --- | --- | ---
1 | 

#### Account Collision

Entry | Collision Count | Shortest Identifier
--- | --- | ---
1 | 

#### Payment Data

Entry | Payment Count | Payment Types | Payment Data
--- | --- | --- | ---
1 | 