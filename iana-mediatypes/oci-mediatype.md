**Name:** Steve Lasker

**Email:** tob@opencontainers.org

**Media type name:** application

**Media subtype name:** application/vnd.oci.image.manifest.v1+json

**Required parameters:** n/a

**Optional parameters:** n/a

**Encoding considerations:** binary

**Security Considerations:** This JSON format media type inherits the security considerations for JSON. See RFC 8259 section 12. (https://tools.ietf.org/html/rfc8259#section-12)

**Privacy Information:** The manifest contains configuration information, with optional annotations. While users may put any content they wish into these annotations, there are no requirements for privacy information that must be protected.

**Extensibility:** The json payload supports annotations as strings. No executable information or code paths are enabled.

**Compression:** The manifest type is not compressed in normal workflows.

**Links:** The manifest contains digests of other linked content, which are separately retrieved, with their own security considerations.

**Interoperability Considerations:** An OCI image manifest may represent one of many different platforms and architectures. When retrieving a manifest, the config descriptor https://github.com/opencontainers/image-spec/blob/master/config.md returns a an object the consumer can determine if this manifest applies to the host platform and architecture.

**Published specification:**
application/vnd.oci.image.manifest.v1+json spec: https://github.com/opencontainers/image-spec/blob/master/manifest.md
Content Digest spec: https://github.com/opencontainers/distribution-spec/blob/master/spec.md#content-digests

**Applications which use this media:**
- Implementations of the oci-distribution spec (https://github.com/opencontainers/distribution-spec), providing registry distribution:
    - docker distribution: https://github.com/docker/distribution
    - project quay: https://www.projectquay.io/
    - harbor project: https://goharbor.io/
- Cloud vendors that implement registries
    - aws https://aws.amazon.com/ecr/
    - azure: https://azure.microsoft.com/services/container-registry/
    - gcp: https://cloud.google.com/container-registry/
- Clients which interact with oci-distribution-spec, including
    - The moby project: https://mobyproject.org/
    - The containerd project: https://containerd.io/

**Fragment identifier considerations:** n/a

**Restrictions on usage:** None

**Provisional registration? (standards tree only):** n/a

**Additional information:** 
1. **Deprecated alias names for this type:** n/a 
2. **Magic number(s):** n/a 
3. **File extension(s):** n/a 
4. **Macintosh file type code:** n/a 
5. **Object Identifiers:** n/a

**General Comments:**

**Person to contact for further information:**

1. Name: Steven Lasker
2. Email: StevenLasker@hotmail.com

**Intended usage:** Common
OCI (https://www.opencontainers.org/<https://www.opencontainers.org/>) is an open governance body for the purpose of creating open industry standards around container formats and runtimes.
The application/vnd.oci.image.manifest.v1+json mediaType is manifest providing a set of layers and an optional config object for a single OCI Artifact (https://github.com/opencontainers/artifacts).
