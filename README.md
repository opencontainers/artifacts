# OCI Artifacts (Experimental) with OCI Artifact Reference Support

This is an experimental branch of oci artifacts, validating the [oci.artifact.manifest spec][oci-artifact-manifest-spec] support of reference types. Reference types are required to meet the [Notary v2 Requirements][nv2-requirements] for not changing the target digest or tag, and the [Notary v2 Scenarios][nv2-scenarios] for content movement within and across registry implementations. Reference types enable a wider range of scenarios, including secure supply chain artifacts that may be represented as a graph as they move across environments.

![](media/net-monitor-graph.svg)

## Table of Contents:

- [OCI Artifact Manifest Overview][oci-artifact-manifest]
- [OCI Artifact Reference Type Manifest Spec](./artifact-reftype-spec.md)
- [ORAS experimental support for oci.artifact.manifest references][oras-artifacts] to `push`, `discover`, `pull` referenced artifact types.

[oci-artifact-manifest]:      ./artifact-manifest.md
[oci-artifact-manifest-spec]: ./artifact-reftype-spec.md
[nv2-requirements]:           https://github.com/notaryproject/notaryproject/blob/main/requirements.md
[nv2-scenarios]:              https://github.com/notaryproject/notaryproject/blob/main/scenarios.md
[oras-artifacts]:             https://github.com/deislabs/oras/blob/prototype-2/docs/artifact-manifest.md