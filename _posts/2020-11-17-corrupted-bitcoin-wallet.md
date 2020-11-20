---
layout: post
date: 2020-11-17 09:00:00 UTC
title: Corrupted Bitcoin Wallet + 30 Lines Of Code = 3000 USD
description: Extracting private keys from a corrupted Bitcoin wallet.
---

# The Story

A friend of a relative had brought in their Macbook Air, and aske me to repair it.
This required me to download the latest MacOS for which I didn't have enough space for.
In a desperate attempt, I decided to clean up my external hard drive, removing files that were taking up too much storage.

This is the exact moment that karma had found its way to my 4 AM adventure to fix this MacBook.

To my surprise I found a folder named "Btc", where I stored episode of television shows. 
Apparently it has been there since 2015 - this must have been my very first wallet!

Sadly, the wallet wouldn't open in bitcoin core, it has been corrupted.
The `db.log` outputs the logs for the BerkeleyDB wallet file, it indicated the following error message:
```
file wallet.dat has LSN 780/2077553, past end of log at 1/333
Commonly caused by moving a database from one database environment
to another without clearing the database LSNs, or by removing all of
the log files from a database environment
DB_ENV->log_flush: LSN of 780/2077553 past current end-of-log of 1/333
Database environment corrupt; the wrong log files may have been removed or incompatible database files imported from another environment
```

Therefore I was forced to salvage the wallet somehow.
The `Bitcoin Core` software has an option `-salvagewallet` which did not appear to help.

A bit of googling around and I found out that uncompressed private keys are prepended with `01 01 04 20`.
Time to take a quick scroll through the file in `HxD`, a popular hex editor!

![](/res/corrupted-bitcoin-wallet/HxD-search.png)

![](/res/corrupted-bitcoin-wallet/HxD-search-2.png)

I didn't expect to get 145 hits, but what I certainly didn't want to do was manually converting all of those hits to private keys and checking the balance.

![](/res/corrupted-bitcoin-wallet/HxD-search-3.png)

The private key is 32 bytes long and is just after our prefix.
```
ffb2b2ed484fa1d5c9b2f4ad8e7ec696
```

So I got to work, the result is below.


# Usage
```
pip install bit
```

Take the code below and put in a file named `salvage.py`
**Do not forget to uncomment and change the address variable on line 7**.

Finally, run the script!
```
python salvage.py
```

It will output something like this, and if it finds funds, will automatically transfer them over to the address your specified.
PS: do not post this publicly, the first string is the `private key`, followed by the `address` and finally the `amount of btc`.
```
91236845f793a1aabd01c1d9fe615b8b 14uxHyEs1rKcPTYxnxMFW9soVTgvQsGvZq 0
323a961695929abf201386aaf2187e1 1tEiRCzpR4fKzvTejZV3MJhymx1iDmHNd 0
f94048834d85608f3a822aa84799fb8f 1JgH79RGzm9QwbYKU8LSs1hiDHsNdvrhR 0
365480736196084e8d6c7bf7b5a5943d 16CUqJjUEpkE7n34UKjTGBn5ZWAC1EGbep 0
636c0638ff10d8d19e9f36b5bed1e06 1MefrcgGg8cuSRSFbjxBwMsid6FAXf2FNb 0
aba4471c95f8217c3bbfdfddde4e17c0 1GfuKWVfk9jcu9hoNekP8DxCu2AUvDWsH3 0
a46802436a1722a0494ea88c391e2358 1CZ1LepDyk4rbD9tbYpUW1fpCJfjGuWb9J 0
23cf4f0d928d0cbcb3791ff659d81ed7 1KvCXp43MxHZGA1uAZeL1qEww8Zvx7syHd 0
d9310b543473af26aaab9e7400b4070 1B1z1GDyUPCg63NwPr4JRNg9omrHnmtcgW 0
b19a010122ce93ced3f5889109e5af99 1HDs9h8SwZn95kBTU3VWZqRfcs1ZBUe8de 0
12fd6615262cb26dc7ef8b1b7be27b8f 1E3HW9Bb6GiD9d79oJsMZB52j9dXshmhzv 0
19cad87841864304a23812a893c8dfe0 18gxN33f8KYGWH9NxrJo1FBkAVq4L2im9B 0
2ede1602dc2a0574c72342362a2c446c 1C4hCdQhy1yEvRWv2qNVJkNmz61bfCPNAZ 0
a3b68584e2d9350e349a29afa555cf2b 16icLfzH2JPrzWEyNzw8Y7FRR2uUH68fG2 0
8f9fa69c9c4098ed98c7ec805131fd4f 1CZfJ3gfvfwPcfesKPxCTJo6PGAw5a4wFS 0
2d022112ea9d6710d08fcb4c695af606 1BSuQypnh14EFhZfqRVN9djjsU7yKSuEhQ 0
da406f8761d993d7138ee75b10e7600b 1LxAr1SBFmbQZacinq5qZhPSghM1Ncq33W 0
8ac6891b9ad67f860e258e82fe24aeaa 18UCnSmxLkjx6vQWULzEVX3wDyEojiFybr 0
54a005c84d9653e19771494c82360861 18VUZhJu5nSuP8ovJDejaVTrYZLaSL4q68 0
9c71627b14ac35b7a0b26dd316c4a324 1HyHdi6suCYDxMPo4vwio55KksHz34Pok4 0
daa514c0f215cb9ac314e47c214be022 1PNMuWtNoi5qdkNRW7P8Wg8b4SBuQxheiZ 0
c0292135b0df7fe7c11d2a76c3ae288b 1F4fEjExUneyDZxjHqRWofpBDf8yTiPyeJ 0
9baf68164b9b14dc1c9c10e49639fcfc 1BrKyAzwTBdVhjZgK4xqi7ZUMWKSD9jdSs 0
7f28390b03957683423956ad890bd0d8 1MbQauBHGR1JWeo9UdP49xaqYVcFP73Fw9 0
```

# The Code
```
from bit import Key
from bit.format import bytes_to_wif
import re

wallet_file = "wallet.dat"
# Uncomment for security, set the address and uncomment.
#address = "ADDRESS_TO_SEND_TO"

def salvage(priv):
    key = Key.from_hex(priv)
    wif = bytes_to_wif(key.to_bytes(), compressed=False)
    key = Key(wif)
    print(priv + " " + key.address + " " + str(key.get_balance('btc')))
    if (float(key.get_balance('btc')) > 0):
        tx = key.send([], leftover=address)
    return key

with open(wallet_file, "rb") as f:
    matches = re.findall(b'\x01\x01\x04\x20(.{32})', f.read())

for priv in matches:
    salvage(priv.hex())
```

# PyWallet
In retrospect, I could've used PyWallet but I decided not to.

## Stricter validation
The PyWallet implementation does a lot more checks before it decides that a key is "valid".
It seems to be using a concept of both "prefixes" and "suffixes" for the keys.
Additionally the prefix used in `PyWallet` is a lot larger, `308201130201010420` instead of `01010420`.
If by any chance a single character of the prefix and/or suffix get corrupted, it will not detect it as a key.
My approach is more aggressive in the sense that it will attempt anything that might look like a key.

## Unencrypted wallet
Additionally, my wallet was not encrypted, therefore it was a bit overkill to use PyWallet.

## The real reason
I was ony my Windows machine and I didn't feel like spending an hour to install dependencies like `python-twisted`, when I could get the job done with less code.

Nonetheless, PyWallet is a great tool and it is worth using for encrypted wallets.
