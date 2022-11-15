# HID Prox

HID Prox keys are notoriously easy to duplicate. 

Probability of success: ridiculously high

Time to complete: <10 seconds

## Identify chipset

Most original keyfobs have "HID Prox" somewhere on them. You can confirm this and the spec with 
```bash
lf search
```

```log
[usb] pm3 --> lf search

[=] NOTE: some demods output possible binary
[=] if it finds something that looks like a tag
[=] False Positives ARE possible
[=] 
[=] Checking for known tags...
[=] 
hid preamble detected
[+] [H10301  ] HID H10301 26-bit                FC: **  CN: *****  parity ( ok )
[+] [ind26   ] Indala 26-bit                    FC: ***  CN: ****  parity ( ok )
[=] found 2 matching formats
[+] DemodBuffer:
[+] ************************

[=] raw: 00000000000000**********

[+] Valid HID Prox ID found!
```

> Sensitive data has been replaced with `*`

If you receive a raw value, congrats! you have all the information needed to clone the card. You can also run `lf hid reader`, but it should return the same result.

## Write data to new card

Take the original key off the proxmark and place the new blank key on the lf antenna (higher deck, copper coil).

Use the `raw` value from the previous step (without any leading zeros) as the raw bytes to write in this step.

```bash
lf hid clone -r YOUR_RAW_KEY
```

```log
[usb] pm3 --> lf hid clone -r YOUR_RAW_KEY
[=] Preparing to clone HID tag using raw YOUR_RAW_KEY
[=] Done
[?] Hint: try `lf hid reader` to verify
```

When writing to a lf hid card, the proxmark behaves the same in the clone step whether successfully writing to a card, to thin air, or anything in between. So now confirm the value was properly written.

```bash 
lf hid reader
```

Confirm that this result has the same raw, DemodBuffer, FC, CN, and passes parity checks.

Congratulations! You can now repeat this single command (or two to be safe) on many more cards.