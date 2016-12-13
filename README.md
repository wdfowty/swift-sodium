# Sodium [![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

Swift-Sodium
============

Swift-Sodium provides a safe and easy to use interface to perform
common cryptographic operations on iOS and OSX.

It leverages the [Sodium](https://download.libsodium.org/doc/) library,
and although Swift is the primary target, the framework can also be used in
Objective C.

Usage
=====

Add `Sodium.framework` as a dependency to your project, and import the module:
```swift
import Sodium
```

The Sodium library itself doesn't have to be installed on the system: the
repository already includes a precompiled library for armv7, armv7s,
arm64, as well as for the iOS simulator.

It targets Swift 3, introduced in Xcode 8.0.

The `libsodium-ios.a` file has been generated by the
[dist-build/ios.sh](https://github.com/jedisct1/libsodium/blob/master/dist-build/ios.sh)
script.
The `libsodium-osx.a` file has been generated by the
[dist-build/osx.sh](https://github.com/jedisct1/libsodium/blob/master/dist-build/osx.sh)
script.

Running these scripts on Xcode 8.0 (`8A218a`) on the revision
`6e2b119d86fe17675456ac1eb50b752253158f67` of libsodium generates files
identical to the ones present in this repository.

Public-key Cryptography
=======================

Authenticated Encryption
------------------------

```swift
let sodium = Sodium()!
let aliceKeyPair = sodium.box.keyPair()!
let bobKeyPair = sodium.box.keyPair()!
let message = "My Test Message".data(using:.utf8)!

let encryptedMessageFromAliceToBob: NSData =
    sodium.box.seal(message: message as NSData,
                    recipientPublicKey: bobKeyPair.publicKey,
                    senderSecretKey: aliceKeyPair.secretKey)!

let messageVerifiedAndDecryptedByBob =
    sodium.box.open(nonceAndAuthenticatedCipherText: encryptedMessageFromAliceToBob,
                    senderPublicKey: aliceKeyPair.publicKey,
                    recipientSecretKey: bobKeyPair.secretKey)
```

`seal()` automatically generates a nonce and prepends it to the
ciphertext. `open()` extracts the nonce and decrypts the ciphertext.

The `Box` class also provides alternative functions and parameters to
deterministically generate key pairs, to retrieve the nonce and/or the
authenticator, and to detach them from the original message.

Anonymous Encryption (Sealed Boxes)
-----------------------------------

```swift
let sodium = Sodium()!
let bobKeyPair = sodium.box.keyPair()!
let message = "My Test Message".data(using:.utf8)!

let encryptedMessageToBob =
    sodium.box.seal(message: message as NSData, recipientPublicKey: bobKeyPair.publicKey)!

let messageDecryptedByBob =
    sodium.box.open(anonymousCipherText: encryptedMessageToBob,
                    recipientPublicKey: bobKeyPair.publicKey,
                    recipientSecretKey: bobKeyPair.secretKey)
```

`seal()` generates an ephemeral keypair, uses the ephemeral secret
key in the encryption process, combines the ephemeral public key with
the ciphertext, then destroys the keypair. The sender can not decrypt
the resulting ciphertext. `open()` extracts the public key and decrypts
using the recipient's secret key. Message integrity is verified, but
the sender's identity cannot be correlated to the ciphertext.

Public-key signatures
=====================

Detached signatures
-------------------

```swift
let sodium = Sodium()!
let message = "My Test Message".data(using:.utf8)!
let keyPair = sodium.sign.keyPair()!
let signature = sodium.sign.signature(message: message as NSData, secretKey: keyPair.secretKey)!
if sodium.sign.verify(message: message as NSData,
                      publicKey: keyPair.publicKey,
                      signature: signature) {
    // signature is valid
}
```

Attached signatures
-------------------

```swift
let sodium = Sodium()!
let message = "My Test Message".data(using:.utf8)!
let keyPair = sodium.sign.keyPair()!
let signedMessage = sodium.sign.sign(message: message as NSData, secretKey: keyPair.secretKey)!
if let unsignedMessage = sodium.sign.open(signedMessage: signedMessage, publicKey: keyPair.publicKey) {
    // signature is valid
}
```

Secret-key authenticated encryption
===================================

```swift
let sodium = Sodium()!
let message = "My Test Message".data(using:.utf8)!
let secretKey = sodium.secretBox.key()!
let encrypted: NSData = sodium.secretBox.seal(message: message as NSData, secretKey: secretKey)!
if let decrypted = sodium.secretBox.open(nonceAndAuthenticatedCipherText: encrypted, secretKey: secretKey) {
    // authenticator is valid, decrypted contains the original message
}
```

Hashing
=======

Deterministic hashing
---------------------

```swift
let sodium = Sodium()!
let message = "My Test Message".data(using:.utf8)!
let h = sodium.genericHash.hash(message: message as NSData)
```

Keyed hashing
-------------

```swift
let sodium = Sodium()!
let message = "My Test Message".data(using:.utf8)!
let key = "Secret key".data(using:.utf8)!
let h = sodium.genericHash.hash(message: message as NSData, key: key as NSData)
```

Streaming
---------

```swift
let sodium = Sodium()!
let message1 = "My Test ".data(using:.utf8)!
let message2 = "Message".data(using:.utf8)!
let key = "Secret key".data(using:.utf8)!
let stream = sodium.genericHash.initStream(key: key as NSData)!
stream.update(input: message1 as NSData)
stream.update(input: message2 as NSData)
let h = stream.final()
```

Short-output hashing (SipHash)
==============================

```swift
let sodium = Sodium()!
let message = "My Test Message".data(using:.utf8)!
let key = sodium.randomBytes.buf(length: sodium.shortHash.KeyBytes)!
let h = sodium.shortHash.hash(message: message as NSData, key: key as NSData)
```

Random numbers generation
=========================

```swift
let sodium = Sodium()!
let randomData = sodium.randomBytes.buf(length: 1000)
```

Password hashing
================

Using Argon2i:

```swift
let sodium = Sodium()!
let password = "Correct Horse Battery Staple".data(using:.utf8)!
let hashedStr = sodium.pwHash.str(passwd: password as NSData,
                                  opsLimit: sodium.pwHash.OpsLimitInteractive,
                                  memLimit: sodium.pwHash.MemLimitInteractive)!

if sodium.pwHash.strVerify(hash: hashedStr, passwd: password as NSData) {
    // Password matches the given hash string
} else {
    // Password doesn't match the given hash string
}
```

Utilities
=========

Zeroing memory
--------------

```swift
let sodium = Sodium()!
var dataToZero: Data
sodium.utils.zero(data: dataToZero)
```

Constant-time comparison
------------------------

```swift
let sodium = Sodium()!
let secret1: Data
let secret2: Data
let equality = sodium.utils.equals(secret1, secret2)
```

Constant-time hexadecimal encoding
----------------------------------

```swift
let sodium = Sodium()!
let data: Data
let hex = sodium.utils.bin2hex(bin: data)
```

Hexadecimal decoding
--------------------

```swift
let sodium = Sodium()!
let data1 = sodium.utils.hex2bin(hex: "deadbeef")
let data2 = sodium.utils.hex2bin(hex: "de:ad be:ef", ignore: " :")
```
