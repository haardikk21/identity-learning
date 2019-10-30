# Linked Data Signatures (Last Updated: 31 October 2019)

### [W3C Spec](https://w3c-dvcg.github.io/ld-signatures/)

## Overview

A linked data signature typically includes the following attributes:

```json
{
  "type": "(REQUIRED) A URI identifying the signature suite used to create the signature. Eg. Ed25519Signature2018",
  "created": "(REQUIRED) String value of combined date and time string",
  "domain": "A string specifying the restricted domain of the signature",
  "nonce": "(RECOMMENDED) String value included in the signature and MUST be only used once for a particular domain and window of time",
  "proof or proofChain": "(REQUIRED) One of any valid representations of signature value. Eg. JWS"
}
```

Example:

Simple LD Document

```json
{
  "@context": "https://www.w3.org/2018/credentials/examples/v1",
  "title": "Hello World!"
}
```

Simple Signed LD Document

```json
{
  "@context": "https://www.w3.org/2018/credentials/examples/v1",
  "title": "Hello World!",
  "proof": {
    "type": "Ed25519Signature2018",
    "proofPurpose": "assertionMethod",
    "created": "2019-08-23T20:21:34Z",
    "verificationMethod": "did:example:123456#key1",
    "domain": "example.org",
    "jws": "eyJ0eXAiOiJK...gFWFOEjXk"
  }
}
```

## Multiple Signatures

LD Signatures Spec supports multiple signatures in a single document. There are two approaches: **Signature Sets** and **Signature Chains**

### Signature Sets

Useful when the same data needs to be signed by multiple entities. Kind of like multisig transactions. But the order of signatures does not matter. A signature set is represented by an array of signatures in the `proof` key of the document.

Example:

```json
{
  "@context": "https://www.w3.org/2018/credentials/examples/v1",
  "title": "Hello World!",
  "proof": [
    {
      "type": "Ed25519Signature2018",
      "proofPurpose": "assertionMethod",
      "created": "2019-08-23T20:21:34Z",
      "verificationMethod": "did:example:123456#key1",
      "domain": "example.org",
      "jws": "eyJ0eXAiOiJK...gFWFOEjXk"
    },
    {
      "type": "RsaSignature2018",
      "proofPurpose": "assertionMethod",
      "created": "2017-09-23T20:21:34Z",
      "verificationMethod": "https://example.com/i/pat/keys/5",
      "domain": "example.org",
      "jws": "eyJ0eXAiOiJK...gFWFOEjXk"
    }
  ]
}
```

### Signature Chains

Useful when the same data needs to be signed by multiple entities and the order of signatures matters. Eg. when a document approval process moves up the chain, it needs to follow a specific order. A signature chain is represented by an ordered array of signatures in the `proofChain` key of the document

```json
{
  "@context": "https://www.w3.org/2018/credentials/examples/v1",
  "title": "Hello World!",
  "proofChain": [
    {
      "type": "Ed25519Signature2018",
      "proofPurpose": "assertionMethod",
      "created": "2019-08-23T20:21:34Z",
      "verificationMethod": "did:example:123456#key1",
      "domain": "example.org",
      "jws": "eyJ0eXAiOiJK...gFWFOEjXk"
    },
    {
      "type": "RsaSignature2018",
      "proofPurpose": "assertionMethod",
      "created": "2017-09-23T20:21:34Z",
      "verificationMethod": "https://example.com/i/pat/keys/5",
      "domain": "example.org",
      "jws": "eyJ0eXAiOiJK...gFWFOEjXk"
    }
  ]
}
```

## Signature Suites

A LD signature is designed to be easy to use by developers and aims to minimize the information one has to remember to generate a signature. Most times, just the signature suite name (Eg. `Ed25519Signature2018`) is required.

The signature suite **must** have at least the following attributes

```json
{
  "id": "A URL identifying the signature suite. Eg. https://web-payments.org/contexts/security-v1.jsonld#Ed25519Signature2018",
  "type": "?",
  "canonicalizationAlgorithm": "A URL that identifies the canonicalization algorithm to be used on the document. A canonicalization algorithm is something that ensures data is represented in a deterministic way, in structures where different representations are possible. Eg. in JSON",
  "digestAlgorithm": "A URL that identifies the message digest algorithm to use on the canonicalized document. A message digest algorithm is essentially a hashing function. Eg. https://www.ietf.org/assignments/jwa-parameters#SHA256",
  "signatureAlgorithm": "A URL that identifies the signature algorithm to use on the data to be signed. Eg. http://w3id.org/security#ed25519"
}
```

Example:

```json
{
  "id": "https://w3id.org/security/v1#Ed25519Signature2018",
  "type": "Ed25519VerificationKey2018",
  "canonicalizationAlgorithm": "https://w3id.org/security#URDNA2015",
  "digestAlgorithm": "https://www.ietf.org/assignments/jwa-parameters#SHA256",
  "signatureAlgorithm": "http://w3id.org/security#ed25519"
}
```

## Algorithms

The following algorithms are basically pseudocode as to how to achieve the intended outcomes.

### Signature Algorithm

```python
    def signature_algorithm(ld_document, signature_options, privateKey):
        output = read_or_generate(ld_document)

        canonicalized_document = canonicalize(document)
        to_be_signed = create_verify_hash(canonicalized_document, canonicalize_algorithm, message_digest_algorithm, signature_options)

        proof = digital_sign(tbs, signature_options.privateKey, digital_signature_algorithm)

        output.proof = proof
        return output
```

### Signature Verification Algorithm

```python
    def signature_verification_algorithm(signed_document):
        public_key = dereference_and_verify(signed_document.proof.verificationMethod)

        document = signed_document
        signature = document.proof
        delete document.proof

        canonicalized_document = canonicalize(document)
        tbv = create_verify_hash(canonicalized_document,
        canonicalize_algorithm, message_digest_algorithm, signature)

        return signature_verify(signature, tbv, public_key)
```

### Create Verify Hash Algorithm

```python
    def create_verify_hash_algorithm(canonicalized_document, canonicalization_algorithm, message_digest_algorithm, signature_options):
        options = signature_options
        delete options.type
        delete options.id
        delete options.proof

        if (!options.created)
            options.created = date+time

        canonicalized_options_document = canonicalize(options)
        hashed_canonicalized_options_document = message_digest(canonicalized_options_document)
        hashed_canonicalized_document = message_digest(canonicalized_document)

        output = hashed_canonicalized_options_document + hashed_canonicalized_document

        return output
```
