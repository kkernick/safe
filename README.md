# Safe

Securely store secrets

## Installation

On Arch-Based Distributions, download the PKGBUILD and run `makepkg -si`

For others, ensure that `systemd` `clevis` `openssl` `python-blessed` `python-termcolor` `gocryptfs` `sudo` `oath-toolkit` `plasma-workspace` and `qt6-tools` are installed, and `chmod +x` the script.

## Usage

`safe` is a utility for storing secrets persistently on disk. To ensure that secrets are securely stored, it uses a `gocryptfs` filesystem, to which all secrets are stored with an identifier, which is used to encrypt the secret itself.

To use `safe`, you will need to create a `gocryptfs` filesystem, located at the `--path` argument of `safe`, which defaults to `$XDG_DATA_HOME/safe` To initialize the filesystem, run:

```bash
gocryptfs -init $XDG_DATA_HOME/safe
```

From there, you can then run `safe`, with some additional arguments:

1. `-p/--path`: The path of the `gocryptfs` folder. Defaults to `$XDG_DATA_HOME/safe`
2. `-m/--mount`: Where to mount the safe. Defaults to `$XDG_CACHE_HOME/safe`
3. `-a/--address`: A TPM address which `safe` can load the password for the `gocryptfs` vault using `handle.tpm` It uses the `-n` argument, so you must seal the secret accordingly.
4. `-r/--rounds`: Rounds used for `openssl` encryption. Defaults to 500000

Once the vault has been loaded, there are several options modifiers that can be toggled besides the main operations:

1. TPM (t): Use `clevis` to encrypt the secret with the TPM first, then encrypt it a second time with the identifier. This ensures that secrets appear the same when viewed.
2. Master Key (m): Load a secret from the safe as a Master Key. All secrets will be encrypted with its identifier + the Master Key appended onto it. This not only improves security by increasing the size of the password, but obfuscates what information is stored in the safe by abstracting the identifier itself.
3. Generator (g): Randomly generate a password for an associated identifier, with a provided key length.

All can be toggled freely with their associated key.

### Encryption

`safe` stores secrets by using an identifier scheme. In this, a provided identifier will be provided first, whose value will be used to directly encrypt the secret. Then, the identifier is hashed using a secure hashing algorithm (SHA512), which is then used as the filename of the secret. Therefore, the identifier is not only needed to decrypt the secret, but to even know what secrets are stored. Simply looking at the content of the safe (Assuming its been mounted by `gocryptfs`) will make it impossible to know what files are associated with what secret.

Identifiers, therefore, must be unique, secure strings--treat them like a password. For example, to store the password to your email, don't use `email` as the identifier, create a random identifier than can be remembered. 

If a generator is used, `safe` will securely generate a secret using `openssl` instead of needing to provide one directly.

If the TPM is in use, the secret will first be encrypted via `clevis`, binding the secret to the current machine, and then encrypted a second time with the identifier. This ensures that the secrets (Again, assuming access to the unencrypted `gocryptfs` folder) are uniform, and it is impossible to tell which secrets are using the TPM, and which are not. **However**, the TPM will bind the secret to the machine, which means other machines will be unable to decrypt it.

Finally, if a master key is in use, the secret associated with it will be appended to the identifier to "salt" it, not only increasing the security of the secret (An identifier will still be required to decrypt), but conceals the identifier further. If a predictable or leaked scheme is used for the identifier, a master key can prevent it from being used to gain access to secrets.

A master key is a secret like any other in the safe, so to use a master key, one must store it within the safe. Then, returning to the main menu, typing *m* and providing the identifier will load the value to be used on subsequent encryption/decryption operations.

### Decryption

Decryption works in inverse of encryption. An identifier is provided, and if that identifier exists it will be decrypted and copied to the clipboard.

You'll need to ensure that the TPM + Master Key are in the same state as when encrypted, otherwise `safe` will be unable to find/decrypt the secret.

`safe` uses `klipper` as the clipboard manager. This is because `wl-clipboard` does not properly clear the clipboard on `KDE`, and thus cannot be used to reliably expunge secrets once copied.

### Deletion

Deletion takes an identifier + Master Key, and deletes the secret from the vault. You will need to type "YES" to delete the secret.

Because secrets are typically rather small, its usually recommended to keep secrets, rather than deleting them, as they increase the size of the vault, and make it considerably more difficult to determine what secrets could be stored within the safe. 

### Securely Erasing Memory

`safe` is written in Python, which has no mechanism for securely erasing memory. This can pose a security vulnerability where intermediary outputs, such as the identifier, its hash, or the secret, can be leaked. To avoid this, add the following kernel parameter to your kernel boot arguments: `init_on_free=1`. This will ensure that memory pages will be initialized (zeroed) when freed.