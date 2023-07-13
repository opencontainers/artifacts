# OCI Artifacts

## Mission Update
**The 1.1 releases of [OCI image-spec](https://github.com/opencontainers/image-spec) and [distribution spec](https://github.com/opencontainers/distribution-spec) are tasked with providing details for enabling OCI artifacts support.** The guidance provided therein replaces all 1.0 guidance content provided herein in this repo. 

The 1.1 version of the [OCI image-spec](https://github.com/opencontainers/image-spec) and [distribution spec](https://github.com/opencontainers/distribution-spec) are currently in release candidate review stage. In particular the image spec provides a few examples and spec changes in the section [Guidelines for Artifact Usage](https://github.com/opencontainers/image-spec/blob/main/manifest.md#guidelines-for-artifact-usage) and also link to changes in the distribution spec for using the referrers api to retrive artifacts that refer to OCI content. Some guidance for supporting artifacts by 1.0 compliant registries and clients is provided in the distribution specification release candidate.

We hope to provide additional guidance to assist in transitioning. That additional guidance may be done in the image and/or distibution specifications, as the scope of this repo is currently being reviewed by the tob.

Catch us on slack for more details and/or please chime in with PRs and help in drafting new guidance docs for artifact authors, registry operators, vendors, and clearing house content issues not already covered by [IANA](https://www.iana.org/assignments/media-types/media-types.xhtml). 

## Artifact Guidance Documents - 1.0 only and not in sync with 1.1 changes

1. [Artifact Author Guidance](./artifact-authors.md)

## Supporting Documents - 1.0 only and not in sync with 1.1 changes

- [Term Definitions](./definitions-terms.md)

## Project Introduction and Scope - 1.0 only and not in sync with 1.1 changes

Container registries, implementing the [distribution-spec][distribution-spec], provide reliable, highly scalable, secured storage services for container images. Customers either use a cloud provider implementation, vendor implementations, or instance the open source implementation of [distribution][distribution]. They configure security and networking to assure the images in the registry are locked down and accessible by the resources required. Cloud providers and vendors often provide additional values atop their registry implementations from security to productivity features.

Applications and services typically require additional artifacts to deploy and manage, including [helm](https://helm.sh) for deployment and [Open Policy Agent (OPA)](https://github.com/open-policy-agent/opa/issues/1413) for policy enforcement.

Utilizing the [manifest][image-manifest] and [index][image-index] definitions, new artifacts, such as the [Singularity project][singularity], can be stored and served using the [distribution-spec][distribution-spec].

This repository provides a reference for artifact authors and registry implementors for supporting new artifact types with the existing implementations of distribution.
More particularly this repository has been tasked by the [OCI TOB](https://github.com/opencontainers/tob/blob/master/proposals/artifacts.md) to serve 3 primary goals:

1. **artifact authors** - [guidance for authoring new artifact types.][artifact-authors] Including a clearing house for well known artifact types.
1. **registry operators and vendors** - guidance for how operators and vendors can support new artifact types, including how they can opt-in or out of well known artifact types. Registry operators that already implement `media-type` filtering will not have to change. The artifact repo will provide context on how new `media-type`s can be used, and how `media-type`s can be associated with a type of artifact.
1. **clearing house for well known artifacts** - artifact authors can submit their artifact definitions, providing registry operators a list by which they can easily support.

By providing an OCI artifact definition, the community can continue to innovate, focusing on new artifact types without having to build yet another storage solution (YASS).

## Project Status - 1.0 only and not in sync with 1.1 changes

The current state of the [OCI Artifacts][oci-artifacts] repository:
- The repository contains guidance for using [v1.0.1][oci-image-v101] of the [OCI image manifest][image-manifest] representing *individual* non-container image artifact types.
- This project recognizes that additional work is needed to find ways to improve existing OCI artifact types, such as OCI images, to formally include a software bill of materials (SBOMs), scan results, signatures, and other OCI artifact related extensions. Depending on the implementation chosen, additional APIs to manage these extensions may also be needed. We believe these requirements will either require modifications to the existing specs or some new specification depending on the output of various working groups.  
  **This project, however, does not currently have the mission to create new specifications or commit changes to the existing specifications.**

## Related Projects Working on Extending OCI Specs

  - [OCI Reference Type Working Group][oci-reftype-wg]
  - [CNCF ORAS Artifacts Spec][oras-artifacts]

## Project Governance and License

- [Artifact Authors- How To][artifact-authors]
- [The Apache License, Version 2.0](LICENSE)
- [Maintainers](MAINTAINERS)
- [Maintainer guidelines](MAINTAINERS_GUIDE.md)
- [Contributor guidelines](CONTRIBUTING.md)
- [Project governance](GOVERNANCE.md)
- [Release procedures](RELEASES.md)

## Code of Conduct

This project incorporates (by reference) the OCI [Code of Conduct][code-of-conduct].

## Governance and Releases

This project incorporates the Governance and Releases processes from the OCI project template: https://github.com/opencontainers/project-template.

## Project Communications

This project uses existing channels in use by the [OCI developer community for communication](https://github.com/opencontainers/org#communications)

### Versioning / Roadmap

Artifacts will reference specific [distribution][distribution-spec], [index][image-index] and [manifest][image-manifest] versions in its examples, identifying any dependencies required.

## Frequently Asked Questions (FAQ)

**Q: Does this change the OCI Charter or Scope Table?**

A: No.  Artifacts are a prescriptive means of storing [index][image-index] and [manifest][image-manifest] within [distribution][distribution-spec] implementations.

[artifact-authors]:     ./artifact-authors.md
[code-of-conduct]:      https://github.com/opencontainers/.github/blob/master/CODE_OF_CONDUCT.md
[distribution]:         https://github.com/distribution/distribution
[distribution-spec]:    https://github.com/opencontainers/distribution-spec/
[image-index]:          https://github.com/opencontainers/image-spec/blob/main/image-index.md
[image-manifest]:       https://github.com/opencontainers/image-spec/blob/main/manifest.md
[oci-artifacts]:        https://github.com/opencontainers/artifacts
[oci-image-v101]:       https://github.com/opencontainers/image-spec/releases/tag/v1.0.1
[oci-reftype-wg]:       https://github.com/opencontainers/wg-reference-types/
[oras-artifacts]:       https://github.com/oras-project/artifacts-spec/
[singularity]:          https://github.com/sylabs/singularity
