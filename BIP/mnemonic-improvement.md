# BIP-00XX : Mnemonic codes improvement

```
Number:  BIP-00XX
Title:   Mnemonic codes improvement
Type:    Standard
Status:  Draft
Authors: Aleksey Karpov <admin@bitaps.com>
Created: 2020-05-19
```

## Abstract

This BIP describes development recommendations for the generation of deterministic wallets and secure backup strategies.


## Motivation

Mnemonic codes proposed in BIP39 are the most successful solution for storing secret keys. 
A key factor is the use of a human-readable format and the zero chance of information loss due to typos.
This BIP is aimed at solving the problem by storing the code by dividing it into parts according 
to the threshold scheme for sharing the secret, as well as a description of how the mnemonic code 
can be generated manually without trusting any hardware/software random generators. Scope of application of this BIP 
is creation and storage cryptocurrency wallets for personal use.


### BIP39 checksum

BIP39 describes an algorithm for converting a secret value (BIP32 seed) into a mnemonic code.
This algorithm provides the calculation of the checksum, which is written at the end and affects the last word in the code.
This checksum is excessive since there is no practical benefit from it. 
The number of bits in a checksum is so small that it does not allow you to restore the word order in case it is lost.
In the case of a typo, the BIP39 wordlist itself allows you to correct it. 
Additionally, the presence of a checklist makes it impossible to generate a mnemonic phrase manually for wallets that 
strictly control checksum. Checksum MUST be ignored by Wallets to enable manually generation 
and protect privacy with opportunity for plausible deniability.



### Manually generation

If you do not trust to any software random number generator due possible bugs or backdoors, you can generate a 
mnemonic code using dice yourself. Critical to security is the use of random numbers. Do not try to select words from 
the mnemonic dictionary in other ways. Even if it seems to you that you are doing this in an unpredictable way, 
most likely you are mistaken.

In previously known dice generation methods, a large secret number is generated and formatted as a hexadecimal string.
Later this hexadecimal string is converted into a mnemonic phrase using software tool. 

This BIP suggests refusing to use cheksum verification and enable ability  to generate a mnemonic phrase using dice in 
a simpler and more obvious way using Dice wordlist table.

In the proposed generation scheme, it is most convenient to use 5 cubes at a time. If you have only 1 die, you should 
make 5 consecutive throws to generate 1 word. With manual dice code generation, it is not possible to calculate the 
checksum, which is written at the end in accordance with BIP39. If the wallet in which you plan to use your mnemonic 
code requires the correct code checksum, then you should adjust last word to correct checksum using software tool.

You should select 12 / 15 / 18 / 21 or 24 (recommended) words using dice rolls and dice wordlist table. If your throw 
result is not in the list, just ignore and continue. Some combinations are excluded for uniform probability distribution.

Five dices with 6 sides give 6 ^ 5 = 7776 possible combinations. Wordlist is 2048 words, that overlaps three times range 
from 1111 to 7776: 2048 * 3 = 6144 good values, first range 1 -> 1110 and last range 7255 -> 7776  excluded.
Each word in the table corresponds to 3 roll combinations.


### Splitting mnemonic code

The mnemonic phrase is inherently a backup of access to the wallet. To ensure reliable backup storage developed 
metal backups that allow you to save information after exposure to aggressive environments and temperatures. 
But this approach does not protect mnemonic code against theft or single point trust problem. 
Threshold secret sharing scheme significantly improve secure backup storage against single point trust problem.


to be continued ...



