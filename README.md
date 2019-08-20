# OCI Artifacts

Container registries, implementing the [distribution-spec][distribution-spec], provide reliable, highly scalable, secured storage services for container images. Customers either use a cloud provider implementation, vendor implementations, or instance the open source implementation of [distribution][distribution]. They configure security and networking to assure the images in the registry are locked down and accessible by the resources required. Cloud providers and vendors often provide additional values atop their registry implementations from security to productivity features.

Applications and services typically require additional artifacts to deploy and manage, including [helm](https://helm.sh) for deployment and [Open Policy Agent (OPA)](https://github.com/open-policy-agent/opa/issues/1413) for policy enforcement. 

Utilizing the [manifest][image-manifest] and [index][image-index] definitions, new artifacts, such as the [Singularity project][singularity], can be stored and served using the [distribution-spec][distribution-spec]. 

This repository provides a reference for artifact authors and registry implementors for supporting new artifact types with the existing implementations of distribution.

By providing an OCI artifact definition, the community can continue to innovate, focusing on new artifact types without having to build yet another storage solution (YASS). 

## Table of Contents

* [The Apache License, Version 2.0](LICENSE)
* [Maintainers](MAINTAINERS)
* [Maintainer guidelines](MAINTAINERS_GUIDE.md)
* [Contributor guidelines](CONTRIBUTING.md)
* [Project governance](GOVERNANCE.md)
* [Release procedures](RELEASES.md)

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

A: No.  Artifacts are a prescriptive means of storing [index][image-index] and [manifest][image-manifest] within [distribution][distribution-spec] implementations. 

[distribution-spec]: https://github.com/opencontainers/distribution-spec/
[code-of-conduct]: https://github.com/opencontainers/org/blob/master/CODE_OF_CONDUCT.md
[image-manifest]: https://github.com/opencontainers/image-spec/blob/master/manifest.md
[image-index]: https://github.com/opencontainers/image-spec/blob/master/image-index.md
[distribution]: https://github.com/docker/distribution
[singularity]: https://github.com/sylabs/singularity