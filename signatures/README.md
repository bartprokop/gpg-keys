# Signing with GPG

## Code signing Master key

### Adopted solution

- I'm using multiple machines to develop the code, thus I'm committing from multiple machines.
- I prefer not to share single signing key across multiple machines. If one machine gets compromised, all my work will be compromised (as long as GPG signature is considered).
- I will generate a single code signing sub-key per machine.
- The master key will be only for `certify` purpose and will be absent from daily machine.
- The master key will be initialised only on air-gapped machine for the purpose of generating signing sub-keys.
- No other subkeys (auth or enc) will be in the keyring.
- Exported public keyring will be uploaded for sites like GitHub.

### Generation of signing master key

In this step we will generate ECC key that will be used ONLY to certify my code signing sub-keys.

```zsh
➜  ~ gpg --full-gen-key --expert
gpg (GnuPG) 2.4.5; Copyright (C) 2024 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
  (14) Existing key from card
Your selection? 11

Possible actions for this ECC key: Sign Certify Authenticate
Current allowed actions: Sign Certify

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? s

Possible actions for this ECC key: Sign Certify Authenticate
Current allowed actions: Certify

   (S) Toggle the sign capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? q
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (2) Curve 448
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Bart Prokop
Email address: bart@******.dev
Comment: code signing
You selected this USER-ID:
    "Bart Prokop (code signing) <bart@prokop.***>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: directory '/home/bart/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/bart/.gnupg/openpgp-revocs.d/4CC5B7436206D43D89BC8A22CE4B83F76925A7FA.rev'
public and secret key created and signed.

pub   ed25519 2024-09-13 [C]
      4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
uid                      Bart Prokop (code signing) <bart@pro***.***>
```

### Distribute Master Key

Sending masterkey to key-server might not be the worst idea:

```
➜  ~ gpg --send-keys 4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
gpg: sending key CE4B83F76925A7FA to hkps://keyserver.ubuntu.com
```

Or even exporting public in plain text to share explicitly:

```bash
➜  ~ gpg --export --armor 4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEZuQYkhYJKwYBBAHaRw8BAQdAX/PoDTBC28stWqlj5kr9i0AlQqUIGK6KFiWf
F14RTpi0LEJhcnQgUHJva29wIChjb2RlIHNpZ25pbmcpIDxiYXJ0QHByb2tvcC5k
ZXY+iJMEExYKADsWIQRMxbdDYgbUPYm8iiLOS4P3aSWn+gUCZuQYkgIbAQULCQgH
AgIiAgYVCgkICwIEFgIDAQIeBwIXgAAKCRDOS4P3aSWn+pbvAP9HBJNtt49/NYoN
Q4fNJPe9Nj8Un3BvggBi5MlAHzmypQD/ey4kw9GLFJpuAamet5P2ap+dgJZKxO0k
Ks3jGR6CLAc=
=SZb2
-----END PGP PUBLIC KEY BLOCK-----
```

### Back-up of the master signing key

We should create backup of private master key (and keep it offline):

```
$ gpg --armour --export-options export-backup --export-secret-keys 4CC5B7436206D43D89BC8A22CE4B83F76925A7FA | \
  gpg --armour --cipher-algo AES256 --output /tmp/4CC5B7436206D43D89BC8A22CE4B83F76925A7FA.asc --symmetric

 ┌─────────────────────────────────────────────────────────────┐
 │ Enter passphrase                                            │
 │                                                             │
 │                                                             │
 │ Passphrases match.                                          │
 │                                                             │
 │ Passphrase: ******************************_________________ │
 │                                                             │
 │ Repeat: ******************************_____________________ │
 │ ┌─────────────────────────────────────────────────────────┐ │
 │ │                          100%                           │ │
 │ └─────────────────────────────────────────────────────────┘ │
 │        <OK>                                   <Cancel>      │
 └─────────────────────────────────────────────────────────────┘

 ┌───────────────────────────────────────────────────────────────┐
 │ Please enter the passphrase to export the OpenPGP secret key: │
 │ "Bart Prokop (code signing) <bart@prokop.dev>"                │
 │ 255-bit EDDSA key, ID CE4B83F76925A7FA,                       │
 │ created 2024-09-13.                                           │
 │                                                               │
 │                                                               │
 │ Passphrase: **********************___________________________ │
 │                                                               │
 │         <OK>                                   <Cancel>       │
 └───────────────────────────────────────────────────────────────┘
```

Store securely (ideally offline) the file from /tmp folder.

### Create signing subkey

If some option are not available, use `--expert` switch.

```bash
$ gpg --edit-key 4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
gpg (GnuPG) 2.4.5; Copyright (C) 2024 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  ed25519/CE4B83F76925A7FA
     created: 2024-09-13  expires: never       usage: C
     trust: ultimate      validity: ultimate
[ultimate] (1). Bart Prokop (code signing) <bart@prokop.dev>

gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
  (10) ECC (sign only)
  (12) ECC (encrypt only)
  (14) Existing key from card
Your selection? 10
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
Key does not expire at all
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  ed25519/CE4B83F76925A7FA
     created: 2024-09-13  expires: never       usage: C
     trust: ultimate      validity: ultimate
ssb  ed25519/3A25D90B2E35828B
     created: 2024-09-15  expires: never       usage: S
[ultimate] (1). Bart Prokop (code signing) <bart@prokop.dev>

gpg> quit
Save changes? (y/N) y
```

