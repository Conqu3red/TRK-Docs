# General Structure
The structure of the TRK File can be broken down as seen below:
* Header
* Features
* SongInfo (Optional)
* Line Data
* Metadata (Optional)

Each section is described in more detail below, along with other references for important structures.


# Header

* **0x00:** Magic Number 54 52 4B F2 Spelling out TRKò
* **0x04:** Byte indicating version (should have value 0x1)
* **0x05:** Signed 16 bit integer denoting the length of the feature string
* **0x07:** Feature string: list of features seperated by semicolons (see above for length)

# Features

* `REDMULTIPLIER` - Red lines have multipliers
* `SCENERYWIDTH` - Width values for scenery lines
* `6.1` - Version 6.1 physics (default 6.2)
* `SONGINFO` - Track contains song metadata
* `IGNORABLE_TRIGGER`
* `ZEROSTART`

# C# Encoded String
This is a reference for the C# binary standard of encoding string (reference: [here](https://docs.microsoft.com/en-us/openspecs/sharepoint_protocols/ms-spptc/89cf867b-260d-4fb2-ba04-46d8f5705555))

* **0x00:** Length of the UTF-8 string: 7BitEncodedInt (T)
    * reference: [here](https://docs.microsoft.com/en-us/openspecs/sharepoint_protocols/ms-spptc/1eeaf7cc-f60b-4144-aa12-4eb9f6e748d1)

    * The value is written out 7 bits at a time starting with the least significant bits. If the value will not fit in 7 bits the high bit of the byte is set to indicate there is another byte of data to be written. The value is then shifted 7 bits to the right and the next byte is written. If the value will fit in the seven bits the high byte is not set and it signals the end of the structure.

* **From the end of T**: UTF-8 string, matching the length above.

# Song Info
Song info is only present if the `SONGINFO` feature is detected.
* **0x00:** Song: C# Encoded String
    * Contains Song name and song offset (as a float converted to a string) seperated by `\r\n`

# Line Data
* **0x00:** Rider start position X, Double precision float (8 Bytes)
* **0x08:** Rider start position Y, Double precision float (8 Bytes)
* **0x016:** 32 bit unsigned integer (N) denoting the number of lines to read

For N Times:
* **0x00:** Line Type + Flags
    * **Bit 1** Line inverted, Boolean
    * **Bit 2 and 3** Line Extension:
        * `0` None
        * `1` Left
        * `2` Right
        * `3` Both
    * **Bits 3 - 8** Line Type:
        * `0` Scenery
        * `1` Blue (Standard)
        * `2` Acceleration
* Type Specific Data (S):
    * Red Line (R)
        * **0x01:** Red Line Multiplier, 1 byte. Only present if this line is type `Acceleration` and the feature `REDMULTIPLIER` is present.

    * Blue or Red Line
        * If feature `IGNORABLE_TRIGGER` is present (T)
            * **R + 1** Zoom trigger, Boolean
            * If Zoom trigger is true:
            * **R + 2:** Target, Single precision float (4 Bytes)
            * **R + 7:** Frames, 16-Bit Signed Integer
        * **From end of T:** Line ID: Signed 32-Bit Integer
        * If extension is not `None`
            * **From end of T + 4:** 32-Bit Signed Integer (Ignored)
            * **From end of T + 8:** 32-Bit Signed Integer (Ignored)

    * Scenery Line
        * If `SCENERYWIDTH` feature is present (W)
            * **0x01:** Line Width, 1 Byte. (Divide by 10.0 to get width value)

* **S + 4:** X position of the start, Double precision float (8 Bytes)
* **S + 12:** Y position of the start, Double precision float (8 Bytes)
* **S + 18** X position of the end, Double precision float (8 Bytes)
* **S + 26:** Y position of the end, Double precision float (8 Bytes)

# Metadata