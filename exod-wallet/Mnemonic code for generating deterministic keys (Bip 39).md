BIP39: Mnemonic -> Seed

Issue: A wallet's root secret needs to be high entropy. but would be hard to keep such random 32bits charactoer. one wrong hex digit and you silently get a different valid looking wallet.

The goal is a way to transport computer-generated randomness with a human-readable transcription.

This involves:
- Generating an entropy: 128 -> 12 word mnemonic, 256 -> 24 word mnemonic
- Append a checksum: SHA256(entropy). ... ENT / 32 bits. appended to the end of the initial entropy.
- Chop into 11 bit chunks: 132 bits/11 = 12 chunks. each chunk is a number from 0 - 2047 which indexes into a fixed 2048 word list to produce one word.
- That's your mnemonic. it's not a seed yet (it's an encoding of the entropy)
- Stretch it to the actual seed: PBKDF2-HMAC-SHA512(password = mnemonic sentance, salt, iterations, output length) => 512 bit output --- the real seed the modules 1's BIP32 step 

Mnemonic codes are a group of easy to remember words, for generation of deterministic wallets.
1. Generate the mnemonic
2. Convert it to a binary seed.
	1. This seed is used to generate deterministic wallets using BIP-0032 or similar methods

```
ent => entropy
cs => checksum
ms => mnenomic sentence

CS = ENT / 32
MS = (ENT + CS) / 11

|  ENT  | CS | ENT+CS |  MS  |
+-------+----+--------+------+
|  128  |  4 |   132  |  12  |
|  160  |  5 |   165  |  15  |
|  192  |  6 |   198  |  18  |
|  224  |  7 |   231  |  21  |
|  256  |  8 |   264  |  24  |
```

Wordlist
This has a smart collection of words
Similar words avoided
sorted wordlists for efficient lookup of the code words

[From mnemonic to seed](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki#from-mnemonic-to-seed)
Mnemonics can be protected using a passphrase. if not present, an empty string is used instead.
To create binary seed from the mnemonic, we use the PBKDF2 function with a mnemonic sentence.
This seed can later be used to generate deterministic wallets using BIP-0032 or similar methods

The conversion of mnemonic sentence to a binary seed is fully independent from generating the sentences. This results in a rather similar code
Clients are free to implement their own wordlists
