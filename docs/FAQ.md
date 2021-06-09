# Questions
- [What are the advantages of paper backups?](#what-are-the-advantages-of-paper-backups)
- [How much data does this back up per page / why don't you back up more data per page?](#how-much-data-does-this-back-up-per-page)
- [How do I back up more data per page?](#how-do-i-back-up-more-data-per-page)
- [How much of my backup can I lose and still restore?](#how-much-of-my-backup-can-i-lose-and-still-restore)
- [Do you support Windows / why don't you support Windows?](#why-dont-you-support-windows)
- [Do you support mac/OS X?](#why-dont-you-support-macos-x)
- [Why doesn't the restore process require qr-backup?](#why-doesnt-the-restore-process-require-qr-backup)
- [How exactly does the backup/restore process work?](#how-exactly-does-the-backuprestore-process-work)
- [Should I encrypt (password-protect) my backups?](#should-i-encrypt-password-protect-my-backups)
- [How can I protect my paper backup?](#how-can-i-protect-my-paper-backup)
- [What are the design goals of qr-backup?](#what-are-the-design-goals-of-qr-backup)
- [How do I find the maximum dimensions of my printer?](#how-do-i-find-the-maximum-dimensions-of-my-printer)
- [When I print a page, part of it is cut off](#when-i-print-a-page-part-of-it-is-cut-off)
- [When I print the backup, the last page is rotated](#when-i-print-the-backup-the-last-page-is-rotated)
- [How do I back up multiple files?](#how-do-i-back-up-multiple-files)
- [How does qr-backup compare to OllyDbg's Paperback?](#how-does-qr-backup-compare-to-ollydbgs-paperback)
- [What other paper backup projects exist?](#what-other-paper-backup-projects-exist)

# Answers

## What are the advantages of paper backups?
- It's easy to think about physical stuff. Everyone can understand whether they still have a backup (by looking), whether it's damaged (by looking), and who can access their backup.
- Paper can't be hacked. It's easy to think about who can access a paper backup compared to an online computer. Paper backups are a popular option to store GPG keys, SSH keys, crypto wallets, or encrypted messages for this reason.
- It's fun. A lot of people make paper backups for the novelty factor.
- Paper lasts a long time. CDs and flash-based storage (USB drives, SD cards, and many modern hard drives) usually stop working within 10 years. Magnetic storage works for a fairly long time unless it is damaged.
- Paper has no parts that can break. It's common for hard drives to break, and for the data inside to become unreadable, even though the data is still okay.
- Damage is visible. Sometimes a flash drive can be silently corrupted, or a drive's parts will break, but it looks OK. You can look at paper and whether it's damaged, and how much damage there is.

## How much data does this back up per page?
qr-backup on default settings (but with compression disabled) backs up about 3KB data/page. This is about the same as written text in a small font--maybe not as good. Compression improves that to 15KB english text/page for my test data.

I picked these settings by experimenting with the restore process on my computer. If I use a higher data rate, zbarcam can't consistently recognize the QR codes using my laptop webcam.  That said, you're welcome to see if your computer can [handle more](#how-do-i-back-up-more-data-per-page).

## How do I back up more data per page?
Sure, maybe you have a better webcam/scanner than I do. Past that, you can just shove more data in, but there will be costs to doing so (you'll lose other benefits).

Before changing the QR size and scale, test your restore! Looking OK to your eyes is not enough. I tried making the default settings more aggressive, but I actually can't scan smaller QR codes on my laptop webcam.

- Print double-sided
- Print smaller. Reduce the scale with `--scale <scale>` (default 8, min 1).
- Use higher-data QR codes with `--qr-version <version>` (default 10, max 40). Bigger codes doesn't always mean more data, because bigger codes don't always fit on the page. Pass `-v` to see how many KB/page you're getting. 
- Reduce error correction using `--error-correction L`. This makes your backup more sensitive to things like paper folds and dirt.
- [Maximize](#how-do-i-find-the-maximum-dimensions-of-my-printer) your page size
- Test and restore using a high-quality scanner, not a webcam.
- Use a [different paper backup program](#what-other-paper-backup-projects-exist). Ultimately, this is designed to make restores easy, not to pack data in as densely as posible.

The absolute max qr-backup allows is about 200KB/page at `--scale 1`, but you'll never be able to read something that small in practice.

## How much of my backup can I lose and still restore?
Depends if you're using qr-backup for restore.

**If you're using the Linux command-line**: If you lose one page, or even one QR code (like if it's torn off or you spill grape juice), you're hosed. You won't be able to restore. If some dirt, a pen mark, etc gets on a QR code, you'll be fine.

**If you're using qr-backup to restore**: You can lose up to 30% of the QR codes and still restore. 

There are some command-line options that reduce the damage:
- `--num-copies` prints duplicates of QR codes. If you're printing duplicates, I recommend three copies (rather arbitrarily).
- `--no-compress` disables compression. This makes the backup longer, but it means that if you have 50% of the data, you can recover 50% of the file. For some backup types (text documents) this is useful. For others (bitcoin wallets) it is not. Make sure your document doesn't contain weird characters (including "\r", the mac/windows newline), or base64 encoding will turn on, which makes recovery harder.
- There is an open [feature request](https://github.com/za3k/qr-backup/issues/2) to improve this and let you lose some QR codes.

## Why don't you support windows?
I might someday, I just haven't done it yet.

- I don't use Windows myself
- I want the restore process to work WITHOUT qr-backup software. I'm not sure how to do this on Windws yet.

In the meantime, you could try [a different paper backup program](#what-other-paper-backup-projects-exist).

## Why don't you support mac/OS X?
Both backup and restore probably work, actually, it's just not tested. `brew install zbar` and let me know in the issue tracker.

## Why doesn't the restore process require qr-backup?
Because I want the restore process to work when qr-backup has been lost to history. Also, I want users to understand how the backup/restore process works.

That said, there's no good Linux command-line tool to do erasure decoding, which is why that feature (only) needs qr-backup.

## How exactly does the backup/restore process work?
The exact commands to run are described in the README and on the printed backup. But here's a conceptual explanation of how things work.

The backup process:
- If compression is on, data is compressed using gzip
- The data is now preprocessed.
- The data is split into small chunks, about 2K each with the default settings. If there are 50 chunks, labels N01 thru N50 is put at the start of each chunk.
- Extra chunks (E01-E21) are generated by erasure coding (reed-solomon). This is advanced math magic.
- Base64 encoding is applied to each chunk.
- Each chunk is printed as a QR code on the paper, and labeled with the code number.

The restore process is
- First, the user scans each QR code (in any order). Since each code contains the code number (N01-N50), the computer sorts everything out, making sure each code 01-50 is present exactly once.
- The codes are put in order (and duplicates removed). The 01-50 labels are removed.
- Each chunk its base64-decoded
- The preprocessed data is restored
    - If all the chunks ARE ALL present, the chunks are just appended together.
    - If the chunks ARE NOT ALL present, erasure coding is applied to restore missing chunks. This is advanced math magic.
- If the data was compressed, it's now decompressed
- The file is now restored.
- The file is checksummed using sha256, which verifies the file is perfectly restored.

## Should I encrypt (password-protect) my backups?
That's up to you. 

I don't, because I think it's likely that I'll forget my password in 5-10 years. But, I'm not backing up my bitcoin wallet either.

There is an open [feature request](https://github.com/za3k/qr-backup/issues/4) to add this to qr-backup directly. Until then, if you want to password-protect your backup, you'll need to do it yourself. I'd use `gpg --symmetric` to encrypt and `gpg --decrypt` to decrypt (because gpg is widely available). 

If you do need to encrypt your backup, remember that you can write your password down (somewhere different!) on physical paper. Preferably several places--you need to remember where it's written. 

The especially geeky can also look into Shamir's secret-sharing scheme, which can let you need any 3 out of 4 pieces of information to restore. Remember to test your restore as usual.

## How can I protect my paper backup?
Test that your restore procedure works. Seriously, do that first.

Then (in order): 
- The most common failure mode for paper backups is to *forget* about your backup or throw it out accidentally.
    - On each copy of the backup, document what it is. You can hand-write or attach a cover sheet.
        - Who you are, and several forms of contact information (phone, address, email, contact info of family, friends, or your place of work)
        - Why it shouldn't be thrown out (or when it should be).
        - What exactly this backup is (what's in the file, but also that this is a physical backup of computer data)
        - Where any other copies are, in case this one is damaged
    - If people other than you should know about the backup, tell them. If anyone would throw this away, tell them not to.
    - Wherever you normally keep your reminders, document that you have a backup, what exactly of, and where it is.
- Make several copies in different buildings/cities. More copies is simply better than one well-protected copy.
- Protect against folding and losing pages. A box or envelope may help. Folding on a QR code can make it unreadable, and losing a page means you lose your data.
- If you're backing up something like pictures or text documents, print them and attach them to the qr-backup paper backup. That way you have the data even if the restore process somehow fails.
- Protect against water damage.
- Use acid-free paper. I don't imagine inkjet vs laser printer is that important, but I'm not an expert.
- Protect against fire damage. The best way to protect against fire damage to paper is to have a copy in another building.

## What are the design goals of qr-backup?
Okay, you caught me, no one has asked this, it's not really an FAQ.

- It should be very easy to restore the backup
- It should actually work on my actual computer, on default settings
- It should actually work with low-quality hardware (ex bad black-and-white printer and bad webcam)
- Restore should not require qr-backup, or any other unusual software. (Unfortunately there is no installed-by-default qr reader on Linux, but zbar is the only requirement)
- An average human being should be able to follow the restore directions (this is not really true currently, but an average command-line Linux user can)
- The output should include good documentation

## How do I find the maximum dimensions of my printer?
If you really want to squeeze things in, you need to know how large you can print on a page. There are two options to figure out the max print size.

- Recommended: Experiment (using `--page`)
- Calculate it from CUPS PPD files.

If you want to figure it out from CUPS, here's what I did:
1. On CUPS, check out the PPD file for your printer. The Debian wiki has useful information about PPDs.
2. Use the ImageableArea for the paper size you want. Subtract the two pairs of numbers--this is the usable size of the page (in 'points', or 1/72 of an inch, the same unit `--page` uses).

You can also mess with `--dpi` but it's unlikely to be your limiting factor.

## When I print a page, part of it is cut off
You may need to adjust the dimensions of your printer (or paper size, are you using A4 instead of US letter?).

If you adjust your page dimensions to be smaller, and it works... but the QR codes are suddenly misaligned from your failing print by a small amount, you've hit a printing bug with full-size pages. You need to upgrade Ghostscript to 9.50 or later.

I believe there is a remaining [issue](https://github.com/OpenPrinting/cups-filters/issues/373) in the new version, unfortunately.

## When I print the backup, the last page is rotated
Pass CUPS the option 'nopdfAutoRotate'.

## How do I back up multiple files?
1. You can run `qr-backup` multiple times, and print each PDF. If you lose any of the QR codes, you won't be able to restore that file.
2. OR, you can tar/zip the files, and back up the tar/zip. If you lose any of the QR codes, you won't be able to restore **any** of the files.

## How does qr-backup compare to OllyDbg's Paperback?
First, here's what Paperback/Paperbak is:
- [Description](https://ollydbg.de/Paperbak/)
- [Original Code](https://github.com/timwaters/paperback )
- [Attempted Linux fork](https://github.com/cyphar/paperback)

Here's how they are similar/different
- Both have the same essential goal and flow: back some stuff up to paper and restore it later.
- Paperbak is Windows-only; qr-backup runs on Linux CLI and probably mac CLI.
- Paperbak is focused around shoving the most data on paper possible (with some nice extras). qr-backup is focused on easy restore that actually works (with some nice extras).
- I'm not super clear if Paperback actually/still works end to end (haven't tested it firsthand, because no Windows). I'll check if I can get cyphar to work on Linux. It would definitely be a good second tool, I'd probably use both.
- Paperbak is designed to want a high-quality scanner (3x print dpi). qr-backup can use a webcam, sucky scanner, or whatever else that can read QR codes with a little hacking.
- At a best estimate, default settings are 300X more data per page on Paperbak. Even at more aggressive qr-backup settings, I bet that's 10-30X. Part of this is QR--most of it is needing a really good scanner (aside from quality, webcams have focus length and stability issues, and I'm not sure zbar is that great).
- Paperbak uses a proprietary format, and needs Paperbak to restore. qr-backup uses an esoteric mix of existing formats like QR and gzip, and can be restored with a bash oneliner of standard linux tools.
- Paperbak uses reed-solomon coding, so you can lose part of the page(s) and still restore. This isn't implemented in qr-backup yet.
- Both support compression.
- Paperbak offers encryption. This isn't implemented in qr-backup yet.
- qr-backup prints a bunch of human-readable info on the page explaining what it is and how to restore. Paperbak optionally prints a little of this (mostly the file name, size, and date)
- Both are black-and-white only
- qr-backup is designed to someday work as an easy app on phones, because it's based around digital cameras instead of scanners. It wouldn't be possible or useful to do this with paperbak.
- qr-backup is maintained (well, as of writing this FAQ answer, at least!)

## What other paper backup projects exist?
2D code based (like qr-backup):
- [qr-backup](https://github.com/za3k/qr-backup): This project. Based on QR codes. Focuses on easy restore using webcam and standard CLI tools. Low data density.
- [qrencode](https://fukuchi.org/works/qrencode/), etc: Small amount of data can be directly printed to one QR code, and restored by any QR scanner.
- [paperbackup](https://github.com/intra2net/paperbackup): Remarkably similar to qr-backup, down to the code format. Based on QR codes. Focused on GPG/SSH key backup. See also the [paperkey](http://www.jabberwocky.com/software/paperkey/) preprocessor.
- [asc2qr.sh](https://github.com/4bitfocus/asc-key-to-qr-code): QR-based, less polished.

Dense pixel grid (like Paperbak). Everything in this section needs a good scanner:
- Paperback [explanation](https://ollydbg.de/Paperbak/) and [code](https://github.com/Rupan/paperbak/) by OlyDbg: Much denser, windows-only. Uses reed-solomon codes.
- [paperback-cli](https://git.teknik.io/scuti/paperback-cli): Cross-OS port for OlyDbg's Paperbak program.
- [ColorSafe](https://github.com/colorsafe/colorsafe): Black and white or color output. Split into sectors. Error correction is reed-solomon within a sector, none outside (as best I could find out).
- [optar](http://ronja.twibright.com/optar/): Black and white. Uses Golay codes.
