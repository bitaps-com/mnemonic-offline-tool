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
This BIP is aimed at solving the problem by storing the code by dividing it into parts according to the threshold 
scheme for sharing the secret, as well as a description of how the mnemonic code can be generated manually without 
trusting any hardware/software random generators. The scope of application of this BIP is the creation and storage of
 cryptocurrency wallets backup for personal use.


### Manually generation

The mnemonic phrase is inherently a backup of access to the wallet. To ensure reliable backup storage developed 
metal backups that allow you to save information after exposure to aggressive environments and temperatures. 
But this approach does not protect mnemonic code against theft or single point trust problem. 
The threshold secret sharing scheme significantly improves secure backup storage against a single storage point trust problem.

In cryptography, several secret sharing schemes are known. The most famous is Shamir's secret scheme and Blakley's scheme.
This BIP is based on Shamir's scheme. With a more detailed description of Shamir's scheme, can be found on 
If you do not trust any software random number generator due to possible bugs or backdoors, you can generate a 
mnemonic code using dice yourself. Critical to security is the use of random numbers. Do not try to select words from the mnemonic dictionary in other ways. Even if it seems to you that you are unpredictably doing this, most likely you are mistaken.

In previously known dice generation methods, a large secret number is generated and formatted as a hexadecimal string.
Later this hexadecimal string is converted into a mnemonic phrase using a software tool. 

This BIP suggests generate a mnemonic phrase using dice in a simpler and more obvious way using the Dice wordlist table.

In the proposed generation scheme, it is most convenient to use 5 cubes at a time. If you have only 1 die, you should make 
5 consecutive throws to generate 1 word. With manual dice code generation, it is not possible to calculate the checksum, 
which is written at the end in accordance with BIP39.  Wallets and splitting schema suggested below, requires the correct mnemonic code checksum,
the last word MUST be adjusted to correct checksum using a software tool. In case user provided incorrect last word,
 wallet software should adjust correct checksum on the fly before converting mnemonic to binary seed. 

You should select 12 / 15 / 18 / 21 or 24 (recommended) words using dice rolls and dice wordlist table. If your throw result is not on the list, just ignore and continue. Some combinations are excluded for a uniform probability distribution.

Five dices with 6 sides give 6 ^ 5 = 7776 possible combinations. Wordlist is 2048 words, that overlaps three times range 
for  7776 combinations, with 2048 * 3 = 6144 good values, rest range (6144, 7776] are excluded.
Each word in the table corresponds to 3 roll combinations.

