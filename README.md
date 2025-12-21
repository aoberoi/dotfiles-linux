# dotfiles-linux

Another dotfiles repo

🥼 **Experimental**: I'm still figuring out how I want this set of dotfiles to work. Don't trust it.

## Set up

1. Clone
2. Install `chezmoi`
3. `chezmoi apply`

## Operating

### GPG Keys and Secrets

The primary key is in encrypted storage and never needs to "come online". Therefore the primary key is
not directly handled in the scripts in this repo. However, the primary key will need to be used to
generate new subkeys as older subkeys expire. Similarly, the revocation certificate for the primary
key is also stored in encrypted storage and never needs to "come online". The revocation cert does not
need to be used at all, unless the primary key is compromised.

Subkeys are used for actual day-to-day operations such as signing and encrypting. The script
`.chezmoiscripts/run_once_import-gpg-subkeys.sh.tmpl` is used to load the subkeys from an export that
is stored in 1Password into the local machine's gpg client. However subkeys will eventually expire and
will need to be replaced by new subkeys. There's [a runbook](runbooks/gpg_subkey_rotation_runbook.md) for this procedure.

Helpful tips: 
- Update the 1Password item with the new file (like a key export) `op document edit <item_name> <filename>`.
- Export a public key: `gpg --export --armor {FINGERPRINT} > publickey.asc`
- The web of trust is bound to grow and change as it getrs used. It's not critical on its own, but its good
  to back up periodically. The `.chezmoiscripts/run_once_import-gpg-ownertrust.sh.tmpl` script imports the
  backup from 1Password into the local machine. Read that script to understand how the web of trust should
  be backed up and updated.

## Helpful commands

Test script or template changes by executing the template:

```
$ chezmoi execute-template < "$(chezmoi source-path)/.chezmoiscripts/some_script.sh.tmpl"
```
