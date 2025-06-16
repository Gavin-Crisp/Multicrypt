# StegStore

## What

StegStore is an encryption format designed for full disk encryption inspired by
Sun Knudsen's [blockcrypt](https://github.com/superbacked/blockcrypt). It allows
for multiple plaintexts to be encoded into one block of ciphertext, each under
multiple keys, while concealing their existence from each other with no forensic
fingerprint.

## Why

To get the best of both worlds from Veracrypt and LUKS: Veracrypt is
steganographically sound but has a limited feature set compared to LUKS; LUKS is
highly customisable and allows for multiple keys to be used to access the
partition but hides very little about itself. StegStore is the combination of
the two that I couldn't find.

## How

### Format

- [1.0](format.md) (current)

### Explanation

The header section contains 32 keyslots which can be decrypted with the entered
password. Each slot contains the master key for the metadata contained in the
header, the master key for the seat the key has access to, and the position and
size of the seat's local header in the data section. The decrypted seat header
contains all necessary information to decrypt the associated plaintext.

### Example

One seat's perspective of the decrypted block

```text
        ┌────────────────────────────────────────┐
     ┌──┼────────────────────────────────────────┼─┬─────────────────────────────────────────────────────────┐
     │  │                                        │ │                                                         │
     │  │           ┌────────────────────────────┼─┼───┐ ┌────────────┐                                      │
     ↓  │           ↓                            ↓ │   │ │            ↓                                      ↓
┌────┬──────┬───┬───┬────────────────────┬───────┬─────────────┬──────┬──────────────┬─────────────────┬─────┬──────┬──┐
│    │ slot │   │   │    data cluster    │       │ seat header │      │ data cluster │                 │     │ slot │  │
└────┴──────┴───┴───┴────────────────────┴───────┴─────────────┴──────┴──────────────┴─────────────────┴─────┴──────┴──┘
 └─────────────┘ └────────────────────────────────────────────────────────────────────────────────────┘ └─────────────┘
     header                                               data                                           backup header
```
