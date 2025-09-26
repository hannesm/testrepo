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
# manually modify root (valid: rootA; maintainer role; keys: root key)
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
