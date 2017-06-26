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

```
opam repo add conex-test https://github.com/hannesm/testrepo.git 2 p3CRLizMuEu6j1rwwv2gt8s3nsrefJjPKYY8vzqJnjU= iTX++W2wO2SobN8nN+TobSbZl2JRS4Z+v4RSdP5MrlY=
```

Vary the quorum or the fingerprints to see verification failures.

## Private keys

For janitor `a`, janitor `b`, and author `c` are help for test purposes in
`priv`.  Do not use these private keys elsewhere, generate your own instead.

If you clone this repository to `/tmp/testrepo` and `cp priv/* ~/.conex/`,
you'll be able to sign updates.  If you choose another directory, rename the
private key filenames accordingly.

## Creation of this repository

Initial setup:

```
$ mkdir -p /tmp/testrepo/packages/foo/foo.0.1.0
$ echo > /tmp/testrepo/packages/foo/foo.0.1.0/opam
# init some ids
$ conex_author init --id a --repo /tmp/testrepo
$ conex_author init --id b --repo /tmp/testrepo
$ conex_author init --id c --repo /tmp/testrepo

# janitor and authorisation
$ conex_author team janitors -m a -m b --id a
$ conex_author authorisation foo -m c --id b

# approve team + auth
$ conex_author team janitors --id b
$ conex_author authorisation foo --id a

# approve keys
$ conex_author key all --id a
$ conex_author key all --id b

# release
$ conex_author release foo --id c

# sign
$ conex_author sign --id a
$ conex_author sign --id b
$ conex_author sign --id c
```

Testing of `conex_verify`:

```
$ conex_author info --id a --quorum 2
author a #2 (created 1487376412) verified 8 resources, 0 queued
4096 bit RSA key created 1487374420 approved, SHA256: p3CRLizMuEu6j1rwwv2gt8s3nsrefJjPKYY8vzqJnjU=
account GitHub a approved
team janitors (2 members) approved
$ conex_author info --id b --quorum 2
author b #2 (created 1487376419) verified 8 resources, 0 queued
4096 bit RSA key created 1487374483 approved, SHA256: iTX++W2wO2SobN8nN+TobSbZl2JRS4Z+v4RSdP5MrlY=
account GitHub b approved
team janitors (2 members) approved
$ conex_verify_nocrypto --dir /tmp/testrepo/ --quorum 2 -t p3CRLizMuEu6j1rwwv2gt8s3nsrefJjPKYY8vzqJnjU= -t iTX++W2wO2SobN8nN+TobSbZl2JRS4Z+v4RSdP5MrlY=
verification of 1 packages successfull with 0 warnings
$ conex_verify_openssl --dir /tmp/testrepo/ --quorum 2 -t p3CRLizMuEu6j1rwwv2gt8s3nsrefJjPKYY8vzqJnjU= -t iTX++W2wO2SobN8nN+TobSbZl2JRS4Z+v4RSdP5MrlY=
verification of 1 packages successfull with 0 warnings
$ conex_author verify -r /tmp/testrepo/ --quorum 2
verified 1 packages, 0 warnings

//bad fp (removed =)
$ conex_verify_nocrypto --dir /tmp/testrepo/ --quorum 2 -t p3CRLizMuEu6j1rwwv2gt8s3nsrefJjPKYY8vzqJnjU= -t iTX++W2wO2SobN8nN+TobSbZl2JRS4Z+v4RSdP5MrlY
conex_verify_nocrypto: quorum for team janitors insufficient: 0/2 empty

$ mkdir /tmp/testrepo/packages/bar
$ conex_verify_nocrypto --dir /tmp/testrepo/ --quorum 2 -t p3CRLizMuEu6j1rwwv2gt8s3nsrefJjPKYY8vzqJnjU= -t iTX++W2wO2SobN8nN+TobSbZl2JRS4Z+v4RSdP5MrlY=
conex_verify_nocrypto: authorisation bar was not found in repository
$ conex_verify_nocrypto --dir /tmp/testrepo/ --quorum 2 -t p3CRLizMuEu6j1rwwv2gt8s3nsrefJjPKYY8vzqJnjU= -t iTX++W2wO2SobN8nN+TobSbZl2JRS4Z+v4RSdP5MrlY= --nostrict
verification of 1 packages successfull with 1 warnings
```
