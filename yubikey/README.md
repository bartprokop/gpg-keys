# YubiKey 5C NFC

Short documentation how to set-up YubiKey OpenPGP applet.

## Checking if everything works

Basic info check can be done with the YubiKey Manager command line:

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

Then follow up with an GPG operational check:

```
$ gpg --card-status
Reader ...........: Yubico YubiKey OTP FIDO CCID 0
Application ID ...: D******************************0
Application type .: OpenPGP
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 2******8
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

## Configuring OpenPGP card - prior to key generation

We want to switch from RSA to EC. My YubiKey support the following EC curves (you might need to use `gpg --card-edit --expert` mode):

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
Application ID ...: D******************************0
Application type .: OpenPGP
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 2******8
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

The `Key attributes` above proves we have the expected key types set.

It is also worth to "personalise" OpenPGP applet, but those values are not really used by GPG:

```
gpg/card> name
Cardholder's surname: Prokop
Cardholder's given name: Bart

gpg/card> lang
Language preferences: en

gpg/card> salutation
Salutation (M = Mr., F = Ms., or space): M
```

## Generating the keys

### Sort out PINs prior to generating keys

```
$ gpg --card-edit

gpg/card> admin

gpg/card> passwd
gpg: OpenPGP card no. D******************************0 detected

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 3
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 4
Reset Code set.

gpg/card> list

Reader ...........: Yubico YubiKey OTP FIDO CCID 0
Application ID ...: D******************************0
Application type .: OpenPGP
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 2******8
Name of cardholder: Bart Prokop
Language prefs ...: en
Salutation .......: Mr.
URL of public key : [not set]
Login data .......: [not set]
Signature PIN ....: not forced
Key attributes ...: ed25519 cv25519 ed25519
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 3 3
Signature counter : 0
KDF setting ......: on
UIF setting ......: Sign=off Decrypt=off Auth=off
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]
General key info..: [none]
```

### On card key generation

```
$ gpg --card-edit

admin
Admin commands are allowed

gpg/card> generate
Make off-card backup of encryption key? (Y/n) n
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
Email address: bart@prokop.dev
Comment: YubiKey 5C NFC
You selected this USER-ID:
    "Bart Prokop (YubiKey 5C NFC) <bart@prokop.dev>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
gpg: revocation certificate stored as '/c/Users/proko/.gnupg/openpgp-revocs.d/6CFEDB429D84EDCE69CCF1CEEA86B9CE4982B047.rev'
public and secret key created and signed.

gpg/card> quit
pub   ed25519 2024-09-03 [SC]
      6CFEDB429D84EDCE69CCF1CEEA86B9CE4982B047
uid                      Bart Prokop (YubiKey 5C NFC) <bart@prokop.dev>
sub   ed25519 2024-09-03 [A]
sub   cv25519 2024-09-03 [E]
```

### Check what private keys are there

```
$ gpg --list-secret-keys --keyid-format=long
/c/Users/proko/.gnupg/pubring.kbx
---------------------------------
sec>  ed25519/EA86B9CE4982B047 2024-09-03 [SC]
      6CFEDB429D84EDCE69CCF1CEEA86B9CE4982B047
      Card serial no. = 0006 22479868
uid                 [ultimate] Bart Prokop (YubiKey 5C NFC) <bart@prokop.dev>
ssb>  ed25519/D6296AB7E297F23D 2024-09-03 [A]
ssb>  cv25519/BB8211404F0A06C6 2024-09-03 [E]
```

### Attestation

The below creates digital signature using Yubikey hardcoded certificates:

```
C:\Program Files\Yubico\YubiKey Manager>ykman.exe openpgp keys attest SIG \Users\proko\code\gpg-keys\yubikey\sig-EA86B9CE4982B047.crt
Enter PIN:

C:\Program Files\Yubico\YubiKey Manager>ykman.exe openpgp keys attest ENC \Users\proko\code\gpg-keys\yubikey\enc-BB8211404F0A06C6.crt
Enter PIN:

C:\Program Files\Yubico\YubiKey Manager>ykman.exe openpgp keys attest AUT \Users\proko\code\gpg-keys\yubikey\aut-D6296AB7E297F23D.crt
Enter PIN:
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