```bash
$ gpg -K
sec   ed25519 2024-09-13 [C]
      4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
uid           [ultimate] Bart Prokop (code signing) <bart@prokop.dev>
ssb   ed25519 2024-09-15 [S]
```

### Remove master (certify) private key

Now (after signing key is created) it is important to remove the certify secret key.

```bash
$ gpg -K --with-subkey-fingerprints
[keyboxd]
---------
sec   ed25519 2024-09-13 [C]
      4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
uid           [ultimate] Bart Prokop (code signing) <bart@prokop.dev>
ssb   ed25519 2024-09-15 [S]
      0F7CA8F21EB8A2E863B5C8723A25D90B2E35828B

gpg --delete-secret-keys 4CC5B7436206D43D89BC8A22CE4B83F76925A7FA\!
gpg (GnuPG) 2.4.5; Copyright (C) 2024 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


sec  ed25519/CE4B83F76925A7FA 2024-09-13 Bart Prokop (code signing) <bart@prokop.dev>

Note: Only the secret part of the shown primary key will be deleted.

Delete this key from the keyring? (y/N) y
This is a secret key! - really delete? (y/N) y
```

Note the exclamation mark at the end of the fingerprint – it forces the use of a specific primary or subkey so that the command does not work on the entire key.
The exclamation mark is quoted due to shell requirements. 

```bash
$ gpg -K --with-subkey-fingerprints
[keyboxd]
---------
sec#  ed25519 2024-09-13 [C]
      4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
uid           [ultimate] Bart Prokop (code signing) <bart@prokop.dev>
ssb   ed25519 2024-09-15 [S]
      0F7CA8F21EB8A2E863B5C8723A25D90B2E35828B
```

Note the `#` after the `sec`, it says that keyring no longer contains private key.
It is also no longer possible to add a subkeys:

```bash
$ gpg --edit-key 4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
gpg (GnuPG) 2.4.5; Copyright (C) 2024 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret subkeys are available.

pub  ed25519/CE4B83F76925A7FA
     created: 2024-09-13  expires: never       usage: C
     trust: ultimate      validity: ultimate
ssb  ed25519/3A25D90B2E35828B
     created: 2024-09-15  expires: never       usage: S
[ultimate] (1). Bart Prokop (code signing) <bart@prokop.dev>

gpg> addkey
Need the secret key to do this.
```

### Importing a new (public) key on other system

This can easily be obtained from key server.

```bash
$ gpg -k
pub   ed25519 2024-09-13 [C]
      4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
uid           [  full  ] Bart Prokop (code signing) <bart@prokop.dev>
sig 3        CE4B83F76925A7FA 2024-09-13  [self-signature]
sig 3  R     EA86B9CE4982B047 2024-09-13  Bart Prokop (YubiKey 5C NFC) <bart@prokop.dev>

$ gpg --recv-keys 4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
gpg: key CE4B83F76925A7FA: "Bart Prokop (code signing) <bart@prokop.dev>" 1 new signature
gpg: key CE4B83F76925A7FA: "Bart Prokop (code signing) <bart@prokop.dev>" 1 new subkey
gpg: Total number processed: 1
gpg:            new subkeys: 1
gpg:         new signatures: 1

$ gpg --list-sig 4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
pub   ed25519 2024-09-13 [C]
      4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
uid           [  full  ] Bart Prokop (code signing) <bart@prokop.dev>
sig 3        CE4B83F76925A7FA 2024-09-13  [self-signature]
sig 3  R     EA86B9CE4982B047 2024-09-13  Bart Prokop (YubiKey 5C NFC) <bart@prokop.dev>
sub   ed25519 2024-09-15 [S]
sig          CE4B83F76925A7FA 2024-09-15  [self-signature]
```

### Restoring master key

This should be done on air-gapped machine and newly created subkey exported for everyday use.

```bash
gpg --decrypt 4CC5B7436206D43D89BC8A22CE4B83F76925A7FA.asc | gpg --import
```

Then follow instructions to:
- Create signing subkey
- Remove master (certify) private key

This should be signing key setup for everyday use:

```bash
$  gpg --list-secret-keys --keyid-format=long

sec#  ed25519/CE4B83F76925A7FA 2024-09-13 [C]
      4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
uid                 [  full  ] Bart Prokop (code signing) <bart@prokop.dev>
ssb#  ed25519/3A25D90B2E35828B 2024-09-15 [S]
ssb   ed25519/3F6970DA4BD40251 2024-09-26 [S]
```

Note that only single subkey is available for use.

## Git Code signing

### Configure Git to allow signing

First set your name and email.

```bash
$ git config --global user.name "Bart Prokop"
$ git config --global user.email "bart@prokop.dev"
```

Then instruct GPG to sign your commits.

```bash
$ gpg --list-secret-keys --keyid-format=long
sec   ed25519/CE4B83F76925A7FA 2024-09-13 [C]
      4CC5B7436206D43D89BC8A22CE4B83F76925A7FA
uid                 [ultimate] Bart Prokop (code signing) <bart@prokop.dev>
ssb   ed25519/3A25D90B2E35828B 2024-09-15 [S]

$ git config --global user.signingkey 3A25D90B2E35828B
$ git config --global commit.gpgsign true
```
