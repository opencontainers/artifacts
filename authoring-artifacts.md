# Authoring Artifacts

As registries become content addressable distribution points, tools that pull artifacts must know if they can operate on the artifact. Artifact types are equivalent to file extensions. When users open files, the host operating system typically launches the appropriate program. When users open a file, from within a program, the open dialog filters to the supported types. When search or security software scans the contents, the software must to know how to process the different types of files. And when users look at the contents of a disk, they see the type of each file.

OCI Artifacts takes the following approach:

* Define a unique type
* Define the format for other tools to operate upon the type
* Define human elements, such as an icon and localized string to be displayed to users
* Optionally, publish the type for registries to consume

Defining a unique type, string and logo; artifacts can be represented as the following:

|Icon|Artifact|`config.mediaType`|
|-|-|-|
|<img src=./media/oci.svg width=30x>|[OCI Image][image-spec]|`application/vnd.oci.image.config.v1+json`|
|<img src=./media/moby.png width=30x>|[Docker Image][docker]|`application/vnd.docker.container.image.v1+json`|
|<img src=./media/helm.svg width=30x>|[Helm Chart](https://helm.sh)|`application/vnd.cncf.helm.chart.config.v1+json`|
|<img src=./media/singularity.svg width=30x>|[Singularity][singularity], by [Sylabs][sylabs]|`application/vnd.sylabs.sif.config.v1+json`|

> *Docker and OCI Image Types*  
In the above examples, Docker and OCI are both listed as image types. The docker type is associated with the original docker format. OCI Image is intended as the vendor neutral format, used by the [containerD][containerd] project and other container runtime tools. Most container tools and registries that support docker should interchangeably convert Docker Image to OCI Image.

## Defining An Artifact

Authoring artifact types involves the following steps:

1. [Defining a Unique Artifact Type](#defining-a-unique-artifact-type)
1. [Defining the Supported Layer Types](#defining-supported-layer-types)
1. [Optional: Defining Config Schema](#optional-defining-config-schema)
1. [Optional: Adding Annotations](#optional-adding-annotations)
1. [Defining the Artifact Type](#defining-the-artifact-type)
1. [Push an Artifact to an OCI Artifact Registry](#pushing-an-oci-artifact)
1. [Optional: Publishing the Artifact Manifest](#optional-publishing-the-artifact-type) as a [Well Known Type][def-well-known-types]

## Defining a Unique Artifact Type

Defining the unique type involves uniqueness for computer processing, and uniqueness for humans.

For the computers, artifacts are defined by setting the `manifest.config.mediaType` to a globally unique value. Defining unique values enables registries and tooling to differentiate types.

> **Note:** The `config.mediaType` of `application/vnd.oci.image.config.v1+json` is reserved for artifacts intended to be instanced by [docker][docker] and [containerD][containerd].  
*Each artifact shall have its own unique type.*

 The following format is used to differentiate the type of artifact:

`application/vnd.`[org|company]`.`[objectType]`.`[optionalSubType]`.config.`[version]`+json`

For the humans, a localized string and logo in `.svg` format are provided in the [artifact type manifest](#defining-the-artifact-type-manifest)

## Defining Supported Layer Types

Artifacts are intended to have content. The content of an artifact is represented through one or more layers. How the layers are constructed, and whether the layers are ordinal is a decision of the artifact author.

As an example, [OCI Images][image-layer] are represented through an ordinal collection of compressed files. Each layer overlays the previous layer.
Other artifacts may be represented by a single file, such as a markdown document, or a config file. Other artifacts may include single config file as one layer, and a collection of binaries, compressed as another layer. By separating the layers, the artifact author can benefit from layer de-duplication and concurrent downloading of each layer.

### Layer File Format

Layers are persisted as blobs within registries. The blobs can be a single file or a collection of files. The persistance format is up to the artifact author. They may choose to persist individual files with their native or custom format, such as `.config`, `.json`, `.bin`, `.sif`, or compress them with various formats such as `.tar`.  
Large files can benefit from compression when being transferred across the network. However, decompression takes time and compute cycles. For smaller, individual files, the decompression may take longer than downloading the file in its original format.

### One or More Layer Types

The number of layer types is up to the artifact author. Some things to consider when designing the layer format:

* **Continuos Builds:** will developers automate building and pushing the artifact to a registry? For each rebuild, what will change? Consider the docker layering model, where users benefit from base layers that don't change often. If only a small piece of content changes, can you separate that content into a unique layer?
* **Layer reuse**: when layers are pushed to a registry, layers can be de-duped. If your artifact type may benefit from shared layers across many artifact instances, consider splitting up the layers to those that change often, and those that don't.
* **Layer sizes**: does your artifact have large content that may benefit from being split up in to smaller elements that can be concurrently uploaded and/or downloaded to a registry?
* **Content and Config**: While a general best practice for container images says the image in the registry should not contain environmental configuration, you may be choosing to push environmental configuration to a registry as it's own artifact. If the same artifact is pushed to the same registry multiple times, only unique by some configuration, consider splitting up the configuration into its own layer, to benefit from layer reuse.

### Shared Layer Example

Consider an Azure Resource Manager (ARM) template stored in a registry. You may have the base template stored for shared team use. As teams deploy that template, storing at least some of the configuration information, consider storing the parameters file as a separate layer. When pushed to a registry, most registry operators will de-dupe the base template, only storing the unique parameters layer for each artifact.

### Layer Versioning

Layers MUST be versioned to future proof any new enhancements that may evolve. How the artifact tooling processes forward and backward compatibility is up to the artifact and tooling authors.

### Defining Layer Types

Artifact layers are expected to be unique, per artifact author. A layer that represents an OCI Image is not expected to be shared with a Helm Chart. To differentiate layers, artifact authors SHOULD create unique artifact layer types.

Artifact layer types utilize the `layer.mediaType` with the following format:  
`application/vnd.`[org|company]`.`[layerType]`.`[layerSubType]`.layer.`[version]`+`[fileFormat]

### Example Layer Types

|Artifact Type|mediaType|
|-|-|
|OCI Image|`application/vnd.oci.image.layer.v1+tar`|
|Docker Image|`application/vnd.docker.image.rootfs.diff.tar.gzip`|
|Helm Chart|`application/vnd.cncf.helm.chart.layer.v1+tar`|
|Singularity SIF Layer|`application/vnd.sylabs.sif.layer.v1+tar`|

## Optional: Defining Config Schema

While the value of `manifest.config.mediaType` is used to determine the artifact type, the persistance of a `config.json` file is OPTIONAL. Artifacts can push a null reference for `config.json` persistance.

When defining an artifact type, the persistance of the artifact may be broken up into content and configuration. Configuration can be stored as a layer, or it can be stored and referenced by the `manifest.config`.

Some benefits of using `manifest.config` include:

* Tooling can pull the configuration prior to any layers. Depending on the artifact type, the layer request might be sent to another compute instance, while the configuration is used to determine how and where the layer should be instanced, such as whether to send the layers to a Windows or Linux Host.
* Registries may opt-into parsing the configuration if it provides meaningful top-level information. [OCI Image Config][image-spec-config] stores `OS`, `Architecture` and `Platform` information that some registry operators may wish to display. The config is easy to pull & parse, as opposed to getting a layer url to pull, possibly decompress and parse.

Distribution instances MAY:

* Parse and process the contents of  `manifest.config`, based on the provided schema of `manifest.config.mediaType`, offering additional information or actions.
* Ignore the contents and validation of the config.json file

## Optional: Adding Annotations

> **TODO**: pull in content from: [OCI Artifact Authoring: Annotations & Config.json](https://stevelasker.blog/2019/08/08/oci-artifact-authoring-annotations-config-json/)

## Defining the Artifact Type

For a registry to understand specific artifacts, validate the artifact type, supported layer types, and optionally present information to users of a registry, an `artifactType.json` file is defined using the [following schema](artifactTypes/artifactTypeSchema.0.1.json).

* **`mediaType`** *string*  
  This REQUIRED property uniquely identifies the artifact for computer consumption. It may be owned by an org or a company and MUST be globally unique and versioned.  
  The format of `mediaType` MUST use the following format:  
  `application/vnd.`[org|company]`.`[objectType]`.`[optionalSubType]`.config.`[version]`+json`

* **`title`** *string-string map*  
  This REQUIRED property must have at least one value, representing the name of the type displayed for human consumption. The title may be displayed in a repository listing, or registry tooling.  
  Title is a collection of localized strings, indexed with [ISO Language Codes][iso-lang-codes].
  * **`locale`** *string*  
    2-2 language codes representing the country and locale. [ISO Language Codes][iso-lang-codes]
  * **`title`** *string*  
Localized title. The max length MUST not exceed 30 characters and MUST not encode formatting characters.
* **`description`** *string-string map*  
    This REQUIRED property must have at least one value, providing a short description of the type for human consumption. The description may be displayed in repository listings or registry tooling.  
    Description is a collection of localized strings, indexed with [ISO Language Codes][iso-lang-codes].  
  * **`locale`** *string*  
    2-2 language codes representing the country and locale. [ISO Language Codes][iso-lang-codes]
  * **`description`** *string*  
Localized description. The max length MUST not exceed 255 characters and MUST not encode formatting characters.
* **`moreInfoUrl`** *url*  
This OPTIONAL property provides additional details, intended for consumers of the type. This is most often a marketing & awareness overview page.
* **`specUrl`** *url*  
This OPTIONAL property references a spec, providing additional details on the type.
* **`tools`** *string-string map*  
  This OPTIONAL property provides a collection of tools that may be used with artifact type. The property is intended for end users to find more info on how to find and install related tools. Registry operators MAY provide links to the tools in their repository listing.
  * **`url`** *url*  
    This REQUIRED property links to a page where users can download the tool. The URL MAY be a direct link to a download URL, or a link to documentation for how to download the tool.
  * **`title`** *string-string map*  
    This OPTIONAL property representes the name of the tool, displayed for human consumption. The title may be displayed in a repository listing, or registry tooling.  
    Title is a collection of localized strings, indexed with [ISO Language Codes][iso-lang-codes].
    * **`locale`** *string*  
      2-2 language codes representing the country and locale. [ISO Language Codes][iso-lang-codes]
    * **`title`** *string*  
Localized title. The max length MUST not exceed 30 characters and MUST not encode formatting characters.
* **`configSchemaReferenceUrl`** *url*  
This OPTIONAL property provides a schema reference for the artifact config object. The schema is provided for registry operators and tools to optionally validate and process information within the config. A registry operator MAY wish to present information, such as the OCI image architecture type. Each versioned artifact type would have a unique version, possibly referencing a unique schema version. To version the schema, the artifactType MUST also be versioned.
* **`layerMediaTypes`** string-string map  
  This REQUIRED property must have at least one value, representing one or more layer `mediaTypes` used by the artifact.  
  Layer mediaTypes SHOULD be unique to the specific artifact.  
  Layer mediaTypes are NOT REQUIRED to be unique across different artifact types when artifacts intend to share layers across different artifact tooling.  
  Registry operators MAY choose to validate layers associated with a specific artifact type. Providing the supported layers enables registry operators to know the supported `mediaTypes`.

  `layerMediaTypes` use the following format:  
  `application/vnd.`[org|company]`.`[objectType]`.`[optionalSubType]`.layer.`[version]`+`[fileFormat].
  * **`mediaType`** *string*  
    This REQUIRED property represents a valid layer `mediaTypes` for the artifact.

### Artifact Type Example

The following is an example of an unknown artifact type.

```json
{
  "mediaType": "application/vnd.oci.image.config.v1+json",
  "spec": "https://github.com/opencontainers/artifacts",
  "title": {
    "locale": "en-US",
    "title": "An unknown type"
  },
  "description": {
    "locale": "en-US",
    "description": "An undefined type - USE ONLY FOR DEVELOPMENT"
  },
  "url": "https://github.com/opencontainers/artifacts",
  "tools":[
    {
      "title": {
        "locale": "en-US",
        "title": "ORAS"
      },
      "url": "https://github.com/deislabs/oras"
    }
  ],
  "configSchemaReference": "https://raw.githubusercontent.com/SteveLasker/scratch/master/artifactTypeSchema.json",
  "layerMediaTypes": [
    "application/vnd.oci.unknown.layer.v1.bin",
    "application/vnd.oci.unknown.layer.v1.json",
    "application/vnd.oci.unknown.layer.v1.tar",
    "application/vnd.oci.unknown.layer.v1.txt",
    "application/vnd.oci.unknown.layer.v1.yaml"
  ]
}
```

## Pushing an OCI Artifact

With the artifact defined, test the artifact by pushing to [a registry that supports OCI Artifacts][implementors].

### hello-world of artifacts

For the hello-world of artifacts, create 2 empty files in a sub directory:

* `config.json`
* `hello-world.txt`

### Push the Artifact with `ORAS push`

[ORAS][oras] (**O**CI **R**egistry **A**s **S**torage) is a client and a go library designed to push and pull artifacts to an OCI Registry.

1. If the registry requires authentication:  
    ```bash
    oras login [registryLoginURL]
    ```
1. Push the hello-world artifact with the `oras push` command:
    ```bash
    oras push [registryLoginURL]/hello-world:1 \
    --manifest-config /dev/null:application/vnd.oci.samples.hello-world.config.v1+json \
    ./hello-world.txt:application/vnd.oci.samples.hello-world.layer.v1+txt
    ```
1. Pull the hello-world artifact with the `oras pull` command:
    ```bash
    mkdir pull
    cd pull
    oras pull [registryLoginURL]/hello-world:1 -a
    ```

## Optional: Publishing the Artifact Type

Artifact Types can be categorized into two major groups:

1. **Per customer, or registry operator types**: These are types that aren't *necessarily* considered interesting for others to consume. They may be unique to a specific company, or a specific vendor. In these cases, the Artifact Type definition may still be maintained, and may be imported to a given registry for support.
1. **Well Known Artifact types**: These are types that many to most registry operators would likely want to support. While registry operators are not required to support all types to be considered compliant with the [OCI Distribution Spec][distribution-spec], registry operators would likely want to support well known types, if there was an easy way to understand the differing types.  
Defining an Artifact Type with the `artifactType.json` format above, artifact authors may publish their types in this repository. Registry operators can then import the types they wish to support, including the localized string and logos.

To publish the artifact type, [create a PR](CONTRIBUTING.md) against this repository with the following format:

1. Create a sub folder under  [./artifactTypes](./artifactTypes), uniquely identifying the type and version. Use the following abbreviated format of the artifact type when naming the folder:  
  `vnd.`[org|company]`.`[objectType]`.`[optionalSubType]`.`[version]
1. Provide the following files under the new folder. Use the exact file names specified.  
    |File|Content|
    |-|-|
    |`artifactType.json`|**REQUIRED**: Information that enables registries and artifact specific tooling to represent the artifact|
    |`artifactConfigSchema.json`|**OPTIONAL**: Schema validation for the optional configuration. If a file is present, the config has schema. A missing `artifactConfigSchema.json` file states the artifact does not support configuration information.
    |`logo.svg`|**OPTIONAL**: The artifact logo, in svg format, enabling registries and tooling to associate the artifact type with its logo.

As the PR is approved, it will be available for registry operators to discover and support. You may have to contact the registry operator, or open a support ticket to have them update their supported types.

### Sample Artifacts

|Artifact|mediaType|Folder Name|
|-|-|-|
|[OCI Image](https://github.com/opencontainers/image-spec/)|`application/vnd.oci.image.config.v1+json`|[vnd.oci.image.1](./artifactTypes/vnd.oci.image.1/)
|[Unknown][artifacts-spec]|`application/vnd.unknown.config.v1+json`|[vnd.oci.unknown.1](./artifactTypes/vnd.oci.unknown.1/)|

### PR Approval Process for new Artifact Types

Discovery of well known OCI Artifacts types is key to the success of the OCI Artifact approach. The purpose of the artifact type PR approval process will focus on:

1. Is the type uniquely identified?
1. Does the owner of the PR represent the org or company by which the artifact is being submitted under.  
For example, `application/vnd.oci` is reserved for projects and types supported by the [OCI][oci], and `application/vnd.contoso` is reserved for the contoso company.
1. Is the type believed to be generally applicable for broad consumption across multiple registry operators?
1. Is there a group, entity or community to support the type over time?
1. Is the logo and text considered in good taste and within copyright rules to submit to the public for use?

The [OCI Artifact Maintainers][maintainers] will make every effort to provide guidance in approving the artifact type, in accordance with the [maintainers guide](MAINTAINERS_GUIDE.md).  
If the requester wishes to appeal a denied PR, they may appeal to the [TOB][tob]. The TOB will have the final decision on contested requests.

[artifacts-spec]:       https://github.com/opencontainers/artifacts
[containerd]:           https://containerd.io/
[distribution]:         https://github.com/docker/distribution
[distribution-spec]:    https://github.com/opencontainers/distribution-spec/
[docker]:               https://github.com/moby/moby
[def-registry]:         definitions-terms.md#registry
[def-well-known-types]: definitions-terms.md#well-known-types
[def-yass]:             definitions-terms.md#yass
[image-index]:          https://github.com/opencontainers/image-spec/blob/master/image-index.md
[image-layer]:          https://github.com/opencontainers/image-spec/blob/master/layer.md
[image-manifest]:       https://github.com/opencontainers/image-spec/blob/master/manifest.md
[image-spec]:           https://github.com/opencontainers/image-spec/
[image-spec-config]:    https://github.com/opencontainers/image-spec/blob/master/config.md
[implementors]:         implementors.md
[iso-lang-codes]:       http://www.lingoes.net/en/translator/langcode.htm
[maintainers]:          MAINTAINERS
[oras]:                 https://github.com/deislabs/oras
[oci]:                  https://opencontainers.org
[singularity]:          https://github.com/sylabs/singularity
[sylabs]:               https://sylabs.io/
[tob]:                  https://github.com/opencontainers/image-spec/tob
