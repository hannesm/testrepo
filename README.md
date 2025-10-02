# conex test-repository

You need conex from main branch to use this, `opam pin add conex --dev` should do the trick!

To test `conex_verify`, you need to add the following to your `$(OPAMROOT)/config`:

```
repository-validation-command: [
  "conex_verify_mirage_crypto" "--quorum" "%{quorum}%"
  "--trust-anchors" "%{anchors}%"
  "--repo" "%{repo}%"
  "--dir=%{dir}%" { ! incremental }
  "--patch=%{patch}%" { incremental }
  "--incremental" { incremental }
  "--no-opam"
]
```

You can use `conex_verify_openssl` instead of `mirage_crypto`, add `-v` flags.
This will only be used for repositories which have trust anchors and a quorum
configured. Updates of the default `opam-repository` will not be affected.

To add this repository with a quorum of 1, you have to type:

```
opam repo add conex-test https://github.com/hannesm/testrepo.git 1 sha256=5a148d3977cb03dbeaeb99fa7033b5c7a43c8c7ee1114fee0c22fada2f7c9687
```

Vary the quorum or the fingerprints to see verification failures.

## Private keys

For maintainer `m1`, maintainer `m2`, and maintainer `m3` are help for test
purposes in `priv`.  Do not use these private keys elsewhere, generate your own
instead.  The `rootA` key is also in `priv`.

If you clone this repository and `cp priv/* ~/.conex/`, you'll be able to sign
updates.

## Creation of this repository

Initial setup:

```
$ mkdir -p /tmp/testrepo/packages/foo/foo.0.1.0
$ echo > /tmp/testrepo/packages/foo/foo.0.1.0/opam
# init some keys
$ conex_key --id rootA
$ conex_key --id m1
$ conex_key --id m2
$ conex_key --id m3

# root
$ conex_root create
$ conex_key --id rootA --pub
$ conex_root add-key --id rootA --key-data <rootA public key>
$ conex_root add-to-role --role maintainer --id m1 --key-hash 2ad4be04dc2975ffb535051182bfd3b50bc335eca8217bee157f28459794d0df
$ conex_root add-to-role --role maintainer --id m2 --key-hash bc89a93e3864fae4b8e854a9bdc321f7e7ed773847c8faf96f49bc28ec87063b
$ conex_root add-to-role --role maintainer --id m3 --quorum 2 --key-hash 4afb0d4da44350852482f423fe72f5599c1072c6970dc05a151dfeab3efc3d9d
$ conex_root sign --id rootA

# targets (maintainer) - repeat for m2 and m3
$ conex_targets create --id m1
# collect targets
$ conex_targets compute --id m1 # --pkg foo
# sign targets
$ conex_targets sign --id m1
```

Testing of `conex_verify`:

```
$ conex_verify_mirage_crypto -v --dir `pwd` -t sha256=3aedc7043e771efc42a3c3e6b60fa6baeb2c26c2d57f994c632bae98a09af701
$ conex_verify_openssl -v --dir `pwd` -t sha256=3aedc7043e771efc42a3c3e6b60fa6baeb2c26c2d57f994c632bae98a09af701
```

## Snapshot service

```
$ conex_key --id snap
$ conex_root add-to-role --role snapshot --id snap --key-hash 330a88f5fcc03dc47f5c7ddadbc8120f73b0ed2a81cfed93a18af93ab1c50253
$ conex_root sign --id rootA
$ conex_snapshot create --id snap
# have to put the key into snap
$ conex_snapshot sign --id snap
```

## Timestamp service

```
$ conex_key --id timestamp
$ conex_root add-to-role --role timestamp --id timestamp --key-hash 66013a5e2b2fe48119423034a1d4aab0a53152f231ede192093d329d25c4916e
$ conex_root sign --id rootA
$ conex_timestamp create --id timestamp
# have to put the key into timestamp
$ conex_timestamp sign --id timestamp
```
