# OCI Artifact Manifest Spec (Phase-1 Reference Types)
The OCI artifact manifest generalizes the use cases of [OCI image manifest][oci-image-manifest-spec] by removing constraints defined on the image-manifest such as a required `config` object and required & ordinal `layers`. It then adds a `subjectManifest` property supporting reference types. The addition of a new manifest does not change, nor impact the `image.manifest`. It provides a means to define a wide range of artifacts, including a chain of related artifacts enabling SBoMs, on-demand loading, signatures and metadata that can be related to an `image.manifest` or `image.index`. By defining a new manifest, registries and clients opt-into new capabilities, without breaking existing registry and client behavior or setting expectations for scenarios to function when the client and/or registry doesn't yet implement the new capabilities.

To enable a fall 2021 focus on supply chain security,  **Phase 1** will narrowly focus on Reference Type support, giving time for further generalization with less time constraints.

For usage and scenarios, see [artifact-manifest.md](./artifact-manifest.md)

## Example OCI Artifact Manifests

The following are Phase 1 examples:

- [`net-monitor:v1` oci container image](./artifact-manifest/net-monitor-oci-image.json)
- [`net-monitor:v1` notary v2 signature](./artifact-manifest/net-monitor-image-signature.json)
- [`net-monitor:v1` sample sbom](./artifact-manifest/net-monitor-image-sbom.json)
- [`net-monitor:v1` nydus image with on-demand loading](./artifact-manifest/net-monitor-image-nydus-ondemand-loading.json)

## OCI Artifact Manifest Properties

For **Phase 1**, an artifact manifest provides an optional collection of blobs and a reference to the manifest of another artifact.

- **`schemaVersion`** *int*

  This REQUIRED property specifies the artifact manifest schema version.
  For this version of the specification, this MUST be `3`. The value of this field WILL change as the manifest schema evolves. Minor version changes to the `oci.artifact.manifest` spec MUST be additive, while major version changes MAY be breaking. Artifact clients MUST implement version checking to allow for future, yet unknown changes. Artifact clients MUST ignore additive properties to minor versions. Artifact clients MAY support major changes, with no guarantee major changes MAY impose breaking changing behaviors. Artifact authors MAY support new and older schemaVersions to provide the best user experience.

- **`mediaType`** *string*

  This field contains the `mediaType` of this document, differentiating from [image-manifest][oci-image-manifest-spec] and [oci-image-index]. The mediaType for this manifest type MUST be `application/vnd.oci.artifact.manifest.v1+json`, where the version WILL change to reflect newer versions. Artifact authors SHOULD support multiple `mediaType` versions to provide the best user experience for their artifact type.

- **`artifactType`** *string*

  Phase 1 of the OCI Artifact spec will support reference types to existing [OCI Artifacts][oci-artifacts]. The REQUIRED `artifactType` is unique value, as registered with iana.org. See [registering unique types.][registering-iana]. The `artifactType` is equivalent to OCI Artifacts that used the `manifest.config.mediaType` to differentiate the type of artifact. Artifact authors that implement `oci.artifact.manifest` use `artifactType` to differentiate the type of artifact. example:(`example.sbom` from `cncf.notary`).

- **`blobs`** *array of objects*

    An OPTIONAL collection of 0 or more blobs. The blobs array is analogous to [oci.image.manifest layers][oci-image-manifest-spec-layers], however unlike [image-manifest][oci-image-manifest-spec], the ordering of blobs is specific to the artifact type. Some artifacts may choose an overlay of files, while other artifact types may store indepdent collections of files.

    - Each item in the array MUST be a [descriptor][descriptor], and MUST NOT refer to another `manifest` providing dependency closure.
    - The max number of blobs is not defined, but MAY be limited by [distribution-spec][oci-distribution-spec] implementations.
    - An encountered `descriptor.mediaType` that is unknown to the implementation MUST be ignored.

- **`subjectManifest`** *descriptor*

   An OPTIONAL reference to any existing manifest within the repository. When specified, the artifact is said to be dependent upon the referenced `subjectManifest`.
   - The item MUST be a [descriptor][descriptor] representing a manifest. Descriptors to blobs are not supported. The registry MUST return a `400` response code when `subjectManifest` is not found in the same repository, and not a manifest.

- **`annotations`** *string-string map*

    This OPTIONAL property contains arbitrary metadata for the image manifest.
    This OPTIONAL property MUST use the [annotation rules](annotations.md#rules).

    See [Pre-Defined Annotation Keys][annotations]

## Push Validation

Following the [distribution-spec push api](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#push), all `blobs` *and* the `subjectManifest` descriptors SHOULD exist when pushed to a distribution instance.

## Lifecycle Management

For Phase 1, artifact types will be limited to reference types. A reference type is an artifact that doesn't have a lifecycle unto itself. A container image is said to have an independent lifecycle. A reference type, such as an SBoM or signature have a lifecycle tied to the `subjectManifest`. When the `subjectManifest` is deleted or marked for garbage collection, the defined artifact is subject to deletion as well. A distribution instance SHOULD delete, (refCount -1) the artifact when the `subjectManifest` is deleted.

### Tagged `referenceTypes`

As signatures and SBoMs are not considered independent artifact types, they SHOULD NOT have a tag, simplifying the lifecycle management. As the `subjectManifest` is marked for deletion (refCount=0), the `referenctType` is also marked for deletion (refCount -1). However, these artifacts MAY have tags as future versions of the artifact manifest MAY support independent types. 

[oci-artifacts]:                   https://github.com/opencontainers/artifacts
[oci-config]:                      https://github.com/opencontainers/image-spec/blob/master/config.md
[oci-image-manifest-spec]:         https://github.com/opencontainers/image-spec/blob/master/manifest.md
[oci-image-manifest-spec-layers]:  https://github.com/opencontainers/image-spec/blob/master/manifest.md#image-manifest-property-descriptions
[oci-image-index]:                 https://github.com/opencontainers/image-spec/blob/master/image-index.md
[oci-distribution-spec]:           https://github.com/opencontainers/distribution-spec
[media-type]:                      https://github.com/opencontainers/image-spec/blob/master/media-types.md
[artifact-type]:                   https://github.com/opencontainers/artifacts/blob/master/artifact-authors.md#defining-a-unique-artifact-type
[registering-iana]:                ./artifact-authors.md#registering-unique-types-with-iana
[descriptor]:                      https://github.com/opencontainers/image-spec/blob/master/descriptor.md
[annotations]:                     https://github.com/opencontainers/image-spec/blob/master/annotations.md