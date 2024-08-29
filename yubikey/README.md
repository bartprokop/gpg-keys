# YubiKey 5C NFC

## Checking if everything works

Basic info check with YubiKey Manager command line:

```
C:\Program Files\Yubico\YubiKey Manager>ykman.exe openpgp info
OpenPGP version:            3.4
Application version:        5.4.3
PIN tries remaining:        3
Reset code tries remaining: 0
Admin PIN tries remaining:  3
Require PIN for signature:  Once
Touch policies:
  Signature key:      Off
  Encryption key:     Off
  Authentication key: Off
  Attestation key:    Off
```

GPG operational check with command line:

```
$ gpg --card-status
Reader ...........: Yubico YubiKey OTP FIDO CCID 0
Application ID ...: D2760001240100000006224798680000
Application type .: OpenPGP
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 22479868
Name of cardholder: [not set]
Language prefs ...: [not set]
Salutation .......:
URL of public key : [not set]
Login data .......: [not set]
Signature PIN ....: not forced
Key attributes ...: rsa2048 rsa2048 rsa2048
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 0 3
Signature counter : 0
KDF setting ......: off
UIF setting ......: Sign=off Decrypt=off Auth=off
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]
General key info..: [none]
```

## Resetting and KDF setup

That how I treat every new YubiKey, note that those commands are irreversible and destructive.

```
C:\Program Files\Yubico\YubiKey Manager>ykman.exe openpgp reset
WARNING! This will delete all stored OpenPGP keys and data and restore factory settings. Proceed? [y/N]: y
Resetting OpenPGP data, don't remove the YubiKey...
Success! All data has been cleared and default PINs are set.
PIN:         123456
Reset code:  NOT SET
Admin PIN:   12345678
```

Then let's enable KDF:

```bash
$ gpg --edit-card
gpg/card> admin
gpg/card> kdf-setup
```

## Configuring card - prior to key generation

We want to switch from RSA to EC.
YubiKey support the following EC curves (you might need to use `gpg --card-edit --expert` mode):

```
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
```

Let's use Admin PIN to switch our keys to Curve 25519.

```
$ gpg --card-edit

gpg/card> admin
Admin commands are allowed

gpg/card> key-attr
Changing card key attribute for: Signature key
Please select what kind of key you want:
   (1) RSA
   (2) ECC
Your selection? 2
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection? 1
The card will now be re-configured to generate a key of type: ed25519
Note: There is no guarantee that the card supports the requested
      key type or size.  If the key generation does not succeed,
      please check the documentation of your card to see which
      key types and sizes are supported.
Changing card key attribute for: Encryption key
Please select what kind of key you want:
   (1) RSA
   (2) ECC
Your selection? 2
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection? 1
The card will now be re-configured to generate a key of type: cv25519
Changing card key attribute for: Authentication key
Please select what kind of key you want:
   (1) RSA
   (2) ECC
Your selection? 2
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection? 1
The card will now be re-configured to generate a key of type: ed25519

gpg/card> list

Reader ...........: Yubico YubiKey OTP FIDO CCID 0
Application ID ...: D2760001240100000006224798680000
Application type .: OpenPGP
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 22479868
Name of cardholder: [not set]
Language prefs ...: [not set]
Salutation .......:
URL of public key : [not set]
Login data .......: [not set]
Signature PIN ....: not forced
Key attributes ...: ed25519 cv25519 ed25519
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 0 3
Signature counter : 0
KDF setting ......: on
UIF setting ......: Sign=off Decrypt=off Auth=off
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]
General key info..: [none]
```



## Additional resources

- https://developers.yubico.com/PGP/opgp-attestation-ca.pem
- https://www.procustodibus.com/blog/2023/04/how-to-set-up-a-yubikey/
- https://security.stackexchange.com/questions/260911/how-to-generate-ssh-keys-inside-yubikey

https://www.liverpool.ac.uk/it/security/two-factor/

```
C:\Program Files\Yubico\YubiKey Manager>ykman.exe piv reset
WARNING! This will delete all stored PIV data and restore factory settings. Proceed? [y/N]: y
Resetting PIV data...
Success! All PIV data have been cleared from the YubiKey.
Your YubiKey now has the default PIN, PUK and Management Key:
        PIN:    123456
        PUK:    12345678
        Management Key: 010203040506070801020304050607080102030405060708
```


$ gpg-connect-agent "SCD RANDOM 10" /bye







1 pazdziernika 


