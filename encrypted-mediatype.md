# Encrypted media types

To be able to protect the confidentiality of the data in layers, encryption of the layer data blobs can be done to prevent unauthorized access to layer data. Encryption is performed on the data blob of a by specifying a media type with the `+encrypted` suffix. For example, `application/vnd.oci.image.layer.v1.tar+encrypted` is an layer representation of an encrypted `application/vnd.oci.image.layer.v1.tar` layer. 

**Note:** Due to ongoing concerns of scalability of the mediatype suffix model, the mediatype definitions for encryption encoding are subject to change and should be treated as Work In Progress (WIP). Details of this discussion can tracked at this [issue](https://github.com/opencontainers/image-spec/issues/791).

## `+encrypted` media type and annotation definitions

Media types:
* The media type `application/vnd.oci.image.layer.v1.tar+encrypted` represents an `application/vnd.oci.image.layer.v1.tar` payload which has been [encrypted](#layer-encryption).
* The media type `application/vnd.oci.image.layer.v1.tar+gzip+encrypted` represents an `application/vnd.oci.image.layer.v1.tar+gzip` payload which has been [encrypted](#layer-encryption).
* The media type `application/vnd.oci.image.layer.nondistributable.v1.tar+encrypted` represents an `application/vnd.oci.image.layer.nondistributable.v1.tar` payload which has been [encrypted](#layer-encryption).
* The media type `application/vnd.oci.image.layer.nondistributable.v1.tar+gzip+encrypted` represents an `application/vnd.oci.image.layer.nondistributable.v1.tar+gzip` payload which has been [encrypted](#layer-encryption).

When using `+encrypted` media types, the data blobs are encrypted. In order to decrypt an image, the encryption metadata is required. In order to benefit from deduplication across multiple authorized recipients, the metadata is stored separately from the data blob. The encryption meatadata is stored in **org.opencontainers.image.enc** prefixed annotations. Details on the contents of the annotations are explained in the following section: [Encryption Metadata](#encryption-metadata).
- `org.opencontainers.image.enc.pubopts` - Contains public encryption parameters for the decryption of the image.
- `org.opencontainers.image.enc.keys.[protocol]` - Contains the private parameters which only authorized users of the image should be able to access. These parameters are encrypted using various encryption protocols. Examples of these protocols are as follows:
  - `org.opencontainers.image.enc.keys.pkcs7` - Contains an array of base64 comma separated encrypted messages (in accordance with [PKCS7(RFC2315)](https://tools.ietf.org/html/rfc2315)) that contain private encryption parameters.
  - `org.opencontainers.image.enc.keys.jwe` - Contains an array of base64 comma separated encrypted messages (in accordance with [JWE(RFC7516)](https://tools.ietf.org/html/rfc7516)) that contain private  encryption parameters.
  - `org.opencontainers.image.enc.keys.openpgp` - Contains an array of base64 comma separated encrypted messages (in accordance with [OpenPGP(RFC4880)](https://tools.ietf.org/html/rfc4880)) that contain private encryption parameters. 

## Encryption Metadata

The encryption metadata consists of 2 parts: the PublicLayerBlockCipherOptions and PrivateLayerBlockCipherOptions. The PublicLayerBlockCipherOptions contain encryption metadata that is public (i.e. cipher type, HMAC, etc.) and the PrivateLayerBlockCipherOptions contains the encryption metadata which should be confidential (i.e. symmetric key, nonce (optional), etc.). These are stored in the **org.opencontainers.image.enc** prefixed annotations.

Below are golang definitions of these JSON objects:

```golang
// LayerCipherType is the ciphertype as specified in the layer metadata
type LayerCipherType string
// PublicLayerBlockCipherOptions includes the information required to encrypt/decrypt
// an image which are public and can be deduplicated in plaintext across multiple
// recipients
type PublicLayerBlockCipherOptions struct {
    // CipherType denotes the cipher type according to the list of OCI suppported
    // cipher types.
    CipherType LayerCipherType `json:"cipher"`
    // Hmac contains the hmac string to help verify encryption
    Hmac []byte `json:"hmac"`
    // CipherOptions contains the cipher metadata used for encryption/decryption
    // This field should be populated by Encrypt/Decrypt calls
    CipherOptions map[string][]byte `json:"cipheroptions"`
}

// PrivateLayerBlockCipherOptions includes the information required to encrypt/decrypt
// an image which are sensitive and should not be in plaintext
type PrivateLayerBlockCipherOptions struct {
	// SymmetricKey represents the symmetric key used for encryption/decryption
	// This field should be populated by Encrypt/Decrypt calls
	SymmetricKey []byte `json:"symkey"`
	// Digest is the digest of the original data for verification.
	// This is NOT populated by Encrypt/Decrypt calls
	Digest digest.Digest `json:"digest"`
	// CipherOptions contains the cipher metadata used for encryption/decryption
	// This field should be populated by Encrypt/Decrypt calls
	CipherOptions map[string][]byte `json:"cipheroptions"`
}
```

Details of the algorithms and protocols used in the encryption of the data blob are defined in these JSON objects. Here are some examples of the Public/Private LayerBlockCipherOptions.
- The `cipher` field specifies the encryption algorithm to use according to the [list of cipher types supported](#cipher-types).
- The `symkey` field specifies the base64 encoded bytes of the symmetric key used to encrypt/decrypt the data blob.
- The `cipherOptions` field specifies additional parameters used in the decryption process of the specified algorithm. This should be in accordance with the RFC standard of the algorithm used.

Example of `PublicLayerBlockCipherOption`:
```json
{
    "cipher": "AES_256_CTR_HMAC_SHA256",
    "hmac": "M0M5OTA5QUZFQzI1MzU0RDU1MURBRTIxNTkwQkIyNkUzOEQ1M0YyMTczQjhEM0RDM0VFRTRDMDQ3RTdBQjFDMQ==",
    "cipheroptions": {}
}
```

Example of `PrivateLayerBlockCipherOption`:
```
{
    "symkey": "54kiln1USEaKnlYhKdz+aA==",
    "cipheroptions": {
        "nonce": "AdcRPTAEhXx6uwuYcOquNA==",
        ...
    }
}
```

The PublicLayerBlockCipherOptions JSON object is stored base64 encoded in the layer annotation **org.opencontainers.image.enc.pubopts**. 

The PrivateLayerBlockCipherOptions JSON object is not stored in plaintext due to the sensitive nature of the contents. Instead, the object goes through a cryptographic wrapping process. This ensures that only authorized parties are able to decrypt the layers, the decryption metadata objects are wrapped as encrypted messages to the authorized recipients in accordance with encrypted message standards such as [OpenPGP(RFC4880)](https://tools.ietf.org/html/rfc4880), [PKCS7(RFC2315)](https://tools.ietf.org/html/rfc2315), [JWE(RFC7516)](https://tools.ietf.org/html/rfc7516). 

The following annotations are used to communicate these encrypted messages:
- `org.opencontainers.image.enc.keys.pkcs7` - An array of base64 comma separated encrypted messages (with payload of json serialized PrivateLayerBlockCipherOptions) to perform decryption of the layer data in accordance with [PKCS7(RFC2315)](https://tools.ietf.org/html/rfc2315)
- `org.opencontainers.image.enc.keys.jwe` - An array of base64 comma separated encrypted messages (with payload of json serialized PrivateLayerBlockCipherOptions) to perform decryption of the layer data in accordance with [JWE(RFC7516)](https://tools.ietf.org/html/rfc7516)
- `org.opencontainers.image.enc.keys.openpgp` - An array of base64 comma separated encrypted messages (with payload of json serialized PrivateLayerBlockCipherOptions) to perform decryption of the layer data in accordance with [OpenPGP(RFC4880)](https://tools.ietf.org/html/rfc4880)
- `org.opencontainers.image.enc.keys.[protocol]` - An array of base64 comma separated encrypted messages (with payload of json serialized PrivateLayerBlockCipherOptions) to perform decryption of the layer data in accordance with an appropriate standard of specified protocol.

The decryption of the image can be performed by unwrapping the PrivateLayerBlockCipherOptions using the `org.opencontainers.image.enc.keys.[protocol]` annotations and using the appropriate cipher with the unwrapped PrivateLayerBlockCipherOptions and PublicLayerBlockCipherOptions to decrypt the layer data blob.

### Cipher Types

The current list of cipher types supported are:
- `AES_256_CTR_HMAC_SHA256` - Encryption with `AES_256_CTR` algorithm [FIPS-197](https://csrc.nist.gov/csrc/media/publications/fips/197/final/documents/fips-197.pdf)  with Encrypt-then-mac [RFC7366](https://tools.ietf.org/html/rfc7366). The protocols used in this cipher type is in accordance to FIPS 140-2 compliant standards. 