[BIP39 Dice word list](https://bitaps.com/dice/wordlist)


### BIP39 checksum

BIP39 describes an algorithm for converting a secret value (BIP32 seed) into a mnemonic code. This algorithm provides 
the calculation of the checksum, which is written at the end and affects the last word in the code. This checksum 
is excessive since there is no practical benefit from it. The number of bits in a checksum is so small that it does
 not allow you to restore the word order in case it is lost. In the case of a typo, the BIP39 wordlist itself allows
  you to correct it. Additionally, the presence of a checklist makes it impossible to generate a mnemonic phrase 
  manually for wallets that strictly control checksum. The checksum MUST be adjusted on the fly by Wallets to enable manually
   generation and protect privacy with the opportunity for plausible deniability.
   
### Splitting mnemonic code

The mnemonic phrase is inherently a backup of access to the wallet. To ensure reliable backup storage developed 
metal backups that allow you to save information after exposure to aggressive environments and temperatures. 
But this approach does not protect mnemonic code against theft or single point trust problem. 
Threshold secret sharing scheme significantly improve secure backup storage against single point trust problem.

In cryptography, several secret sharing schemes are known. The most famous are the Shamir's secret scheme and Blakley's scheme.
This BIP is based on the Shamir's scheme. With a more detailed description of Shamir's scheme, scan be found on 
[Wikipedia](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing).

There are already exists two great implementation of the Shamir scheme for mnemonic codes. First one is 
[SLIP39](https://github.com/satoshilabs/slips/blob/master/slip-0039.md) from Satoshi Labs,
and second one is [shamir39](https://github.com/iancoleman/shamir39/blob/master/specification.md) from 
[iancoleman](https://github.com/iancoleman).

The main idea of creating this BIP is to create the most simple and understandable scheme for mass use.
SLIP39 is a complicated scheme with division into groups and division within groups into shares, a checksum is used to validate shares. 
This scheme is more likely focused on corporate users. Iancoleman scheme is more simple and more suitable for mass use.

This BIP proposes technical implementation Shamir's secret sharing scheme applied separately to each byte of the shared
 secret and GF(256) is used as the underlying finite field. Secret stored in share f(0). On GF(256) finite field 
255 shares are maximal possible. For the need for personal use, this is more than enough.
Share index (x coordinate) should be randomly selected to prevent any information leak about sharing split schema.

### Mnemonic code share

The share in appearance and format should not differ from the usual mnemonic code, to protect privacy. 
To recover the mnemonic code from shares, you need to store and know the share indexes (x coordinate). 
Since the index is randomly selected and the checksum designed in BIP 39 is redundant, the index can 
be written into bits reserved for checksum without any leaks about splitting scheme and breaking existed design. 
Since the number of bits for checksum varies depending on the number of words of the mnemonic code, we have 
a limit for the maximum number of total shares in the secret sharing scheme, depending on the length of the mnemonic code word.

    - 12 words: 4 bits -> 15 total shares
    - 15 words: 5 bits -> 31 total shares
    - 18 words: 6 bits -> 63 total shares
    - 21 words: 7 bits -> 127 total shares
    - 24 words: 8 bits -> 255 total shares

The range for random index selection MUST be limited by the maximal number of total shares.

### Reference implementation


<code>

    S.__split_secret = (threshold, total,  secret, indexBits=8) => {
        if (threshold > 255) throw new Error("threshold limit 255");
        if (total > 255) throw new Error("total limit 255");
        let index_mask = 2**indexBits - 1;
        if (total > index_mask) throw new Error("index bits is to low");
        if (threshold > total) throw new Error("invalid threshold");
        let shares = {};
        let sharesIndexes = [];
        let e = S.generateEntropy({hex:false});
        let ePointer = 0;
        let i = 0;
        let index;
        // generate random indexes (x coordinate)
        do {
           if (ePointer >= e.length) {
               // get more 32 bytes entropy
               e = S.generateEntropy({hex:false});
               ePointer = 0;
           }
           index = e[ePointer] & index_mask;
           if ((shares[index] === undefined)&&(index !== 0)) {
               i++;
               shares[index] = BF([]);
               sharesIndexes.push(index)
           }
           ePointer++;
        } while (i !== total);

        e = S.generateEntropy({hex:false});
        ePointer = 0;
        let w;
        for (let b = 0; b < secret.length; b++) {
            let q = [secret[b]];
            for (let i = 0; i < threshold - 1; i++) {
                do {
                    if (ePointer >= e.length) {
                        ePointer = 0;
                        e = S.generateEntropy({hex:false});
                    }
                    w  = e[ePointer++];
                } while (q.includes(w));
                q.push(w);
            }
            for (let i of sharesIndexes)
                shares[i] = BC([shares[i], BF([S.__shamirFn(i, q)])]);

        }
        return shares;
    };
    
</code>

[shamir_secret_sharing.js](https://github.com/bitaps-com/jsbtc/blob/master/src/functions/shamir_secret_sharing.js)

[bip39_mnemonic.js](https://github.com/bitaps-com/jsbtc/blob/master/src/functions/bip39_mnemonic.js)

## References

* [Tool for mnemonic code generation/splitting](https://bitaps.com/mnemonic)
* [Offline version: Tool for mnemonic code generation/splitting](https://bitaps.com/mnemonic/offline)
* [BIP39 Dice word list](https://bitaps.com/dice/wordlist)
