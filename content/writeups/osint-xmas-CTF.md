---
date: '2025-12-11T16:14:57+01:00'
draft: false
title: 'Santa osint | Root Me xmas CTF 2025'
tags: ["osint"]
keywords: ["osint"]
ShowReadingTime: true
ShowWordCount: true
---
Author: Herethicc

## Introduction

{{< figure src="/image/dog-work.gif" >}}

Oh no, it looks like the Grinch’s assistant pwned us ! Luckily, our experts managed to recover two key informations:


> - A file hash:
bf6a54d181b9bbcb168720b9d466ca27d14ead283220ee5a9e777352c16edca3
> - A leaked message that said more intels can be found somewhere in a vault.


Lets check the file hash on virus total :

> bf6a54d181b9bbcb168720b9d466ca27d14ead283220ee5a9e777352c16edca3
sleighctl.py 

We get a file name **sleightctl.py**

We copy paste in github and we found this repo : https://github.com/jbashdapuppet/santa-root-kit

```
~/santa-root-kit$ tree
.
├── Disclaimer
├── etc
│   └── santa.conf
├── lib
│   ├── elf_parser.py
│   └── reindeer_hooks.py
├── README.md
└── sleighctl.py
```

After checking the commits we found a nice fake flag :


```
~/santa-root-kit$ git log -p

commit cff1a5dcff4a348824956daf4541532bb4d73048 (HEAD -> main, origin/main, origin/HEAD)
Author: jbashdapuppet jbash@tutamail.com
Date:   Wed Dec 10 22:16:37 2025 +0100

    Update santa.conf

diff --git a/etc/santa.conf b/etc/santa.conf
index 3ad6880..0b9a91b 100644
--- a/etc/santa.conf
+++ b/etc/santa.conf
@@ -1,5 +1,5 @@
 {
"magic_token": "RM{D1D_Y0U_r3411Y_7H1N6_17_W0U1D_83_7H47_345Y}",
+    "magic_token": "NPM{30L133N3}",
     "enabled_hooks": [
         "rudolph_syscall_hide",
         "dasher_filecloak"
     ]
}
```

And we found another intersting comments and the end of the **sleighctl.py** file :

```
+#TODO: Optimize the code / Add comments / Replace our PGP key on "that website"
```

We can guess that there is a PGP key in a certain website, after some research i found this website where there is the same username https://keybase.io/jbashdapuppet/

Bingo !

Look at what we found on this website:
```
jbashdapuppet's public key
fingerprint:	3E5D3EC729A1814055DA68A22A2E08081E78AB3D
64-bit: 	2A2E08081E78AB3D
curl/raw: 	this key | all their PGP keys

# curl + gpg pro tip: import jbashdapuppet's keys
curl https://keybase.io/jbashdapuppet/pgp_keys.asc | gpg --import

# the Keybase app can push to gpg keychain, too
keybase pgp pull jbashdapuppet
```
Now just use the first command on the website and we are done !
```
curl https://keybase.io/jbashdapuppet/pgp_keys.asc | gpg --import

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3277  100  3277    0     0   9719      0 --:--:-- --:--:-- --:--:--  9724
gpg: clef 2A2E08081E78AB3D : clef publique « jasonbashdapuppet (Temporary key, replace with the one located at https://drive.proton.me/urls/MEK165VH8R#PhiRcVi8lCw1) jbash@tutamail.com » importée
gpg: Quantité totale traitée : 1
gpg:               importées : 1
```
We get a nice proton drive link with a flag.txt file inside:

RM{H0_H0_H0_1_M16H7_83_1N_7r0U813}

