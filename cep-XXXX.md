# CEP XXXX - Repodata Wheel Support

<table>
<tr><td> Title </td><td> Repodata Wheel Support</td>
<tr><td> Status </td><td> Draft </td></tr>
<tr><td> Author(s) </td><td>
  Travis Hathaway &lt;travis.j.hathaway@gmail.com&gt;
</td></tr>
<tr><td> Created </td><td> Dec 19, 2025</td></tr>
<tr><td> Updated </td><td> Dec 19, 2025</td></tr>
<tr><td> Discussion </td><td> https://github.com/conda/ceps/pull/ </td></tr>
<tr><td> Implementation </td><td> TBD </td></tr>
<tr><td> Requires </td>N/A</tr>
</table>

> The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
  "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as
  described in [RFC2119][RFC2119] when, and only when, they appear in all capitals, as shown here.

## Abstract

This CEP outlines how native support for Python packages (i.e. "wheels") will be achieved by adding support
for them to conda's package index (i.e. "repodata"). We cover the rationale behind this proposal as well as
alternatives that were considered along the way.


## Motivation

While conda is and remains a language agnostic packaging ecosystem, the authors of this CEP cannot deny
importance of Python in this ecosystem.  Users have long wanted a way to seamlessly install packages only
available as Python wheels into their conda compatible environments. Rather force users to deal only with
the "conda" package format, this CEP propose a way for us to meet them halfway by directly integrate 
"wheel" support into conda compatible tooling.


## Specification

### Add a new top-level key `packages.whl` to list wheels in a channel

According to the current draft schema for [repodata.json][repodata-schema], repodata consists of five top
level keys:

- repodata_version
- info
- packages
- packages.conda
- removed
- signatures

This CEP proposes the addition of new `packages.whl` to account for the wheel format. This key points to a
mapping that MUST contain [repodata record][repodata-record-schema] objects. The key of this mapping MUST be
the wheel file name, and the contents of the package record MUST contain metadata related to that wheel.

### Add a new `url_prefix` property to repodata records

**TBD**


## Examples

To illustrate, an example of this how this new section in the repodata will look showing the `requests` package:

```json
{
  "packages.whl": {
    "requests-2.32.5-py3-none-any.whl": {
      "record_version": 3,
      "name": "requests",
      "version": "2.32.5",
      "build": "py3_0",
      "build_number": 0,
      "depends": [
        "charset-normalizer <4,>=2",
        "idna <4,>=2.5",
        "urllib3 <3,>=1.21.1",
        "certifi >=2017.4.17",
        "python >=3.9"
      ],
      "fn": "requests-2.32.5-py3-none-any.whl",
      "sha256": "78820a3e5d9d3b25ce8e1c99c1c89cd19caa904a92973a3e50f8426009e8a4b3",
      "size": 6899,
      "subdir": "noarch",
      "timestamp": 1764005009,
      "noarch": "python"
    }
  }
}
```

## Rejected ideas

- Adding interoperability support for tools like pip/uv into conda compatible tooling
- Only the fly conversion of wheels to conda packages
- Build farms to perform large scale and continuous conversion of wheels to conda packages

## References

- [conda-pypi project][conda-pypi]
- [Adopting uv in pixi][uv-in-pixi]

## Copyright

All CEPs are explicitly [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).

<!-- links -->
[RFC2119]: https://datatracker.ietf.org/doc/html/rfc2119
[conda-schemas]: https://schemas.conda.org
[repodata-schema]: https://schemas.conda.org/repodata-1.schema.json
[repodata-record-schema]: https://schemas.conda.org/repodata-record-1.schema.json
[conda-pypi]: https://github.com/conda-incubator/conda-pypi
[uv-in-pixi]: https://prefix.dev/blog/uv_in_pixi