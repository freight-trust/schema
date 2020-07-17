<pre>
 org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.name="@freight-trust/$APPLICATIONh" \
      org.label-schema.description="Freight Trust & Clearing Corporation" \
      org.label-schema.url="https://schema.freighttrust.com/" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url="https://github.com/freight-trust/$VCS_URL" \
      org.label-schema.vendor="Freight Trust & Clearing" \
      org.label-schema.version=$VERSION \
      org.label-schema.schema-version="1.0"

</pre>



# Annotations

[source](https://raw.githubusercontent.com/opencontainers/image-spec/master/annotations.md)

Several components of the specification, like [Image Manifests](manifest.md) and [Descriptors](descriptor.md), feature an optional annotations property, whose format is common and defined in this section.

This property contains arbitrary metadata.

## Rules

* Annotations MUST be a key-value map where both the key and value MUST be strings.
* While the value MUST be present, it MAY be an empty string.
* Keys MUST be unique within this map, and best practice is to namespace the keys.
* Keys SHOULD be named using a reverse domain notation - e.g. `com.example.myKey`.
* The prefix `org.opencontainers` is reserved for keys defined in Open Container Initiative (OCI) specifications and MUST NOT be used by other specifications and extensions.
* Keys using the `org.opencontainers.image` namespace are reserved for use in the OCI Image Specification and MUST NOT be used by other specifications and extensions, including other OCI specifications.
* If there are no annotations then this property MUST either be absent or be an empty map.
* Consumers MUST NOT generate an error if they encounter an unknown annotation key.

## Pre-Defined Annotation Keys

This specification defines the following annotation keys, intended for but not limited to [image index](image-index.md) and image [manifest](manifest.md) authors:
* **org.opencontainers.image.created** date and time on which the image was built (string, date-time as defined by [RFC 3339](https://tools.ietf.org/html/rfc3339#section-5.6)).
* **org.opencontainers.image.authors** contact details of the people or organization responsible for the image (freeform string)
* **org.opencontainers.image.url** URL to find more information on the image (string)
* **org.opencontainers.image.documentation** URL to get documentation on the image (string)
* **org.opencontainers.image.source** URL to get source code for building the image (string)
* **org.opencontainers.image.version** version of the packaged software
  * The version MAY match a label or tag in the source code repository
  * version MAY be [Semantic versioning-compatible](http://semver.org/)
* **org.opencontainers.image.revision** Source control revision identifier for the packaged software.
* **org.opencontainers.image.vendor** Name of the distributing entity, organization or individual.
* **org.opencontainers.image.licenses** License(s) under which contained software is distributed as an [SPDX License Expression][spdx-license-expression].
* **org.opencontainers.image.ref.name** Name of the reference for a target (string).
  * SHOULD only be considered valid when on descriptors on `index.json` within [image layout](image-layout.md).
  * Character set of the value SHOULD conform to alphanum of `A-Za-z0-9` and separator set of `-._:@/+`
  * The reference must match the following [grammar](considerations.md#ebnf):
    ```
    ref       ::= component ("/" component)*
    component ::= alphanum (separator alphanum)*
    alphanum  ::= [A-Za-z0-9]+
    separator ::= [-._:@+] | "--"
    ```
* **org.opencontainers.image.title** Human-readable title of the image (string)
* **org.opencontainers.image.description** Human-readable description of the software packaged in the image (string)

## Back-compatibility with Label Schema

[Label Schema](https://label-schema.org) defined a number of conventional labels for container images, and these are now superceded by annotations with keys starting **org.opencontainers.image**.

While users are encouraged to use the **org.opencontainers.image** keys, tools MAY choose to support compatible annotations using the **org.label-schema** prefix as follows.

| `org.opencontainers.image` prefix | `org.label-schema` prefix | Compatibility notes |
|---------------------------|-------------------------|---------------------|
| `created` | `build-date` | Compatible |
| `url` | `url` | Compatible |
| `source` | `vcs-url` | Compatible |
| `version` | `version` | Compatible |
| `revision` | `vcs-ref` | Compatible |
| `vendor` | `vendor` | Compatible |
| `title` | `name` | Compatible |
| `description` | `description` | Compatible |
| `documentation` | `usage` | Value is compatible if the documentation is located by a URL |
| `authors` |  | No equivalent in Label Schema |
| `licenses` | | No equivalent in Label Schema |
| `ref.name` | | No equivalent in Label Schema |
| | `schema-version`| No equivalent in the OCI Image Spec |
| | `docker.*`, `rkt.*` | No equivalent in the OCI Image Spec |

[spdx-license-expression]: https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60

## OCI Image Manifest Specification

There are three main goals of the Image Manifest Specification.
The first goal is content-addressable images, by supporting an image model where the image's configuration can be hashed to generate a unique ID for the image and its components.
The second goal is to allow multi-architecture images, through a "fat manifest" which references image manifests for platform-specific versions of an image.
In OCI, this is codified in an [image index](image-index.md).
The third goal is to be [translatable](conversion.md) to the [OCI Runtime Specification](https://github.com/opencontainers/runtime-spec).

This section defines the `application/vnd.oci.image.manifest.v1+json` [media type](media-types.md).
For the media type(s) that this is compatible with see the [matrix](media-types.md#compatibility-matrix).

### Image Manifest

Unlike the [image index](image-index.md), which contains information about a set of images that can span a variety of architectures and operating systems, an image manifest provides a configuration and set of layers for a single container image for a specific architecture and operating system.

#### *Image Manifest* Property Descriptions

- **`schemaVersion`** *int*

  This REQUIRED property specifies the image manifest schema version.
  For this version of the specification, this MUST be `2` to ensure backward compatibility with older versions of Docker. The value of this field will not change. This field MAY be removed in a future version of the specification.

- **`mediaType`** *string*

  This property is *reserved* for use, to [maintain compatibility](media-types.md#compatibility-matrix).
  When used, this field contains the media type of this document, which differs from the [descriptor](descriptor.md#properties) use of `mediaType`.

- **`config`** *[descriptor](descriptor.md)*

    This REQUIRED property references a configuration object for a container, by digest.
    Beyond the [descriptor requirements](descriptor.md#properties), the value has the following additional restrictions:

    - **`mediaType`** *string*

        This [descriptor property](descriptor.md#properties) has additional restrictions for `config`.
        Implementations MUST support at least the following media types:

        - [`application/vnd.oci.image.config.v1+json`](config.md)

        Manifests concerned with portability SHOULD use one of the above media types.

- **`layers`** *array of objects*

    Each item in the array MUST be a [descriptor](descriptor.md).
    The array MUST have the base layer at index 0.
    Subsequent layers MUST then follow in stack order (i.e. from `layers[0]` to `layers[len(layers)-1]`).
    The final filesystem layout MUST match the result of [applying](layer.md#applying-changesets) the layers to an empty directory.
    The [ownership, mode, and other attributes](layer.md#file-attributes) of the initial empty directory are unspecified.

    Beyond the [descriptor requirements](descriptor.md#properties), the value has the following additional restrictions:

    - **`mediaType`** *string*

        This [descriptor property](descriptor.md#properties) has additional restrictions for `layers[]`.
        Implementations MUST support at least the following media types:

        - [`application/vnd.oci.image.layer.v1.tar`](layer.md)
        - [`application/vnd.oci.image.layer.v1.tar+gzip`](layer.md#gzip-media-types)
        - [`application/vnd.oci.image.layer.nondistributable.v1.tar`](layer.md#non-distributable-layers)
        - [`application/vnd.oci.image.layer.nondistributable.v1.tar+gzip`](layer.md#gzip-media-types)

        Manifests concerned with portability SHOULD use one of the above media types.
        An encountered `mediaType` that is unknown to the implementation MUST be ignored.


        Entries in this field will frequently use the `+gzip` types.

- **`annotations`** *string-string map*

    This OPTIONAL property contains arbitrary metadata for the image manifest.
    This OPTIONAL property MUST use the [annotation rules](annotations.md#rules).

    See [Pre-Defined Annotation Keys](annotations.md#pre-defined-annotation-keys).

#### Example Image Manifest

*Example showing an image manifest:*
```json,title=Manifest&mediatype=application/vnd.oci.image.manifest.v1%2Bjson
{
  "schemaVersion": 2,
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "size": 7023,
    "digest": "sha256:b5b2b2c507a0944348e0303114d8d93aaaa081732b86451d9bce1f432a537bc7"
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 32654,
      "digest": "sha256:9834876dcfb05cb167a5c24953eba58c4ac89b1adf57f28f2f9d09af107ee8f0"
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 16724,
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b"
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 73109,
      "digest": "sha256:ec4b8955958665577945c89419d1af06b5f7636b4ac3da7f12184802ad867736"
    }
  ],
  "annotations": {
    "com.example.key1": "value1",
    "com.example.key2": "value2"
  }
}
```
