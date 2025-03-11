# PEP 759 - External Wheel Hosting

| Resource            | Link                                                                |
| ------------------- | ------------------------------------------------------------------- |
| `PEP Link`          | <https://peps.python.org/pep-0759/>                                 |
| `DPO Discussion`    | [PEP 759, External Wheel Hosting](https://discuss.python.org/t/pep-759-external-wheel-hosting/66458) |
| `Github Repository` |                                                                     |

## Summary

The intent of this PEP was to provide a safe mechanism for hosting wheels on external indexes
(i.e. other than PyPI).  It proposed a new file format called a `.rim` file which, like `.whl`
files, are zip files.  However, unlike `.whl` files, these zip files *only* contain the `.dist-info`
metadata directory, and do not contain any package contents.

The PEP proposed a new file inside this metadata directory called `EXTERNAL-HOSTING.json` which
contained some additional keys which are used to locate and verify the wheels on the external index.
The primary effect of this file was to change the download URL for the actual wheel on the original
index's Simple Index.  Hashes are included to act as a checksum so that the external artifact could
be verified to avoid bait-and-switching on the external index.

See the [PEP](https://peps.python.org/pep-0759/) for additional details.

## Resolution

As of 31-Jan-2025, the PEP was *withdrawn*.  The primary reason was that in the author's opinion,
there was little appetite from the Python packaging community to support `.rim` files.  Instead, the
consensus appeared to be to make it easier and safer to use multiple indexes, including the adoption
by index software authors for [PEP 708](https://peps.python.org/pep-0708/).

Even so, PEP 759 had its fans.  With a withdrawn status, PEP 759 is not officially rejected, and may
be reopened in the future, if its advocates can build sufficient support.
