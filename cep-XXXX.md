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

### Add a new `download_url` property to repodata records

Additionally, this CEP proposes the addition of a new `download_url` property that will give channel operators
added flexibility to inform conda compatible clients where to retrieve individual packages. Compatible
clients MAY prefer this location if it is present and MUST fall back to traditional means of fetching
if not present.

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
      "noarch": "python",
      "download_url": "https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl"
    }
  }
}
```

## Rejected ideas

### Add interoperability for tools like pip/uv to conda compatible tooling

This is currently the de facto solution for adding wheel packages to conda environments. It is the
method that most conda compatible clients (i.e. conda, pixi and mamba) use in order to allow users
to install wheel packages directly into their conda environments.

While this solution works perfectly fine for many use cases, it is currently not seen as stable
for the following reasons:

- Mixing conda and wheel packages can lead to unpredictable behavior because conda compatible
  clients cannot account for wheel packages during solves.
- Iteratively installing conda, then wheel and then conda packages into a single environment
  over time leads to instability because each tool may unknowingly overwrite what the other has
  done.

For more information on key differences between conda and wheel packages, please see the following
article from the [conda-pypi][conda-pypi] documentation:

- [Key differences between conda and PyPI](https://conda-incubator.github.io/conda-pypi/why/conda-vs-pypi/)

### On the fly conversion of wheels to conda packages (client-side)

After rejecting the previous approach, the conda maintainers pursued client side conversions
of wheels to conda packages. This work was originally laid out in the [conda-pupa][conda-pupa] project
and later integrated into the [conda-pypi][conda-pypi] project.

The method relies on creating a local channel on the user's computer that gets added to each time
a new wheel package is installed. This method was rejected for the following reasons:

- Converting these wheels locally takes time and presents unacceptable performance penalties
  for users.
- Users must rely on a local cache for installing wheels and this cache cannot easily be shared.

### Build farms for large conversion of wheel to conda formats (server-side)

The last alternative considered was establishing a build farm to for large scale conversion
of wheel to conda packages. This idea was rejected for the following reasons:

- Maintaining such a build farm present an unacceptable maintenance burden to the conda community.
- Adding native handling of wheel files to conda compatible clients is seen as feasible alternative.

## References

- [conda-pypi project][conda-pypi]
- [conda-pupa][conda-pupa]
- [Adopting uv in pixi][uv-in-pixi]

## Copyright

All CEPs are explicitly [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).

<!-- links -->
[RFC2119]: https://datatracker.ietf.org/doc/html/rfc2119
[conda-schemas]: https://schemas.conda.org
[repodata-schema]: https://schemas.conda.org/repodata-1.schema.json
[repodata-record-schema]: https://schemas.conda.org/repodata-record-1.schema.json
[conda-pypi]: https://github.com/conda-incubator/conda-pypi
[conda-pupa]: https://github.com/dholth/conda-pupa
[uv-in-pixi]: https://prefix.dev/blog/uv_in_pixi