# PEP 694 - Upload 2.0 API for Python Package Indexes

| Resource               | Link                                                                                                 |
| ---------------------- | ---------------------------------------------------------------------------------------------------- |
| `PEP Link`             | <https://peps.python.org/pep-0694/>                                                                  |
| `DPO Discussion`       | [PEP 694: Upload 2.0 API for Python Package Indexes - Rebooted](https://discuss.python.org/t/pep-694-pypi-upload-api-2-0/76316) |
| `DPO Discussion [old]` | [PEP 694: Upload 2.0 API for Python Package Indexes](https://discuss.python.org/t/pep-694-upload-2-0-api-for-python-package-repositories/16879) |

## Abstract

PEP 694 proposes a formal API for uploading packages to PyPI.  Currently, the only way to upload
packages is to use the [legacy API](https://peps.python.org/pep-0694/#legacy-api), which is
ill-defined and essentially a stripped down version of the old form submission API that the PyPI web
UI used to provide.

## Motivation

Ostensibly, PEP 694 aims to formalize and modernize the upload API.  In addition to that, the PEP
proposes some substantial improvements to the package upload experience.  These include:

* the use of an upload session, which can be used to simultaneously publish all wheels in a package release
* the ability to “staging” a release, which can be used to test uploads before publicly publishing
  them, without the need for test.pypi.org;
* artifacts which can be overwritten and replaced, until a session is published;
* an asynchronous and “chunked”, resumable file uploads, for more efficient use of network bandwidth;
* a detailed status on the state of artifact uploads;
* new project creation without requiring the uploading of an artifact.

## Rationale

Some of the other deficiencies of the legacy API include:

* it is fully synchronous, which forces requests to be held open both for the upload itself, and
  while the index processes the uploaded file to determine success or failure.
* it does not support any mechanism for resuming an upload. With the largest default file size on
  PyPI being around 1GB in size, requiring the entire upload to complete successfully means
  bandwidth is wasted when such uploads experience a network interruption while the request is in
  progress.
* the atomic unit of operation is a single file. This is problematic when a release logically
  includes an sdist and multiple binary wheels, leading to race conditions where consumers get
  different versions of the package if they are unlucky enough to require a package before their
  platform’s wheel has completely uploaded. If the release uploads its sdist first, this may also
  manifest in some consumers seeing only the sdist, triggering a local build from source.
* status reporting is very limited. There’s no support for reporting multiple errors, warnings,
  deprecations, etc. Status is limited to the HTTP status code and reason phrase, of which the
  reason phrase has been deprecated since HTTP/2 (RFC 7540).
* metadata for a release is submitted alongside the file. However, as this metadata is famously
  unreliable, most installers instead choose to download the entire file and read the metadata from
  there.
* there is no mechanism for allowing an index to do any sort of sanity checks before bandwidth gets
  expended on an upload. Many cases of invalid metadata or incorrect permissions could be checked
  prior to uploading files.
* there is no support for “staging” a release prior to publishing it to the index.
* creation of new projects requires the uploading of at least one file, leading to “stub” uploads to
  claim a project namespace.

## Specification

The formal protocol is too detailed to describe here; please refer to [the PEP
specification](https://peps.python.org/pep-0694/#upload-2-0-api-specification) itself for the
relevant details.

## Backward Compatibility

Once PEP 694 is implemented, the legacy API will be deprecated.  However, it is not expected to be
removed any time soon (if ever).  PEP 694 explicitly does not propose a deprecation time frame.

## Security Implications

The security of the new upload API depends on the security and permission structure of the existing
legacy API, with the assumption that its authentication and authorization services will directly
apply to the new API.

The PEP's proposed staged release preview feature can be optionally obfuscated, but it cannot be
protected behind an authentication wall.  When the client initiates an upload session, they can
provide a "nonce" which is used as input to the staged token hash algorithm.  If not provided, the
nonce defaults to the empty string, meaning that only the package name and version provide input to
the hash algorithm.
