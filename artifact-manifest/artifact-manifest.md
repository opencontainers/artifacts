# OCI Artifact Manifest

The OCI artifact manifest provides a means to define a wide range of artifacts, including a chain of dependencies of related artifacts. It provides a means to define a collection of items, including blobs and linked artifacts with the information needed for reference counting, garbage collection and indexing. 

## Goals of Artifact Manifest

### Bi-directional Hierarchal Support

The OCI Distribution-spec 1.0 supports tops down hierarchies for tracking a manifest, a config object, and a collection of layers. This works well for immutable artifacts that are individually pushed to a registry. As an artifact is pushed, a manifest and digest of the manifest and it's referenced layers are computed.

The artifact-manifest will support additional references to existing content within a registry. Rather than update the original manifest with the additional objects, the additional object will provide the digest by which it's enhancing.

- `hello-world:v1` with a digest of `sha256:8b895ffec9fe33301cee47b0edc1600ea67604c3d138fa8ecb43ae62ad3b6fd4` is pushed to the registry
- A **S**oftware **B**ill **o**f **M**aterials (SBoM) for `hello-world:v1` is pushed to the registry. 
- A `hello-world:v1` SBoM is pushed, using digest of the `hello-world:v1` image.

In the above case, the original `hello-world:v1` image digest remains the same. Deployments of the `hello-world:v1` image can be made using the `hello-world:v1` tag, or the `sha256:8b89...` digest. With additional distribution-spec artifact APIs, requests may be made to list objects that reference the `hello-world:v1` artifact. In this case, returning the SBoM references.

### Artifact Copying Within and Across OCI Compliant Registries

Distribution-spec APIs will provide a means to discover, pull and push content within and across registries. No knowledge of the specific artifact type will be necessary.

### Delete Operations

Distribution-spec APIs will provide standard delete operations, including options for deleting referenced artifacts, or blocking a delete as the artifact is referenced by other artifacts.

- Which references should be deleted (ref counted)
- Which references should just reduce ref counting?
- Which artifacts should be blocked from deletion as another artifact depends upon it?
- Examples:
  - deleting the wordpress helm chart deletes the config, chart and values blobs
  - deleting the mysql image should warn if referenced by helm charts
  - deleting the wordpress chart removes a ref count to the mysql image, for mysql deletion

## *Image Manifest* Property Descriptions

- **`schemaVersion`** *int*

  This REQUIRED property specifies the artifact manifest schema version.
  For this version of the specification, this MUST be `1`. The value of this MAY change if the schema is enhanced.

- **`mediaType`** *string*

  This property identifies the OCI Artifact Manifest schema. This field MUST be `"application/vnd.oci.artifact.manifest.v1+json"`

