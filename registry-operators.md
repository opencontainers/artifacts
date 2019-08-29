# Registry Operators - Hosting Artifacts

The primary goal of OCI Artifacts brings consistent user experiences across all artifact types. Customers can consistently use `docker push/pull` between [OCI Distribution][distribution-spec] implementations. Rather than asking every registry to support different types with registry specify CLIs, OCI Artifacts enables artifact authors native experiences like `helm registry push`|`pull` and `singularity push`|`pull`.

For OCI artifacts to be broadly successful, there are a set of expectations customers should have, while giving registry vendors the opportunity to provide differentiated value atop artifacts.

## Hosting Artifacts - Table of Contents

* [What's Required to Host OCI Artifacts?](#what's-required-to-host-oci-artifacts)
* [What's Optional to Host OCI Artifacts?](#what's-optional-to-host-oci-artifacts)
* [Garbage Collecting Artifacts](#garbage-collecting-artifacts)
* [Referencing Artifacts with Tags and Digests](#tagging-and-digest-references)

## What's Required to Host OCI Artifacts

Serving OCI Artifacts is a generalization of the infrastructure used to store Docker and OCI images. Manifests are used to define the artifact, with layers representing the files used to represent the artifact. Supporting artifacts is a matter of allowing additional `mediaTypes` for `manifest.config.mediaType` and `layer.config.mediaType`.

## What's Optional to Host OCI Artifacts

The following are considered decisions left to the registry vendor and/or operator.

* **Unique artifact support per registry instance, customer instance or repository**  
  As artifacts evolve, the market will determine the expectation for how unique the scope of a set of artifacts mus be supported. It is expected customers will have a set of unique types they will want to support. Whether these are scoped to a specific repo within the registry is currently left to the vendor and/or operator to determine.
* **Tag listing to identify the artifact type**  
  Registry users will want to understand the types stored within a registry, and within a given registry/repo. The [Authoring Artifacts][authoring-artifacts] section defines the unique identifier, localized type name and a logo to be displayed. This information is defined by OCI Artifacts for the registry operator to decide how they wish to present the information. All [well-known types][def-well-known-types] published under [OCI Artifacts][oci-artifacts-repo] are published with consent that all registry operators may display the name and logo.

## Supporting Additional MediaTypes

OCI Artifacts defines a goal that registry operators must have knowledge of the types they store. Artifacts declare their type by setting `manifest.config.mediaType` to a globally unique value. As a registry operator, validations must be expanded to understand these types.

As images are pushed to a registry, many registries perform some amount of validation as manifests are posted. Supporting artifacts means supporting additional `config.mediaTypes` in the [OCI Manifest][image-manifest].

### Layer MediaTypes

For each artifact type, a collection of valid `layer.mediaTypes` are defined. As users push artifacts to a registry, the list of layer `mediaTypes` are also expanded, as they correlate with a given artifact type. Registries operators MUST fail validation if layers of type2 are pushed to an artifact defined as type1.

### Layer file formats

OCI Artifacts defines layers as blobs. The file format of the layers is up to the artifact author. Artifacts may push their layers in any format they deem needed for their type. This includes `.tar`, `.json`, `.config`, `.sif` or any single file representation.

Layer sizes are not determined by the OCI Image or Artifact Specs and left for the operator to decide.

### Discovering and Importing Artifact Type Definitions

The [OCI Artifacts repo][oci-artifacts-repo] maintains a collection of [well known][def-well-known-types] types. Registry operators may import these well-known types, or any other type they wish to support.

Supporting OCI Artifacts does NOT require supporting *all* well-known types. The decision for which types to support is left to the registry operator. Registry operators may support a subset, or superset of types they wish to uniquely support for all of their customer registries, a subset of customer registries, or scoped to a repo within a registry.

Additional reference:

* [Authoring Artifacts][authoring-artifacts]
* [Publishing Artifact Types][publishing-artifact-types]

## Garbage Collecting Artifacts

OCI Artifacts uses [oci manifest][image-manifest] to define referenced layers. As with all oci manifests, layers can be shared. Registry operators should use the same Docker and OCI Image semantics to track layers. OCI Artifacts may be pulled by `:tags` or their manifest digest. When manifests are deleted from a registry, ref counting of layers are typically performed, deleting layers without manifest references. Docker and OCI Images are a specific artifact type, where all should be treated equally.

## Tagging and Digest References

OCI Artifacts are a generalization of Docker and OCI Images. Registry operators should support the same push/pull semantics for all artifacts.

The following example uses [ORAS][oras] for CLI based testing.

### Using Tags

```bash
# Docker Push/Pull
docker push [loginURL]/my-image:v1

docker pull [loginURL]/my-image:v1

# Artifacts w/ORAS Push/Pull
oras push [loginURL]/my-artifact:v1 \
  --manifest-config /dev/null:application/vnd.my-company.foo.config.v1+json \
  ./artifact.tar:application/vnd.my-company.foo.layer.v1+tar

oras pull [loginURL]/my-artifact:v1 \
  --media-type application/vnd.my-company.foo.config.v1+json
```

### Using Digests

```bash
# Docker Push/Pull
docker pull [loginURL]/my-image@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2

# Artifacts w/ORAS Push/Pull
oras pull [loginURL]/my-artifact@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2 \
  --media-type application/vnd.my-company.foo.config.v1+json
```

## OPTIONAL: Manifest Config Validation

The value of `manifest.config.mediaType` is the unique identifier of the artifact type. However, artifact types are not required to define config schema. The value of config may be a `null` reference. Registry operators MAY choose to validate and/or parse the `manifest.config.mediaType` for interesting information within the type. Artifacts define the optional use and schema in the [Defining Config Schema][schema-config] section. Config schemas MUST be versioned to protect consumers, such as registry operators and artifact tools.

## Future Discussions

* **Search API - replacement for `_catalog`**  
  The `_catalog` api is [proposed as deprecated and reserved](https://github.com/opencontainers/distribution-spec/pull/69) for a future specification. While registry operators are encouraged to contribute to a new search api, it is expected that registry vendors and operators will need to expose a solution while this new spec evolves. When presenting search results, it is proposed to display the localized text of the artifact type, as the artifact `mediaType` is intended for computer uniqueness.
* **tag listing api**  
  Similar to the `_catalog` API, it's recognized the tag listing api will need to evolve to express the various types that will now be available.

[authoring-artifacts]:        authoring-artifacts.md
[containerd]:                 https://containerd.io/
[code-of-conduct]:            CODE_OF_CONDUCT.md
[distribution]:               https://github.com/docker/distribution
[distribution-spec]:          https://github.com/opencontainers/distribution-spec/
[def-registry]:               definitions-terms.md#registry
[def-well-known-types]:       definitions-terms.md#well-known-types
[def-yass]:                   definitions-terms.md#yass
[image-index]:                https://github.com/opencontainers/image-spec/blob/master/image-index.md
[image-manifest]:             https://github.com/opencontainers/image-spec/blob/master/manifest.md
[image-spec]:                 https://github.com/opencontainers/image-spec/
[oci-artifacts-repo]:         https://github.com/opencontainers/artifacts
[oras]:                       https://github.com/deislabs/oras
[publishing-artifact-types]:  authoring-artifacts.md#optional:-publishing-the-artifact-type
[schema-config]:              authoring-artifacts.md#optional-defining-config-schema
[singularity]:                https://github.com/sylabs/singularity
