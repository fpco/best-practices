Password store
==============

We keep shared admin secrets in the a Git reporitory. We also have separate
password store Git repositories for some clients. The files in these
repositories are managed by [pass](https://www.passwordstore.org/).

In this document, `GPG-ID` should always be a key signature in "0xlong" format
(e.g. `0x65101FF31C5C154D`). To have GPG always display the long format, put
`keyid-format 0xlong` in `~/.gnupg/gpg.conf`, as recommended in the
[riseup.net best practices](https://help.riseup.net/en/security/message-security/openpgp/best-practices#dont-rely-on-the-key-id).

Prerequisites
-------------

* Install GPG, using the standard method for your OS. This document was written
  using GPG 1.4, but GPG 2.x should work equally well (just replace `gpg` with
  `gpg2` everywhere).  See
  [this blog post](https://www.fpcomplete.com/blog/2016/05/stack-security-gnupg-keys)
  for some more information on selecting a version and installing it.

* Create a GPG private key. Follow GPG best practices, such as those from
  [riseup.net](https://help.riseup.net/en/security/message-security/openpgp/best-practices).
  See [this blog post](https://www.fpcomplete.com/blog/2016/05/stack-security-gnupg-keys)
  for more detailed instructions on creating a good private key.

* [Download](https://www.passwordstore.org/#download) and install `pass` for
  your operating system. Ensure that you're installing version 1.6.* or higher.
  You may have to build from source (`1.4.2` is the latest in the trusty apt
  repo, for instance).

Using an existing keystore
--------------------------

### Setup

 1. Clone the repo: `git clone <REPO-URL>
    ~/path/to/password-store`. Adjust the destination directory to suit your needs.
    `pass` looks in `~/.password-store` by default, so if you will only ever use
    one password storage repo, you can put it there.

 2. If you cloned to a location other than `~/.password-store`, set the
    `PASSWORD_STORE_DIR` environment variable to that path.

 3. Change directory to the password store directory.

 4. Get all admins' public keys using `gpg --recv-keys $(cat .gpg-id)`.

 5. If you have not already done so: verify, sign, and marginally trust at least
    three other admins' keys:

     1. Get the fingerprint using `gpg --fingerprint <GPG-ID>`, and verify with
        the key's owner out-of-band.
     2. Sign the key: `gpg --sign-key <GPG-ID>`.
     3. Upload the signed key to the keyserver: `gpg --send-key <GPG-ID>`.
     3. Trust the key:
         1. Run `gpg --edit-key <GPG-ID>`.
         2. At the `gpg>` prompt, enter `trust`, and select **3 = I trust marginally**.
         3. At the `gpg>` prompt, enter `save`.

### Usage

In general, see the `pass` [introduction](https://www.passwordstore.org/) and
[man page](https://git.zx2c4.com/password-store/about/).

Don't forget to pull the repo (`pass git pull`) before making any changes, and
push it (`pass git push`) afterward.

#### Binary files

For binary files, add using `pass insert -m PASS-NAME <PATH`. Use `pass show
PASS-NAME|sha1sum` to compare the checksum afterward, just to double check.

### Troubleshooting

#### Failure to encrypt

Ensure you have all the admins' public keys in your keyring, in case a new one
has been added. `gpg --recv-keys $(cat .gpg-id)` will download them all.

If you get a message like:

> gpg: 2500AB5A: There is no assurance this key belongs to the named user

it means the web of trust is not quite complete enough. Try receiving the keys
again (`gpg --recv-keys $(cat .gpg-id)`) in case additional signatures have been
uploaded to the keyserver, and if that doesn't help, trust some more other
admins (see [Setup](#setup)) and/or verify/sign/upload the key yourself.

### Onboarding

To add a new user who can access the shared secrets:

 1. Get their GPG public key (paste into `gpg --import` or get from keyserver
    using `gpg --recv-key <GPG-ID>`).
 2. Verify the fingerprint (`gpg --fingerprint <GPG-ID>`) out-of-band.
 3. Sign the key (`gpg --sign-key <GPG-ID>`).
 4. Upload the the signature to a keyserver (`gpg --send-key <GPG-ID>`).
 5. Re-encrypt the password store, adding the new user's key: `pass init $(cat
    .gpg-id) <GPG-ID>`. Be sure to use the "0xlong" keyid format. You probably
    want to be running `gpg-agent`, otherwise you will have to enter your
    passphrase over and over again.
 6. At least two other admins should also verify, sign, and upload the new key,
    to help form a complete web of trust.

Creating a new keystore
-----------------------

 1. Set the `PASSWORD_STORE_DIR` environment variable to the local path
    of the new password store (e.g. `~/path/to/password-store`).
 2. Receive, verify, and sign the keys of everyone who will have initial access to the
    store, as when [onboarding](#onboarding).
 3. Run `pass init <GPG-ID> <GPG-ID> ...` with the keys of everyone who will
    have initial access.
 4. Run `pass git init`.
 5. Create a private Git repository on your preferred host (e.g. Github).
 6. Run `pass git remote add origin <REPO>`.
 7. Run `pass git push -u origin master`.
