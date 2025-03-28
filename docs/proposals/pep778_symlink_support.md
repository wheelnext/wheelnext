# PEP 778 - Supporting Symlinks in Wheels

| Resource            | Link                                                                         |
| ------------------- | ---------------------------------------------------------------------------- |
| `PEP Link`          | <https://github.com/python/peps/pull/3786>                                   |
| `DPO Discussion`    | [PEP 778: Supporting Symlinks in Wheels](https://discuss.python.org/t/pep-778-supporting-symlinks-in-wheels/53824) |
| `Github Repository` |                                                                              |

## Abstract

Wheels currently do not handle symlinks well, copying content instead of making symlinks when installed. To properly
handle distributing libraries in wheels, we propose a new `LINKS` metadata file to handle symlinks in a platform
portable manner. This specification requires a new wheel major version, discussed in [PEP 777 - How to Re-Invent The Wheel](site:proposals/pep777_how_to_reinvent_the_wheel/).

## Motivation

Symlinks in wheels are currently created as copies of files due to security reasons in the `zipfile` module. This
presents problems for projects shipping large compiled libraries, as they must either increase install size or omit
symlinks, potentially breaking downstream use cases. Proper symlink handling is essential for Unix loader and linker
conventions, which require "soname," "real name," and "linker name" files. Popular projects like `numpy`, `scipy`, and
`pyarrow` would benefit from symlink support to avoid conflicts with system libraries.

## Rationale

To support the three main namings of a library used in loading and linking on Unix, we propose adding symlink support in
Python wheels. A new `LINKS` metadata file in the `.dist-info` directory will track symlinks, allowing for
cross-platform symlink-like usage. This approach accommodates platforms like Windows, where symlinks may require special
permissions. The `LINKS` file will enable installers to use alternative methods like junctions on Windows.

## Specification

### Wheel Major Version Bump

This PEP requires a wheel major version bump to at least version `2.0` to ensure older installers do not fail silently.

### New `LINKS` Metadata File

The `LINKS` file format is `source_path,target_path`, where `source_path` is relative to the root of any namespace or
package root in the wheel, and `target_path` is a non-dangling path in the wheel.

### Installer Behavior Specification

Installers must:

1. Check for the existence of a `LINKS` file.
2. Extract all files in the wheel packages and data directory.
3. Verify that `target_path` exists in the package namespaces.
4. Ensure the installer can create a link for each pair.
5. Add a platform-relevant link between `source_path` and `target_path`.

Installers must not copy files instead of generating symlinks by default.

### Build Backend Specification

Build backends must treat symlinks like their targets, verify no dangling symlinks, and recognize platform-relevant symlinks.

## Backward Compatibility

Introducing symlinks requires a wheel format major version increment, causing new wheels to raise errors on older
installer tools.

## Security Implications

Symlinks can be dangerous if not handled carefully. Installers must ensure symlinks do not point outside packages, are
not dangling, and are not cyclical. Symlinks should not be followed on removal.

## How to Teach This

End users should experience symlink benefits transparently. Installers should provide clear error messages if symlinks
are unsupported. Documentation on `packaging.python.org` should describe symlink use cases and caveats.

## Reference Implementation

TODO

## Rejected Ideas

- **Just Use Unix Symlinks Everywhere**: Future PEPs should support Windows, potentially using junctions.
- **Don't Use Junctions in `LINKS`**: Junctions support folder symlinks on Windows, useful for future PEPs.
- **Put Symlinks in the `RECORD` Metadata File**: This would clutter the `RECORD` file.
- **Library Maintainers Should Use Python to Locate Libraries**: Some libraries require loader dependencies.
- **Include Support for Hardlinks**: This is left for a future PEP.

## Open Issues

- **[PEP 660](https://peps.python.org/pep-0660/) and Deferring Editable Installation Support**: Should this PEP specify
editable installation mechanisms?
- **Security**: Are additional restrictions needed to protect users?
- **Allow Inter-Package Symlinks**: Useful for sharding dependencies between wheels.
- **The Format of `LINKS`**: Is there a better format than the current one derived from `RECORD`?
