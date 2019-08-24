# Definitions and Terms

A collection of definitions and terms used within this repository.

* [Artifact Author](#artifact-author)
* [Distribution Operator](#distribution-operator)
* [Media Type](#media-type)
* [OCI Image](#oci-image)
* [Registry](#registry)
* [YASS](#yass)

## Artifact Author

The owner of an artifact format. The [OCI Image Spec](https://github.com/opencontainers/image-spec/) is owned by the OCI working group. 

<<<<<<< HEAD
An artifact is defined to be unique by it's `config.mediaType`. 
=======
An artifact is defined to be unique by its `config.mediaType`. 
>>>>>>> readme-overview

## Container Registry

See [Registry](#registry)


## Distribution Operator

<<<<<<< HEAD
Vendors that [implement][Implementors] and/or run the [OCI Distribution Spec](https://github.com/opencontainers/distribution-spec/). 

## Media Type

The uniqueness of an artifact is defined by it's type. An artifact has a type, which has a collection of layers.

The Artifact is defined as unique by it's `manifest.config.mediaType`. Layers are defined by their `layer.config.mediaType`. 
=======
Vendors that implement and/or run the [OCI Distribution Spec](https://github.com/opencontainers/distribution-spec/). 

## Media Type

The uniqueness of an artifact is defined by its type. An artifact has a type, which has a collection of layers.

The Artifact is defined as unique by its `manifest.config.mediaType`. Layers are defined by their `layer.config.mediaType`. 
>>>>>>> readme-overview

## OCI Image

OCI Image is a specific type of artifact. However, an OCI image is not meant to define all artifacts. Tooling, such as docker, containerD and vulnerability scanners that perform security checks upon container images, use the `config.mediaType` to know they can pull and instance container images. Docker and containerD are not intended to pull and instance Helm Charts, Singularity, OPA or other artifact types. 

## Registry

<<<<<<< HEAD
A registry, or container registry, is an instance of the [distribution-spec]. See [Implementors][Implementors] for a list of registries that support OCI Artifacts. 
=======
A registry, or container registry, is an instance of the [distribution-spec]. See [Implementors][implementors] for a list of registries that support OCI Artifacts. 
>>>>>>> readme-overview

## YASS

[OCI Artifacts][artifacts] provides an alternative to having to build, distribute and run "**Y**et **A**nother **S**torage **S**ervice". 

[artifacts]: https://github.com/opencontainers/artifacts
<<<<<<< HEAD
[Implementors]: https://github.com/SteveLasker/artifacts/blob/implementors/implementations.md
=======
[implementors]: https://github.com/SteveLasker/artifacts/blob/implementors/implementations.md
>>>>>>> readme-overview
[distribution-spec]: https://github.com/opencontainers/distribution-spec/
