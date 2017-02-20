# conex test-repository

To test `conex_verify`, you need to add the following to your `$(OPAMROOT)/config`:

```
repository-validation-command: [
  "conex_verify_nocrypto" "--quorum" "%{quorum}%"
  "--trust-anchors" "%{anchors}%"
  "--repo" "%{repo}%"
  "--dir" "%{dir}%"
  "%{incremental:--patch}%" "%{patch}%"
  "%{incremental:--incremental}%"
]
```

You can use `conex_verify_openssl` instead of `nocrypto`, add `-v` flags.  This
will only be used for repositories which have trust anchors and a quorum
configured.  Update of the default `opam-repository` will not be affected.

To add this repository with a quorum of 2, you have to type:

```
opam repo add conex-test https://github.com/hannesm/testrepo.git 2 e9814aa297418e29d195160c065eab53a1a80d943bd0f7c99a315160dc478a16 bd22323ba8d661158297504968e8b92cb313c16029684a968ffc390bdcbe591f
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
$ conex_author status --id a --quorum 2
author a #2 (created 1492129051) verified 8 resources, 0 queued
4096 bit RSA key created 1487374420 approved, SHA256: e9814aa297418e29d195160c065eab53a1a80d943bd0f7c99a315160dc478a16
account GitHub a approved
team janitors (2 members) approved
$ conex_author status --id b --quorum 2
author b #2 (created 1492129069) verified 8 resources, 0 queued
4096 bit RSA key created 1487374483 approved, SHA256: bd22323ba8d661158297504968e8b92cb313c16029684a968ffc390bdcbe591f
account GitHub b approved
team janitors (2 members) approved
$ conex_verify_nocrypto --dir /tmp/testrepo/ --quorum 2 -t e9814aa297418e29d195160c065eab53a1a80d943bd0f7c99a315160dc478a16 -t bd22323ba8d661158297504968e8b92cb313c16029684a968ffc390bdcbe591f
verification of 1 packages successfull with 0 warnings
$ conex_verify_openssl --quiet --dir /tmp/testrepo/ --quorum 2 -t e9814aa297418e29d195160c065eab53a1a80d943bd0f7c99a315160dc478a16 -t bd22323ba8d661158297504968e8b92cb313c16029684a968ffc390bdcbe591f
verification of 1 packages successfull with 0 warnings
$ conex_author verify -r /tmp/testrepo/ --quorum 2
verified 1 packages, 0 warnings
```

When passing a bad fingerprint as trust anchor (removed the some hex characters), conex_verify will complain:
```
$ conex_verify_nocrypto --dir /tmp/testrepo/ --quorum 2 -t e9814aa297418e29d195160c065eab53a1a80d943bd0f7c99a315160dc478a16 -t bd22323ba8d661158297504968e8b92cb313c16029684a968ffc390bdcbe59
conex_verify_nocrypto: quorum for team janitors insufficient: 0/2 empty
```

When an unsigned package is in the repository, conex_verify will complain (unless run in non-strict mode):
```
$ mkdir /tmp/testrepo/packages/bar
$ conex_verify_nocrypto --dir /tmp/testrepo/ --quorum 2 -t e9814aa297418e29d195160c065eab53a1a80d943bd0f7c99a315160dc478a16 -t bd22323ba8d661158297504968e8b92cb313c16029684a968ffc390bdcbe591f
conex_verify_nocrypto: authorisation bar was not found in repository
$ conex_verify_nocrypto --nostrict --dir /tmp/testrepo/ --quorum 2 -t e9814aa297418e29d195160c065eab53a1a80d943bd0f7c99a315160dc478a16 -t bd22323ba8d661158297504968e8b92cb313c16029684a968ffc390bdcbe591f
conex_verify_nocrypto: [WARNING] ignoring bar authorisation bar was not found in repository
verification of 1 packages successfull with 1 warnings
```
