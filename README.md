# GPG Keys

This repository contains my GPG keys currently in use.

## All my GPG keys

| Account    | Key file             | Fingerprint                                        |
| ---------- | -------------------- | -------------------------------------------------- |
| bart@gcs   | B61CDFDAC50F3186.pub | D9E1 B3CE B81F CDFD 2026  4CF0 B61C DFDA C50F 3186 |
| bart@paris | bart-paris.pub       |  |
| bart@t20   | bart-t20.pub         |  |

## Generation of GPG keys

To generate GPG keys, use the following command and select 9) ECC and ECC folloed by 1) Curve 25519.

```bash
gpg --expert --full-gen-key
```

To export public key use:

```bash
gpg --export --armour > KEY_ID.pub
gpg --fingerprint
```

To configure key to be used for signing git commits:

```bash
git config --global user.email "bart@prokop.dev"
git config --global user.name "Bart Prokop"
git config --global commit.gpgsign true

gpg --list-secret-keys --keyid-format LONG
git config --global user.signingkey KEY_ID
```
