# Signing with GPG

## Code signing

### Adopted solution

- I'm using multiple machines to develop the code, so I'm committing from multiple machines.
- I prefer not to share single signing key across multiple machines. If one machine is compromised, all my work will be compromised (as long as GPG signature is considered).
- I will generate a single signing sub-key per machine.

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
