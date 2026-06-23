# password-store

## Setup

The following repository use some tooling to ensure a good Developer Experience :

- [Password Store](https://www.passwordstore.org)
- [GPG](https://gnupg.org/)
- [Mise-en-Place](https://mise.jdx.dev/)

Please, follow the instructions at each of the official sites to setup you env accordingly.

## Usage

```bash
# fork/clone this template repo
cd <repo>
mise trust # only first time
```

## What is it ?

We use password-store project and its `pass` command to protect passwords and secrets.

Password store encrypts all data with the public GPG keys of each authorized users, this means only these
authorized users can retrieve the data, using their individual GPG secret key.

Each path can be subjects to different access rights, for example, the path `example/production` can be
limited to the production team, while others paths can be accessible to the whole company.

## Prerequisites

You need gpg installed and a GPG key, please create one specifically for all your work.
You will also need to have a GPG agent running to avoid typing your passphrase each time you look up a secret with `pass`

If you need access to some data and it is not encrypted for your public key, you need someone who has access to
add your public key to the list and re-encrypt the data.

## Daily usage

The `PASSWORD_STORE_DIR` environment variable needs to point to the password-store repository.
This is done for you automatically in repository that need the password-store through `mise-en-place`.

### GnuPG

- To generate a new key pair for a user:

```bash
gpg --generate-key
# [...]
#
# Real name: User Name
# Email address: user@example.com
# You selected this USER-ID:
#     "User Name <user@example.com>"
# 
# Change (N)ame, (E)mail, or (O)kay/(Q)uit? O
# [...]
# gpg: key 9172FE87F5DA8F77 marked as ultimately trusted
# gpg: directory '/home/user/.gnupg/openpgp-revocs.d' created
# gpg: revocation certificate stored as '/home/user/.gnupg/openpgp-revocs.d/A208302242581A8E70B1304E9172FE87F5DA8F79.rev'
# public and secret key created and signed.
# 
# pub   rsa3072 2022-04-11 [SC] [expires: 2024-04-10]
#       A208302242581A8E70B1304E9172FE87F5DA8F79
# uid                      User Name <user@example.com>
# sub   rsa3072 2022-04-11 [E] [expires: 2024-04-10]
```

```bash
gpg --list-keys
# [...]
# -------------------------------------
# pub   rsa3072 2022-04-11 [SC] [expires: 2024-04-10]
#       A208302242581A8E70B1304E9172FE87F5DA8F79
# uid           [ultimate] User Name <user@example.com>
# sub   rsa3072 2022-04-11 [E] [expires: 2024-04-10]
```

- To export a key:

```bash
gpg --export -a User Name > uname.asc
```

- To check if pass is well setup: `pass list`
- To check if you have access: `pass some/secret/path`
- To give access to another co-worker:

import their public keys (all public keys are present in ```~/password-store/gpg-keys```

```bash
gpg --import *
```

Change the gpg trust level of the imported keys to 5 (maximum trust):

```bash
gpg --list-keys
# pub   rsa4096 2020-09-03 [SC]
#       AAE276AE5745E13C95336B263204648F2C48EA42
# uid           [ unknown] Ivan Beauté <user@gmail.com>
# sub   rsa4096 2020-09-03 [E]
# ... lot of output
# 
```

```bash
gpg --edit-key AAE276AE5745E13C95336B263204648F2C48EA42...
# gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
# This is free software: you are free to change and redistribute it.
# There is NO WARRANTY, to the extent permitted by law.
# 
# pub  rsa2048/A5165366C6C654F0
#      created: 2020-03-20  expires: 2023-03-16  usage: SC
#      trust: unknown       validity: unknown
# sub  rsa2048/8382B5DB0572A407
#      created: 2020-03-20  expires: 2023-03-16  usage: E
# [ unknown] (1). user Leger <frederic@gmail.com>
# 
# gpg> trust
# pub  rsa2048/A5165366C6C654F0
#      created: 2020-03-20  expires: 2023-03-16  usage: SC
#      trust: unknown       validity: unknown
# sub  rsa2048/8382B5DB0572A407
#      created: 2020-03-20  expires: 2023-03-16  usage: E
# [ unknown] (1). user Leger <user@gmail.com>
# 
# Please decide how far you trust this user to correctly verify other users' keys
# (by looking at passports, checking fingerprints from different sources, etc.)
# 
#   1 = I don't know or won't say
#   2 = I do NOT trust
#   3 = I trust marginally
#   4 = I trust fully
#   5 = I trust ultimately
#   m = back to the main menu
# 
# Your decision? 5
# Do you really want to set this key to ultimate trust? (y/N) y
# 
# pub  rsa2048/A5165366C6C654F0
#      created: 2020-03-20  expires: 2023-03-16  usage: SC
#      trust: ultimate      validity: unknown
# sub  rsa2048/8382B5DB0572A407
#      created: 2020-03-20  expires: 2023-03-16  usage: E
# [ unknown] (1). user Leger <user@gmail.com>
# Please note that the shown key validity is not necessarily correct
# unless you restart the program.
# 
# gpg> quit
```

### Password Store

To use password store follow these advices:

- Never use file extension (ex: .gpg)
- Make sure your keyring is setup (gpg keys loaded and trusted)
- Define the list of GPG identifiers in the `.gpg-id` file (use `gpg --list-keys` for the list of available identifiers)

#### Encrypt / Re-Encrypt a repository/folder

```bash
pass init -p . $(cat .gpg-id)
```

### Adding Secrets

```bash
# example
pass insert dev/databases/main
# Enter password for dev/databases/main:
# Retype password for dev/databases/main:
# [main 067c914] Add given password for dev/databases/main to store.
# 1 file changed, 0 insertions(+), 0 deletions(-)
```

Or if you have multiple lines to store, use the `-m` flag:

```bash
# example
pass insert -m some/secret/pass
```

Of if you havec a more complex object to store, you can use the `edit` command:

```bash
# example
pass edit some/secret/pass
# 1st line is password
# other lines can have some key=value pairs
# example :
# SomePassword!
# username: john.doe
# email: john.doe@example.com
```

See password store documentation for more details.
