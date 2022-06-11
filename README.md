# GPG Keys

This repository contains my GPG keys currently in use.

## All my GPG keys

### Low value GPG keys

The low value GPG keys are tied to account I have on various machines I work on.
Those varies from personal laptop, through dedicated cloud servers to cloud shells.

| Account    | Key file             | Fingerprint                                        |
| ---------- | -------------------- | -------------------------------------------------- |
| bart@gcs   | B61CDFDAC50F3186.gpg | D9E1 B3CE B81F CDFD 2026  4CF0 B61C DFDA C50F 3186 |
| bart@paris | F69338AE4FE91AFB.gpg | 07EE 7CE8 2136 B462 9979  A74E F693 38AE 4FE9 1AFB |
| bart@t20   | bart-t20.pub         |  |

## Command line cheatsheet

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

To import a (public) key from file:

```bash
gpg --import public_key.gpg
gpg --sign-key KEY_IDENTIFIER
```

To import a (public) key from key server

```
gpg --recv-key KEY_IDENTIFIER
gpg -u SIGNING_KEY --sign-key KEY_TO_SIGN
gpg --list-signatures KEY_IDENTIFIER
gpg --send-keys KEY_IDENTIFIER
```

To delete keys from keyring

```
gpg --delete-keys KEY_IDENTIFIER
gpg --delete-secret-keys KEY_IDENTIFIER
```
