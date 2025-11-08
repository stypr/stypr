# Public Key Verification

This document describes how to securely verify and import stypr's root and subordinate PGP keys.

Ensure that your network connection is secure before following these steps.

You can also contact stypr over a different secure communication medium for additional verifications.


## 1. Verify the root key

The following script downloads and verifies the stypr's root public key.

```sh
#!/usr/bin/env bash
set -euo pipefail

# Make sure to confirm public key ID before running the script
PUBKEY_FILENAME="root.pub.asc"
PUBKEY_ID="0x2064ff9330111a9094b319dab43975c459ed7a46"

# Download public keys from different sources
wget -q -O $PUBKEY_FILENAME.1 "https://harold.kim/keys/$PUBKEY_FILENAME"
wget -q -O $PUBKEY_FILENAME.2 "https://raw.githubusercontent.com/stypr/stypr/refs/heads/main/keys/$PUBKEY_FILENAME"
wget -q -O $PUBKEY_FILENAME.3 "https://keyserver.ubuntu.com/pks/lookup?op=get&search=$PUBKEY_ID"

# Compute key lists for convenience
PUBKEY_V1=$(gpg --with-colons --import-options show-only --import ./$PUBKEY_FILENAME.1 | sort | shasum -a 512)
PUBKEY_V2=$(gpg --with-colons --import-options show-only --import ./$PUBKEY_FILENAME.2 | sort | shasum -a 512)
PUBKEY_V3=$(gpg --with-colons --import-options show-only --import ./$PUBKEY_FILENAME.3 | sort | shasum -a 512)
echo "===================="
echo "Source 1: $PUBKEY_V1"
echo "Source 2: $PUBKEY_V2"
echo "Source 3: $PUBKEY_V3"
echo "===================="

# Ensure that ALL downloaded keys are identical
if [[ "$PUBKEY_V1" == "$PUBKEY_V2" && "$PUBKEY_V1" == "$PUBKEY_V3" ]]; then
  echo "[*] All keys are identical!"
  exit 0
else
  echo "[!] Mismatch detected!"
  exit 1
fi
```

Confirm that **all** keys from verified sources are identical.

```sh
$ chmod +x ./verify.sh
$ ./verify.sh
====================
Source 1: 872b6f38049c43f4e4689ed816bc4ba93ce04d8369a66156ec325ee2d4206d4445432cc82fc9a6fb869edbec799f88d8cf39b9fa5114d43d9f3f0d31fe5dc4c3  -
Source 2: 872b6f38049c43f4e4689ed816bc4ba93ce04d8369a66156ec325ee2d4206d4445432cc82fc9a6fb869edbec799f88d8cf39b9fa5114d43d9f3f0d31fe5dc4c3  -
Source 3: 872b6f38049c43f4e4689ed816bc4ba93ce04d8369a66156ec325ee2d4206d4445432cc82fc9a6fb869edbec799f88d8cf39b9fa5114d43d9f3f0d31fe5dc4c3  -
====================
[*] All keys are identical!
```


## 2. Import and trust the root key

Import the root public key.

```sh
$ gpg --import root.pub.asc.1
gpg: key B43975C459ED7A46: public key "stypr root" imported
gpg: Total number processed: 1
gpg:               imported: 1
$
```

Now, continue and trust the key.
```sh
$ gpg --edit-key B43975C459ED7A46
...

pub  ed25519/B43975C459ED7A46
     created: 2023-01-14  expires: 2043-01-09  usage: SC
     trust: full          validity: unknown
sub  cv25519/0B4442E83302154D
     created: 2023-01-14  expires: 2043-01-09  usage: E
sub  ed25519/39AC2D4D30EFE7B7
     created: 2023-01-14  expires: 2043-01-09  usage: A
[ unknown] (1). stypr root

gpg> trust

...

Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y

...

Please note that the shown key validity is not necessarily correct
unless you restart the program.

gpg> q
```

## 3. Download subordinate keys

Use WKD to download the remaining keys.

```sh
$ gpg --locate-external-keys --auto-key-locate wkd me@harold.kim
gpg: key 87C4CD66A509906B: public key "Harold Kim (General) <me@harold.kim>" imported
gpg: key B43975C459ED7A46: no valid user IDs
gpg: key F01CD491240E43A6: public key "Harold Kim (Confidential) <me@harold.kim>" imported
gpg: Total number processed: 3
gpg:           w/o user IDs: 1
gpg:               imported: 2
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   2  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: depth: 1  valid:   2  signed:   0  trust: 2-, 0q, 0n, 0m, 0f, 0u
gpg: next trustdb check due at 2043-01-09
pub   ed25519 2023-01-14 [SC] [expires: 2043-01-09]
      4F3D0B5DA557FC3535ACEE3F87C4CD66A509906B
uid           [  full  ] Harold Kim (General) <me@harold.kim>
sub   cv25519 2023-01-14 [E] [expires: 2043-01-09]
sub   ed25519 2023-01-14 [A] [expires: 2043-01-09]
```


## 4. Confirm key validation

If all keys are marked as `[full]` or `[ultimate]`, your environment is now correctly configured to verify signed commits 
or encrypted communications from stypr.

```
$ gpg --list-keys
-----------------------------------------------
pub   ed25519 2023-01-14 [SC] [expires: 2043-01-09]
      2064FF9330111A9094B319DAB43975C459ED7A46
uid           [ultimate] stypr root
sub   cv25519 2023-01-14 [E] [expires: 2043-01-09]
sub   ed25519 2023-01-14 [A] [expires: 2043-01-09]

pub   ed25519 2023-01-14 [SC] [expires: 2043-01-09]
      4F3D0B5DA557FC3535ACEE3F87C4CD66A509906B
uid           [  full  ] Harold Kim (General) <me@harold.kim>
sub   cv25519 2023-01-14 [E] [expires: 2043-01-09]
sub   ed25519 2023-01-14 [A] [expires: 2043-01-09]

pub   ed25519 2023-01-14 [SC] [expires: 2043-01-09]
      9C1D006897CD998081C7A457F01CD491240E43A6
uid           [  full  ] Harold Kim (Confidential) <me@harold.kim>
sub   cv25519 2023-01-14 [E] [expires: 2043-01-09]
sub   ed25519 2023-01-14 [A] [expires: 2043-01-09]
```
