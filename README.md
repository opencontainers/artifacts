# OCI Artifacts

Container registries, implementing the [distribution-spec][distribution-spec], provide reliable, highly scalable, secured storage services for container images. Customers either use a cloud provider implementation, vendor implementations, or instance the open source implementation of [distribution][distribution]. They configure security and networking to assure the images in the registry are locked down and accessible by the resources required. Cloud providers and vendors often provide additional values atop their registry implementations from security to productivity features.

Applications and services typically require additional artifacts to deploy and manage, including [helm](https://helm.sh) for deployment and [Open Policy Agent (OPA)](https://github.com/open-policy-agent/opa/issues/1413) for policy enforcement. 

Utilizing the [manifest][image-manifest] and [index][image-index] definitions, new artifacts, such as the [Singularity project][singularity], can be stored and served using the [distribution-spec][distribution-spec]. 

This repository provides a reference for artifact authors and registry implementors for supporting new artifact types with the existing implementations of distribution.

By providing an OCI artifact definition, the community can continue to innovate, focusing on new artifact types without having to build yet another storage solution ([YASS][def-yass]). 

## OCI Artifact Table of Contents

* [What Is an OCI Artifact](#what-is-an-oci-artifact)
* [Defining OCI Artifact Types](#defining-oci-artifact-types)
* [Definitions & Terms](definitions-terms.md)
* [OCI Artifact Implementations](implementors.md)

## OCI Governance
* [The Apache License, Version 2.0](LICENSE)
* [Maintainers](MAINTAINERS)
* [Maintainer guidelines](MAINTAINERS_GUIDE.md)
* [Contributor guidelines](CONTRIBUTING.md)
* [Project governance](GOVERNANCE.md)
* [Release procedures](RELEASES.md)

## What Is an OCI Artifact

An OCI Artifact is a generic reference to an object that can be stored in an instance of [OCI Distribution][distribution-spec]. The artifact can be [def-well known types][well-known-types], such as [OCI-Image][image-spec] and [Singularity][singularity], or customer or vendor specific types. 

## Defining OCI Artifact Types

[Registries][def-registry], vulnerability scanners and artifact tooling must understand the types of artifacts they support. Registry scanning tools may only support a subset of artifact types, or they may need to apply different scanning methods based on the artifact type. 

If a security scanning solution were to scan all types, it would fail when it encounters unsupported types, representing false negatives. By differentiating types, a registry scanning solution can ignore unknown types, representing a known state. As new artifact types become [well known][def-well-known-types], scanners can expand the types they offer, providing a more complete known state. 

Artifact tooling must also know the types they support. The docker and containerD client know how to instance container images. However, they are not intended to instance Helm Charts or Singularity images. By defining the artifact type, registries can present the type to their users, and tools pulling artifacts from a registry can determine if they can support the specific type before encountering a runtime error. 

Artifacts are defined by setting the `manifest.config.mediaType` to a globally unique value. The `config.mediaType` of `application/vnd.oci.image.config.v1+json` is reserved for artifacts intended to be instanced by docker and [containerD][containerd].

## Code of Conduct

This project incorporates (by reference) the OCI [Code of Conduct][code-of-conduct].

## Governance and Releases

This project incorporates the Governance and Releases processes from the OCI project template: https://github.com/opencontainers/project-template.

## Project Communications

This project would continue to use existing channels in use by the [OCI developer community for communication](https://github.com/opencontainers/org#communications)

### Versioning / Roadmap

Artifacts will reference specific [distribution][distribution-spec], [index][image-index] and [manifest][image-manifest] versions in its examples, identifying any dependencies required. 

## Frequently Asked Questions (FAQ)

**Q: Does this change the OCI Charter or Scope Table?**

A: No. Artifacts are a prescriptive means of storing [index][image-index] and [manifest][image-manifest] within [distribution][distribution-spec] implementations. 

[containerd]:           https://containerd.io/
[code-of-conduct]:      CODE_OF_CONDUCT.md
[distribution]:         https://github.com/docker/distribution
[distribution-spec]:    https://github.com/opencontainers/distribution-spec/
[def-registry]:         definitions-terms.md#registry
[def-well-known-types]: definitions-terms.md#well-known-types
[def-yass]:             definitions-terms.md#yass
[image-index]:          https://github.com/opencontainers/image-spec/blob/master/image-index.md
[image-manifest]:       https://github.com/opencontainers/image-spec/blob/master/manifest.md
[image-spec]:           https://github.com/opencontainers/image-spec/
[singularity]:          https://github.com/sylabs/singularity
