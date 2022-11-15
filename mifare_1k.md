# Mifare 1k

## Retrive data from original key

Place original key on hf antenna (lower deck opposite from lights and USB port)

Connect proxmark3 to computer via USB

Start [pm3 utility (iceman fork)](https://github.com/RfidResearchGroup/proxmark3)

### Determine card type
```bash
hf search
```

```log
[usb] pm3 --> hf search
 🕖  Searching for ISO14443-A tag...          
[+]  UID: <four bytes> 
[+] ATQA: <two bytes
[+]  SAK: <one byte> [2]
[+] Possible types:
[+]    MIFARE Classic 1K
[=] proprietary non iso14443-4 card found, RATS not supported
[+] Prng detection: hard
```

Save UID, ATQA, and SAK bytes for later

### Search for default keys
This will attempt to access each sector using the default key `FFFFFFFFFFFF`. Most cards have at least one sector using this key. All other keys should be easy to extract as long as one key is known.

```bash
hf mf chk
```

```log
[usb] pm3 --> hf mf chk
[=] Start check for keys...
[=] .................................
[=] time in checkkeys 6 seconds

[=] testing to read key B...

[+] found keys:

[+] -----+-----+--------------+---+--------------+----
[+]  Sec | Blk | key A        |res| key B        |res
[+] -----+-----+--------------+---+--------------+----
[+]  000 | 003 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  001 | 007 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  002 | 011 | FFFFFFFFFFFF | 1 | FFFFFFFFFFFF | 1
[+]  003 | 015 | FFFFFFFFFFFF | 1 | FFFFFFFFFFFF | 1
[+]  004 | 019 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  005 | 023 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  006 | 027 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  007 | 031 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  008 | 035 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  009 | 039 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  010 | 043 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  011 | 047 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  012 | 051 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  013 | 055 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  014 | 059 | ------------ | 0 | FFFFFFFFFFFF | 1
[+]  015 | 063 | ------------ | 0 | FFFFFFFFFFFF | 1
[+] -----+-----+--------------+---+--------------+----
[+] ( 0:Failed / 1:Success )
```

As long as one result for key A is successful, retrieving all other keys is trivial. We can now autopwn the rest.

### Extracting unknown keys
> Note: this may take VERY long if no default keys were found in the previous step

`autopwn` attempts to recover all keys using fchk, chk, darkside, nested, hardnested and staticnested attacks in order of complexity/time.

```bash
hf mf autopwn
```

```log
[usb] pm3 --> hf mf autopwn
[!] ⚠️  no known key was supplied, key recovery might fail
...
<logs shortened for readability>
...
[+] found keys:

[+] -----+-----+--------------+---+--------------+----
[+]  Sec | Blk | key A        |res| key B        |res
[+] -----+-----+--------------+---+--------------+----
[+]  000 | 003 |  <redacted>  | H |  <redacted>  | D
[+]  001 | 007 |  <redacted>  | H |  <redacted>  | D
[+]  002 | 011 |  <redacted>  | D |  <redacted>  | D
[+]  003 | 015 |  <redacted>  | D |  <redacted>  | D
[+]  004 | 019 |  <redacted>  | R |  <redacted>  | D
[+]  005 | 023 |  <redacted>  | R |  <redacted>  | D
[+]  006 | 027 |  <redacted>  | R |  <redacted>  | D
[+]  007 | 031 |  <redacted>  | R |  <redacted>  | D
[+]  008 | 035 |  <redacted>  | R |  <redacted>  | D
[+]  009 | 039 |  <redacted>  | R |  <redacted>  | D
[+]  010 | 043 |  <redacted>  | R |  <redacted>  | D
[+]  011 | 047 |  <redacted>  | R |  <redacted>  | D
[+]  012 | 051 |  <redacted>  | R |  <redacted>  | D
[+]  013 | 055 |  <redacted>  | R |  <redacted>  | D
[+]  014 | 059 |  <redacted>  | R |  <redacted>  | D
[+]  015 | 063 |  <redacted>  | R |  <redacted>  | D
[+] -----+-----+--------------+---+--------------+----
[=] ( D:Dictionary / S:darkSide / U:User / R:Reused / N:Nested / H:Hardnested / C:statiCnested / A:keyA  )


[+] Generating binary key file
[+] Found keys have been dumped to hf-mf-YOUR_UID-key.bin
[=] FYI! --> 0xFFFFFFFFFFFF <-- has been inserted for unknown keys where res is 0
[+] transferring keys to simulator memory (Cmd Error: 04 can occur)
[=] downloading the card content from emulator memory
[+] saved 1024 bytes to binary file hf-mf-YOUR_UID-dump.bin
[+] saved 64 blocks to text file hf-mf-YOUR_UID-dump.eml
[+] saved to json file hf-mf-YOUR_UID-dump.json
[=] autopwn execution time: 52 seconds
```

## Restore data to new key

Remove the original key from the proxmark3 and place the blank card on the hf antenna (lower deck, opposite from lights)

### Determine new key's chipset

```bash
hf search
```

In the result of this command, look for `[+] Magic capabilities : Gen 1a`

If no magic capabilities are found, the UID may be read-only.

Also, make sure to confirm `Possible types` contains `MIFARE Clasic 1K`.

### Wipe, restore dump and set UID/ATQA/SAK

> Magic Gen 2 cards do not require the `c` prefix for commands, for example `hf mf cwipe` becomes `hf mf wipe`.

Wipe the card (MAKE SURE YOU HAVE THE NEW BLANK CARD ON THE PROXMARK)

```bash
hf mf cwipe
```

```log
[usb] pm3 --> hf mf cwipe
 🕒 wipe block 63
[+] Card wiped successfully
```

Load dump from original card into new card

```bash
hf mf cload --file hf-mf-YOUR_UID-dump.bin
```

```log
[usb] pm3 --> hf mf cload -f hf-mf-YOUR_UID-dump.bin
[+] loaded 1024 bytes from binary file hf-mf-YOUR_UID-dump.bin
[=] Copying to magic gen1a card
[=] .................................................................

[+] Card loaded 64 blocks from file
[=] Done!
```

Set UID, ATQA, and SAK, using the values obtained from `hf search`. These values should be 4, 2, and 1 byte(s) long, respectively.

```bash
hf mf csetuid --uid YOUR_UID --atqa YOUR_ATQA --sak YOUR_SAK
```

```log
[usb] pm3 --> hf mf csetuid --uid YOUR_UID --atqa YOUR_ATQA --sak YOUR_SAK
[+] old block 0... <redacted (16 bytes)>           
[+] new block 0... <redacted (16 bytes)>           
[+] Old UID... <redacted (4 bytes)> 
[+] New UID... <redacted (4 bytes)>   ( verified )
```

### Verify new card is identical to original

Run `hf 14a info` for both cards and confirm that (most) fields match. The least-secure of access control readers only verify the card's dumped data. However, more secure systems may also verify UID, ATQA, and SAK. The most secure systems even can detect the chipset (and if you are using a magic card). If the reader does this, you are probably SOL.
