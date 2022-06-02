# GPG Keys

This repository contains my GPG keys currently in use.

## My public GPG keys

## Generation of GPG keys

To generate GPG keys, use the following command and select 9) ECC and ECC folloed by 1) Curve 25519.

```bash
gpg --expert --full-gen-key
```

To export public key use:

```bash
gpg --export --armour > KEY_ID.pub
```

To configure key to be used for signing git commits:

```bash
git config --global user.email "bart@prokop.dev"
git config --global user.name "Bart Prokop"
git config --global commit.gpgsign true

gpg --list-secret-keys --keyid-format LONG
git config --global user.signingkey KEY_ID
```
