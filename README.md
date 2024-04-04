# qr-backup

## TLDR

Backup (Make one of each):
    - PDF: `qr-backup /path/to/input/file -o /path/to/output.pdf`
    - TXT: `qr-backup /path/to/input/file --human-readable-stdout > /path/to/output.txt`

Restore from scanned images: `./qr-backup --restore /path/to/your/images/*.png`

## About This Project

Generate paper backups for Linux. Currently **command-linux Linux only**.

Takes any file, and outputs a "paper backup": a printable black-and-white pdf full of QR codes. To back up your file, print the PDF. The pile of paper in your hand is now a backup of the file.

This is alpha software--I use it for my own backups, but I offer no guarantees. Test your restore when you make it, not when you need it!

## What is a paper backup?
A paper backup is a number of printed sheets of paper containing special barcodes.

If your file is lost, corrupted, deleted, etc, you can restore from your paper backup. qr-backup reads the [QR barcodes](https://en.wikipedia.org/wiki/QR_code) using your computer's webcam (or scanner) to get your file back.

## Should I back up to paper?
Possibly. You should still back it up to something more usual like a USB thumbstick *first*, because it's easier to restore and update.

Common files to back up are small important records, and small secret files. Examples include: a diary, an address book, a short story you wrote, financial records, medical records, an ssh or gpg cryptographic key, or a cryptocurrency (bitcoin) wallet.

Paper is not the best or most efficient storage method, so you can't back up big files. 10KB or 100KB files is a reasonable limit.

[Learn about the advantages](docs/FAQ.md#what-are-the-advantages-of-paper-backups) of paper backups. 

## Example Backup
![Example Backup](docs/example.png)

## System Requirements
### Backup Requirements
- **A Linux computer and knowledge of how to use the command line**
- A printer
- python 3.6 or later
- python-qrcode
- python-pillow
- imagemagick
- zbar (optional, used to digitally test restore)
### Restore Requirements
- **A Linux computer and knowledge of how to use the command line**
- **The restore process works without qr-backup installed**
- A webcam or scanner
- imagemagick
- zbar

## Making a backup
1. Run qr-backup on your file. On the Linux command-line, run `qr-backup <YOUR_FILE>`
2. This generates a black-and-white PDF (`<YOUR_FILE>.qr.pdf`)
3. Print the PDF on your printer

There are many command-line options available for advanced users. For a full list, read the [USAGE](docs/USAGE.md) doc online, or run `qr-backup --help` on your computer.

## Restoring a paper backup
The restore process **does NOT require qr-backup**. It does require a command-linux Linux computer.

(Option 1): Use qr-backup, if you have it.
- Webcam option
    1. Run `qr-backup --restore`
- Scanner option
    1. Scan images using your scanner
    2. Run `qr-backup --restore IMAGES`

(Option 2): Use the linux command line, if you lose qr-backup. Commands are provided in the PDF printout. You will need to install `zbar`.

## More questions
For more questions, see the [FAQ](docs/FAQ.md).
