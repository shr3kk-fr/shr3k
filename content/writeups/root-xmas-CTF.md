---
date: '2025-12-10T10:19:30+01:00'
draft: false
title: 'PKZIP vulnerabilty | Root Me xmas CTF 2025'
tags: ["misc", "stegano"]
keywords: ["misc", "steagno"]
showtoc: true
UseHugoToc: true
tocopen: true 
ShowReadingTime: true
ShowWordCount: true

---

Author : Evix

## Context

A mysterious ZIP archive has slipped down the chimney, straight from Santa’s computer. You would like to take a glimpse at the files inside, in case they look... elf-incriminating.

Can you crack the archive and uncover the secret Santa hoped to keep under wraps ?

## Zip analysis

The zip file is protected with a password : 
```
shr3k@shr3k:~$ unzip santa-secret-memes.zip 
Archive:  santa-secret-memes.zip
[santa-secret-memes.zip] dark_style.jpg password:
```

Let's take a look at our ```santa-secret-memes.zip``` using ```zipinfo```
```
shr3k@shr3k:~$ zipinfo -Z santa-secret-memes.zip 

Archive:  santa-secret-memes.zip
Zip file size: 605772 bytes, number of entries: 7
-rw-r--r--  3.0 unx   103341 BX defN 25-Dec-09 02:05 dark_style.jpg
-rw-r--r--  3.0 unx   124973 BX defN 25-Dec-09 02:05 green_bench.jpg
-rw-r--r--  3.0 unx    98878 BX defN 25-Dec-09 02:05 just_a_dream.jpg
-rw-r--r--  3.0 unx    85890 BX defN 25-Dec-09 02:05 mod_meme.jpg
-rw-r--r--  3.0 unx     1221 BX stor 25-Dec-09 02:05 portrait.jpg
-rw-r--r--  3.0 unx    81268 BX defN 25-Dec-09 02:05 raccoon.jpg
-rw-r--r--  3.0 unx   109829 BX defN 25-Dec-09 02:05 rev_meme.jpg
7 files, 605400 bytes uncompressed, 604474 bytes compressed:  0.2%

```
Take a look at `stor`, this means no compression was applied to the 'portrait.jpg' file.

With this information we can perform a **Known Plaintext Attack** on the PKZIP stream cipher.

To perform this attack we need to guess the first 12 bytes of the corresponding plaintext.

The most common JPEG 12 bytes are ```FF D8 FF E0 00 10 4A 46 49 46 00 01```

## PKA attack using bkcrack

Just write this out in a file ```jpg_header.bin```

```
shr3k@shr3k:~$ echo -ne '\xFF\xD8\xFF\xE0\x00\x10\x4A\x46\x49\x46\x00\x01' > jpg_header.bin
```
Then we can use [bkcrack](https://github.com/kimci86/bkcrack) recover the internal keys

```
shr3k@shr3k:~/bkcrack/install$ ./bkcrack -C santa-secret-memes.zip -c portrait.jpg -p jpg_header.bin 
bkcrack 1.8.1 - 2025-10-25
[11:47:25] Z reduction using 5 bytes of known plaintext
100.0 % (5 / 5)
[11:47:25] Attack on 1127172 Z values at index 6
Keys: 4c0a34dd 9f68579b 9fd87f2f
11.5 % (129829 / 1127172)
Found a solution. Stopping.
You may resume the attack with the option: --continue-attack 129829
[11:49:47] Keys
4c0a34dd 9f68579b 9fd87f2f
```

After 2 minutes, we finally recover the keys ```4c0a34dd 9f68579b 9fd87f2f```.

We can use them to recover all the archives files

```
shr3k@shr3k:~/bkcrack/install$ ./bkcrack -C santa-secret-memes.zip -c dark_style.jpg -k aca16b21 8fb459a8 89c3e395 -d dark_style_raw.jpg
```
## Recover the images
Then we can use ```deflate.py``` in the ```tools``` folder to decompress the image, remember the other files were compressed !

```
shr3k@shr3k:~/bkcrack/install$ python3 tools/inflate.py < dark_style_raw.deflate > dark_style.jpg
```
We get this nice meme

{{< figure
    src="/image/pkzip.jpg"
    width="200px"
    class="right"
>}}

## Stegano

After recovering all the images, there is no apparent flag.

I decide to run an ```exiftool``` to see if there's any hidden information in the images.

After testing a bunch we get :

```
shr3k@shr3k:~$ exiftool dream.jpg
ExifTool Version Number         : 12.76
File Name                       : dream.jpg
Directory                       : .
File Size                       : 99 kB
File Modification Date/Time     : 2025:12:09 18:10:31+01:00
File Access Date/Time           : 2025:12:09 18:10:35+01:00
File Inode Change Date/Time     : 2025:12:09 18:10:31+01:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Exif Byte Order                 : Big-endian (Motorola, MM)
Image Description               : b64(passphrase)=bWFnaWNfa2V5
X Resolution                    : 1
Y Resolution                    : 1
Resolution Unit                 : None
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : Well you find a tool and a key, time to find the 
good image 🥸
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Comment                         : tool: steghide | passphrase=magic_key
Image Width                     : 457
Image Height                    : 446
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 457x446
Megapixels                      : 0.204
```

Check this out, in the user comment we find a tool and a key.

```
shr3k@shr3k:~$ steghide extract -sf gn.jpg -p magic_key
Ecriture des donnees extraites dans "flag.txt".

shr3k@shr3k:~$ cat flag.txt 
RM{s4nt4_l0v3s_st3g4n0}
```
Happy new year !
