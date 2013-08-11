# PKCS#11

This is a Go implementation of the PKCS#11 API. It wraps the library closely, but uses Go idiom
were it makes sense.

It is *assumed*, that:

* Go's uint size == PKCS11's CK_ULONG size
* CK_ULONG never overflows an Go uint

## SoftHSM

* Make it use a custom configuration file

        export SOFTHSM_CONF=$PWD/softhsm.conf

* Then use `softhsm` to init it

        softhsm --init-token --slot 0 --label test --pin 1234

## Examples

The following examples are available:

* sign/ directory contains a program, that generates a keypair and then signs
    some data;
* sha1sum/ directory contains a program that hashes a string with SHA1.

A skeleton program would look somewhat like this (yes, pkcs#11 is verbose):

    p := pkcs11.New("/usr/lib/softhsm/libsofthsm.so")
    p.Initialize()
    defer p.Destroy()
    defer p.Finalize()
    slots, _ := p.GetSlotList(true)
    session, _ := p.OpenSession(slots[0], pkcs11.CKF_SERIAL_SESSION|pkcs11.CKF_RW_SESSION)
    defer p.CloseSession(session)
    p.Login(session, pkcs11.CKU_USER, "1234")
    defer p.Logout(session)
    p.DigestInit(session, []*pkcs11.Mechanism{pkcs11.NewMechanism(pkcs11.CKM_SHA_1, nil)})
    hash, err := p.Digest(session, []byte("this is a string"))
    for _, d := range hash {
            fmt.Printf("%x", d)
    }
    fmt.Println()

## TODO

* Implement the whole spec
* sha1sum like implementation
