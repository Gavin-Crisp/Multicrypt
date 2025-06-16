# StegStore On Disk Format

## Format

| Offset (bytes) | Size (KiB) | Description    |          Format           |
|:--------------:|:----------:|:---------------|:-------------------------:|
|       0        |     4      | Primary header | [Header](#header-section) |
|       4        |    Var     | Data           |   [Data](#data-section)   |
|     S - 4      |     4      | Backup header  | [Header](#header-section) |

The backup header is encrypted with different keys derived from a different salt.

### Header Section

| Offset (bytes) | Size (bytes) |  Encryption Status   | Description                                     |  Format  |
|:--------------:|:------------:|:--------------------:|:------------------------------------------------|:--------:|
|       0        |      32      |          No          | Salt. Differs between primary and backup header |  u8[32]  |
|       32       |      96      |   Yes (key I) / No   | [Key entry](#key-entry) or padding. Slot 0      |  u8[96]  |
|      ...       |     ...      |         ...          | ...                                             |   ...    |
|      3008      |      96      | Yes (key XXXII) / No | [Key entry](#key-entry) or padding. Slot 31     |  u8[96]  |
|      3104      |      4       |   Yes (header key)   | Format magic in ASCII -- "STGS"                 | char[4]  |
|      3108      |      2       |   Yes (header key)   | Format version -- 1                             |   u16    |
|      3110      |      2       |   Yes (header key)   | Minimum version required to open -- 1           |   u16    |
|      3112      |      8       |   Yes (header key)   | Size of data section in sectors                 |   u64    |
|      3120      |      8       |   Yes (header key)   | Size of sector in bytes                         |   u64    |
|      3128      |      16      |   Yes (header key)   | UUID                                            |  u8[16]  |
|      3144      |      64      |   Yes (header key)   | ASCII label                                     | char[64] |
|      3208      |     872      |   Yes (header key)   | Reserved                                        | u8[872]  |
|      4080      |      16      |          No          | Auth tag for header                             |  u8[16]  |

If a key entry is referenced by a seat header it is owned by that seat; All unowned keyslots are random padding

#### Key Entry

| Offset (bytes) | Size (bytes) | Encryption Status | Description           |                 Format                  |
|:--------------:|:------------:|:-----------------:|:----------------------|:---------------------------------------:|
|       0        |      32      |        Yes        | Header key            |                 u8[32]                  |
|       32       |      32      |        Yes        | Seat master key       |                 u8[32]                  |
|       64       |      16      |        Yes        | Seat header reference | [Cluster reference](#cluster-reference) |
|       80       |      16      |        No         | Auth tag              |                 u8[16]                  |

### Data Section

If a sector is referenced by a seat header it is owned by that seat. All owned sectors are encrypted with aes-gcm, all
unowned sectors are random padding.

#### Seat Header

| Offset (bytes) | Size (bytes) | Encryption Status | Description                          |                  Format                   |
|:--------------:|:------------:|:-----------------:|:-------------------------------------|:-----------------------------------------:|
|       0        |      16      |        Yes        | UUID                                 |                  u8[16]                   |
|       16       |      64      |        Yes        | ASCII label                          |                 char[64]                  |
|       80       |      4       |        Yes        | Owned keyslots - bit 0: slot 0; etc. |                    u32                    |
|       84       |      4       |        Yes        | Bitflags                             |           [Bitflags](#bitflags)           |
|       88       |      40      |        Yes        | Reserved                             |                  u8[40]                   |
|      128       |     Var      |        Yes        | Data clusters table                  | [Cluster reference](#cluster-reference)[] |
|     S - 16     |      16      |        No         | Auth tag                             |                  u8[16]                   |

A single seat header that spans multiple sectors will only have one auth tag at the end of its last sector. The IV of
the seat header is 0.

##### Bitflags

| Offset (bits) | Size (bits) | Description                                                                                 |
|:-------------:|:-----------:|:--------------------------------------------------------------------------------------------|
|       0       |      1      | Opened UUID - 0: Disk UUID; 1: Seat UUID                                                    |
|       1       |      2      | Opened Label - 0: No label; 1: Disk label; 2: Seat label; 3: Hyphenated disk and seat label |
|       3       |     29      | Reserved                                                                                    |

#### Data Sector

Each data sector is encrypted independently using an ESSIV based on the sector's logical index

### Cluster reference

| Offset (bytes) | Size (bytes) | Description                                      | Format |
|:--------------:|:------------:|:-------------------------------------------------|:------:|
|       0        |      8       | Offset from beginning of data section in sectors |  u64   |
|       8        |      8       | Size of cluster in sectors                       |  u64   |

Cluster references with a size of 0 are null
