# EQOA-Frontiers-ISO-Patch
This code is intended to patch the DNAS from the EQOA Frontiers ISO, assuming it is unmodified. This code will also patch the ilinkid check to be 0'd out ensuring decryption can happen with specific memory card's

How does this work?
 Download the relevant executable from releases for your PC type.
 Click and drag your EQOA Frontiers iso over it
 If any issues or errors occur, the program will let you know
   This checks the iso sha + crc of the elf to ensure it hasn't been modified, if it has, it will fail
It will spit out the modified ISO that will...
  Patch the DNAS skip directly to disc
  0 out the ILink ID check for decryption of files
    This is beneficial in certain scenario's where the ilink of an ID console was 0, and allow's the user to decrypt the game files

What can I use this on?
 PCSX2
 PS2 - IlinkID check overwrite doesn't work due to McFreeBoot hi-jacking OPL to date.
