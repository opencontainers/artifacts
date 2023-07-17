# OCI Artifacts

This repository has been archived and is no longer maintained. 

## OCI Artifacts Merges with Image and Distribution Specs

OCI Artifacts started in 2019 with [PRs to the Image](https://github.com/opencontainers/image-spec/pull/770) and the [Distribution specs](https://github.com/opencontainers/distribution-spec/pull/65).  In September of 2019, the TOB Voted to create the [New OCI Artifacts Project](https://opencontainers.org/posts/blog/2019-09-10-new-oci-artifacts-project/), generalizing the deployed container registry infrastructure to serve ecosystems of new artifacts. Artifact Authors no longer needed to create, nor host new package managers. By leveraging existing public and private registries, artifact authors benefit from the reliability, performance and security benefits users already manage. 

In 2020, early supply chain security initiative were evolving, requiring the addition of [reference information to existing artifacts #29](https://github.com/opencontainers/artifacts/pull/29). This ranged from signatures, Software Bill of Materials (SBOM), to a breadth of new developing types.

We're happy to see the journey completed with the Image and Distribution specs formalizing the addition of OCI Artifacts and Reference Types. With the [Image 1.1](https://github.com/opencontainers/image-spec/releases), and [Distribution 1.1 releases](https://github.com/opencontainers/distribution-spec/releases), the effort has come full circle, making it time to archiving the OCI Artifacts project.

Guidance is now available at:
- [Artifact Authors Guidance](https://github.com/opencontainers/image-spec/blob/main/manifest.md#guidelines-for-artifact-usage)
- [Referrers API](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#enabling-the-referrers-api)
