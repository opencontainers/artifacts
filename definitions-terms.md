# Definitions and Terms

A collection of definitions and terms used within this repository.

* [Artifact](#artifact)
* [Artifact Author](#artifact-author)
* [Distribution Operator](#distribution-operator)
* [Media Type](#media-type)
* [OCI Image](#oci-image)
* [Registry](#registry)
* [Well Known Type](#well-known-type)
* [YASS](#yass)

## Artifact

An artifact is a piece of data that is being cohesively considered from a users perspective, meaning that the user shouldn't be concerned about the individual parts that he needs to get the "artifact" as long as he gets all the parts needed for the user to consider the artifact to be complete. What is the type of content of an artifact depends on the project implementing the artifacts specification. It's up to the implementations of the specification to define which type of content and how it's internally organized.

This definition is much easier to understand with some examples:
- For container images, the content type that a user wants to manage cohesively is an image. The user shouldn't care about manifests or layers, only about images.
- For helm charts, the content type that the user wants to manage cohesively is a helm chart. The user shouldn't need to care about the fact that the helm chart will be always accompanyed by a file that helps vefifying the provenance of the helm chart.
- For a packaging system, the content type that a user wants to manage cohesively is a package. It's an implementation detail irrelevant for the user, if a "package" provides only the binaries, or also additional content like installation hooks, a SBOM, all licenses, the package signature,...

## Artifact Author

The owner of an artifact format. The [OCI Image Spec](https://github.com/opencontainers/image-spec/) is owned by the OCI working group.

An artifact is defined to be unique by its `config.mediaType`.

## Container Registry

See [Registry](#registry)

## Distribution Operator

Vendors that implement and/or run the [OCI Distribution Spec](https://github.com/opencontainers/distribution-spec/).

## Media Type

The uniqueness of an artifact is defined by its type. An artifact has a type, which has a collection of layers.

The Artifact is defined as unique by its `manifest.config.mediaType`. Layers are defined by their `layer.config.mediaType`.

## OCI Image

OCI Image is a specific type of artifact. However, an OCI image is not meant to define all artifacts. Tooling, such as docker, containerD and vulnerability scanners that perform security checks upon container images, use the `config.mediaType` to know they can pull and instance container images. Docker and containerD are not intended to pull and instance Helm Charts, Singularity, OPA or other artifact types.

## Registry

A registry, or container registry, is an instance of the [distribution-spec]. See [Implementers][implementers] for a list of registries that support OCI Artifacts.

## Well Known Type

Types that many to most registry operators would likely want to support ([OCI Image][image-spec], [Helm][helm], [Singularity][singularity]). While registry operators are not required to support all types, registry operators would likely want to support well known types, if there was an easy way to understand the differing types. OCI Artifacts includes publishing of well-known types for registry operators to import.

## YASS

[OCI Artifacts][artifacts] provides an alternative to having to build, distribute and run "**Y**et **A**nother **S**torage **S**ervice".

[artifacts]:          https://github.com/opencontainers/artifacts
[helm]:               https://helm.sh
[implementers]:       implementers.md
[image-spec]:         https://github.com/opencontainers/image-spec/
[distribution-spec]:  https://github.com/opencontainers/distribution-spec/
[singularity]:        https://github.com/sylabs/singularity
