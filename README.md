# conex test-repository

To test `conex_verify`, you need to add the following to your `$(OPAMROOT)/config`:

```
repository-validation-command: [
  "conex_verify_nocrypto" "--quorum" "%{quorum}%"
  "--trust-anchors" "%{anchors}%"
  "--repo" "%{repo}%"
  "--dir=%{dir}%" { ! incremental }
  "--patch=%{patch}%" { incremental }
  "--incremental" { incremental }
]
```

You can use `conex_verify_openssl` instead of `nocrypto`, add `-v` flags.  This
will only be used for repositories which have trust anchors and a quorum
configured.  Update of the default `opam-repository` will not be affected.

To add this repository with a quorum of 2, you have to type:

TODO
```
opam repo add conex-test https://github.com/hannesm/testrepo.git 2 p3CRLizMuEu6j1rwwv2gt8s3nsrefJjPKYY8vzqJnjU= iTX++W2wO2SobN8nN+TobSbZl2JRS4Z+v4RSdP5MrlY=
```

Vary the quorum or the fingerprints to see verification failures.

## Private keys

TODO For janitor `a`, janitor `b`, and author `c` are help for test purposes in
`priv`.  Do not use these private keys elsewhere, generate your own instead.

If you clone this repository and `cp priv/* ~/.conex/`, you'll be able to sign
updates.

## Creation of this repository

Initial setup:

```
$ mkdir -p /tmp/testrepo/packages/foo/foo.0.1.0
$ echo > /tmp/testrepo/packages/foo/foo.0.1.0/opam
# init some keys
$ conex_key --id rootA
$ conex_key --id j1

# root
$ conex_root edit
# manually modify root (roles: root / janitor; keys: root key)
$ conex_root sign --id rootA

# targets (janitor)
$ conex_targets create --id j1
# collect targets
$ conex_targets compute --pkg foo
# sign targets
$ conex_targets sign --id j1
```

Testing of `conex_verify`:

```
$ conex_verify_nocrypto -v --dir `pwd` -t sha256=5a148d3977cb03dbeaeb99fa7033b5c7a43c8c7ee1114fee0c22fada2f7c9687
$ conex_verify_openssl -v --dir `pwd` -t sha256=5a148d3977cb03dbeaeb99fa7033b5c7a43c8c7ee1114fee0c22fada2f7c9687
```
