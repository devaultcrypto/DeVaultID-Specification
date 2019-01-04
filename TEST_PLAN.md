# Test Plan

## Overview

### Data Entries

* 1: `Jonathan#100;`
* 2: `Timothy#200;`
* 3: `ABC#124.9;`
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
1 | 563720 | 590d1fdf7e04af0ee08f9194bb9e8d19<br/>71bdcbf55d29303d5bf32d4eae5e7136 | 6a 04 01010101 08 4a6f6e617468616e 15 01 <br/> ebdeb6430f3d16a9c6758d6c0d7a400c8e6bbee4
2 | 563820 | 17d080c925662159f806a75ad9394c58<br/>f482348bad9dbb5192e129ca6a3503da | 6a 04 01010101 07 54696d6f746879 4c 51 03 <br/> 01000345765a84a03c288708eb35e53e4cecfd0f113db8968235a4c3887f97dbceada0e20064dab0<br/>909bcad5ecd7388cddf3ae0cc57fa57135418230974002f9504d1e00000000000000000000000000

#### Account Identity

Entry | Identicon | Account Name | Account Number | Collision Hash
--- | --- | --- | --- | ---
1 | ☯ | Jonathan | #100 | .5876958390
2 | 🚀 | Timothy | #200 | .7910708913

#### Account Collision

Entry | Collision Count | Shortest Identifier
--- | --- | ---
1 | 0 | Jonathan#100;
2 | 0 | Timothy#200;

#### Payment Data

Entry | Payment Count | Payment Types | Payment Data
--- | --- | --- | ---
1 | 1 | Key Hash | qr4aadjrpu73d2wxwkxkcrt6gqxgu6a7usxfm96fst
2 | 1 | Payment Code | PM8TJa84G8yD81hRCF7kWoDcXDEgSdYNXWWe26<br/>HRGzwszeC1uc7JwUyZkqkTSQ2sjRXPEqVAST9aN<br/>4wrmXfvf59vzsDxCHbVuwB1oe5gKnR2nfkVvhcc