- **`config`** *[descriptor](descriptor.md)*

    This REQUIRED property references a configuration object for a container, by digest.
    Beyond the [descriptor requirements](descriptor.md#properties), the value has the following additional restrictions:

    - **`mediaType`** *string*

        This [descriptor property](descriptor.md#properties) has additional restrictions for `config`.
        Implementations MUST support at least the following media types:

        - [`application/vnd.oci.image.config.v1+json`](config.md)

        Manifests concerned with portability SHOULD use one of the above media types.

- **`layers`** *array of objects*

    Each item in the array MUST be a [descriptor](descriptor.md).
    The array MUST have the base layer at index 0.
    Subsequent layers MUST then follow in stack order (i.e. from `layers[0]` to `layers[len(layers)-1]`).
    The final filesystem layout MUST match the result of [applying](layer.md#applying-changesets) the layers to an empty directory.
    The [ownership, mode, and other attributes](layer.md#file-attributes) of the initial empty directory are unspecified.

    Beyond the [descriptor requirements](descriptor.md#properties), the value has the following additional restrictions:

    - **`mediaType`** *string*

        This [descriptor property](descriptor.md#properties) has additional restrictions for `layers[]`.
        Implementations MUST support at least the following media types:

        - [`application/vnd.oci.image.layer.v1.tar`](layer.md)
        - [`application/vnd.oci.image.layer.v1.tar+gzip`](layer.md#gzip-media-types)
        - [`application/vnd.oci.image.layer.nondistributable.v1.tar`](layer.md#non-distributable-layers)
        - [`application/vnd.oci.image.layer.nondistributable.v1.tar+gzip`](layer.md#gzip-media-types)

        Manifests concerned with portability SHOULD use one of the above media types.
        An encountered `mediaType` that is unknown to the implementation MUST be ignored.


        Entries in this field will frequently use the `+gzip` types.

- **`annotations`** *string-string map*

    This OPTIONAL property contains arbitrary metadata for the image manifest.
    This OPTIONAL property MUST use the [annotation rules](annotations.md#rules).

    See [Pre-Defined Annotation Keys](annotations.md#pre-defined-annotation-keys).

## Example Image Manifest

*Example showing an image manifest:*
```json,title=Manifest&mediatype=application/vnd.oci.image.manifest.v1%2Bjson
{
  "schemaVersion": 2,
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "size": 7023,
    "digest": "sha256:b5b2b2c507a0944348e0303114d8d93aaaa081732b86451d9bce1f432a537bc7"
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 32654,
      "digest": "sha256:9834876dcfb05cb167a5c24953eba58c4ac89b1adf57f28f2f9d09af107ee8f0"
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 16724,
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b"
    },
    {
      "mediaType": "application/vnd.oci.artifact.layer.v1.tar+gzip",
      "size": 73109,
      "digest": "sha256:ec4b8955958665577945c89419d1af06b5f7636b4ac3da7f12184802ad867736"
    }
  ],
  "annotations": {
    "com.example.key1": "value1",
    "com.example.key2": "value2"
  }
}
```

## Pre-Defined Annotation Keys

This specification defines the following annotation keys, intended for but not limited to  Artifact Manifest authors:
* **org.opencontainers.artifact.created** date and time on which the artifact was built (string, date-time as defined by [RFC 3339](https://tools.ietf.org/html/rfc3339#section-5.6)).
* **org.opencontainers.artifact.authors** contact details of the people or organization responsible for the artifact (freeform string)
* **org.opencontainers.artifact.url** URL to find more information on the artifact (string)
* **org.opencontainers.artifact.documentation** URL to get documentation on the artifact (string)
* **org.opencontainers.artifact.source** URL to get source code for building the artifact (string)
* **org.opencontainers.artifact.version** version of the packaged software
  * The version MAY match a label or tag in the source code repository
  * version MAY be [Semantic versioning-compatible](http://semver.org/)
* **org.opencontainers.artifact.revision** Source control revision identifier for the packaged software.
* **org.opencontainers.artifact.vendor** Name of the distributing entity, organization or individual.
* **org.opencontainers.artifact.licenses** License(s) under which contained software is distributed as an [SPDX License Expression][spdx-license-expression].
* **org.opencontainers.artifact.title** Human-readable title of the artifact (string)
* **org.opencontainers.artifact.description** Human-readable description of the software packaged in the artifact (string)

## Setting meta-data

Should be as simple as setting a name/value pair for a specific tag and/or digest
Setting a name/value pair for a tag will assign the meta-data to the digest currently associated with the tag. We do not currently see the need to set meta-data specific to a tag.

Setting the git digest to a tagged artifact:

```shell
/charts/wordpress:5.7
{
  "name": "git.digest",
  "value": "1124125"
}
```

Setting the contact info to a tagged artifact:

```shell
/charts/wordpress:5.7
{
  "name": "oci.meta-data.contact",
  "value": '{
    "first": "Steve",
    "last": "Lasker",
    "email" "stevenlasker@hotmail.com"
    }'
}
```

## Collections

## Parent

Parent elements MUST NOT have tags as they are attributions to the parent element

Optional elements are optional as they represent metadata that has persistance.

### Blobs

All blobs are considered to be hard dependencies. These support ref counting, but would be deleted when the manifest is deleted.

### Dependencies

All dependencies are considered soft dependencies.


## Pushing Artifact Manifests

Manifest validation
Each mediaType is evaluated. If the manifestType is 

## Open Questions

Should the references collection support additional types, like loose urls