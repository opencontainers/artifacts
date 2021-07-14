To clarify the current high-level differences with the artifact-manifest and the existing image-manifest:

| Image-spec | Artifacts Spec |
|-|-|
| `config` REQUIRED | `config` optional as it's just another entry in the `blobs` collection with a `config mediaType` |
| `layers` REQUIRED | `blobs`, which renamed `layers` to reflect general usage are OPTIONAL |
| `layers` ORDINAL | `blobs` are defined by the specific artifact spec. Helm isn't ordinal, while other artifact types, like container images MAY make them ordinal |
| `manifest.config.mediaType` used to uniquely identify different artifact types. | `manifest.artifactType` added to lift the workaround for using `manifest.config.mediaType` on a REQUIRED, but not always used property. |
| | `subjectManifest` OPTIONAL, enabling an artifact to extend another artifact (SBOM, Signatures, Nydus, Scan Results, )
| | `/referrers` api for discovering referenced artifacts, with the ability to filter by `artifactType` |
| | Lifecycle management defined, starting to provide standard expectations for how users can manage their content. It doesn't define GC as an internal detail|

The artifact manifest approach to reference types is based on a new manifest, enabling registries and clients to opt-into the behavior, with clear and consistent expectations, rather than slipping new content into a registry, or client, that may, or may not know how to lifecycle manage the new content. See [Discussion of a new manifest #41](https://github.com/opencontainers/artifacts/discussions/41)
