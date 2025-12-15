# dotfiles-linux

Another dotfiles repo

🥼 **Experimental**: I'm still figuring out how I want this set of dotfiles to work. Don't trust it.

## Set up

1. Clone
2. Install `chezmoi`
3. `chezmoi apply`

## Operating

### Manual sync

When data or config doesn't live in a format that is designed to be portable (e.g. mixed with caches and
generated data), we might have some manual steps to keep things in sync.

* gpg
    * private key
      - The update procedure is described in `.chezmoiscripts/run_once_import-gpg-private-key.sh.tmpl`.
      - List current keys: `gpg --list-secret-keys --keyid-format long`
      - Export the current private key into a file: `gpg --export-secret-keys --armor {FINGERPRINT} > privatekey.asc` ⚠ **Delete this file when you're done with it**.
      - Update the 1Password item with the new private key export: `op document edit gpg-privatekey-primary privatekey.asc`.
      - Add a 1Password item for the revocation cert: `op document create ~/.gnupg/openpgp-revocs.d/{NEW_FINGERPRINT}.rev --title "gpg-revocation-{FIRST_EIGHT_FINGERPRINT}"`
      - The corresponding public key can be exported after the private key is restored: `gpg --export --armor {FINGERPRINT} > publickey.asc`
    * web of trust
      - The web of trust is bound to grow and change as it gets used. It's not critical on its own, but its
        good to backup preiodically.
      - The update procedure is described in `.chezmoiscripts/run_once_import-gpg-ownertrust.sh.tmpl`.
      - Export the current web of trust into a file: `gpg --export-ownertrust > ownertrust.txt` **Delete this file when you're done with it**.
      - Update the 1Password item with the new private key export: `op document edit gpg-ownertrust ownertrust.txt`. 

## Helpful commands

Test script or template changes by executing the template:

```
$ chezmoi execute-template < "$(chezmoi source-path)/.chezmoiscripts/some_script.sh.tmpl"
```
