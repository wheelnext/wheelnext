# PEP 777 - How to Re-invent the Wheel

| Resource            | Link                                                                    |
| ------------------- | ----------------------------------------------------------------------- |
| `PEP Link`          | <https://peps.python.org/pep-0777/>                                     |
| `DPO Discussion`    | [PEP 777: How to Re-invent the Wheel](https://discuss.python.org/t/pep-777-how-to-re-invent-the-wheel/67484) |
| `Github Repository` |                                                                         |

## Abstract

The Python wheel format has been a cornerstone of Python packaging for over a decade, providing a
reliable way to distribute Python packages. However, as the Python ecosystem has evolved, there's
been growing demand for new wheel features. This PEP addresses a fundamental challenge: how to safely
introduce backwards-incompatible improvements to the wheel specification. By establishing clear
compatibility requirements for future wheel revisions, this PEP paves the way for other proposals
to enhance the wheel format. While this PEP doesn't introduce a new wheel version itself, it creates
the framework needed for future improvements to the wheel specification.

## Rationale

The current wheel specification faces a significant challenge: any changes that require new installer
behavior must increment the major version number, which can cause widespread installation failures. This
is because older installers **MUST** reject wheels with unsupported major versions, potentially affecting
many users who haven't updated their tools.

This strict requirement has blocked several valuable improvements to the wheel format, including:
- Better compression methods
- Enhanced data format capabilities
- Improved wheel content information
- Modern JSON-formatted metadata

The current situation creates a catch-22: while the wheel format needs to evolve to support new features,
the risk of breaking existing installations makes it difficult to make any backwards-incompatible changes.
This PEP solves this problem by establishing clear rules for how new wheel versions should handle
compatibility, ensuring that only tools explicitly designed for newer wheel versions will be affected by
changes.

## Specification

This PEP introduces several key changes to how wheel versions are handled:

### Core Metadata Changes
- A new `Wheel-Version` field will be added to the Core Metadata Specification
- This field must match the version specified in the wheel's `WHEEL` file
- If `Wheel-Version` is absent, tools must assume the wheel version is 1.0
- This field is not allowed in source distribution metadata

### Installation Requirements
Installers must follow these steps:
1. Verify that `Wheel-Version` matches between core metadata and wheel metadata
2. Check installer compatibility with the wheel version
3. Proceed with installation according to the Binary Distribution Format specification

### Resolver Behavior
- Resolvers must check and filter out incompatible wheel versions
- When multiple compatible wheels exist, prioritize the highest compatible version
- Installers should warn users when skipping incompatible wheels

### File Extension Change
- Future wheel versions (2.0 and beyond) will use the `.whlx` extension instead of `.whl`
- This change allows for immediate adoption of new wheel features without breaking existing installers

### Compatibility Requirements
Future wheel revisions must maintain these compatibility promises:
- Stay compatible with `importlib.metadata` for all supported CPython versions
- Maintain ZIP format as the outer container
- Keep `.dist-info` metadata directory at the root of the zip archive without further compression
- Only use compression formats available in the CPython standard library

## Backward Compatibility

Backward compatibility is crucial for the successful evolution of the wheel format. If adopting new
wheel versions causes problems such as installation failures and CI pipeline issues, package creators
will be reluctant to adopt new standards.

The PEP includes several key features designed to maintain compatibility. Resolvers will proactively
filter out incompatible wheel versions before installation attempts, preventing failed installations.
Future wheel revisions will maintain compatibility with the standard library's `importlib.metadata`,
ensuring that package installations continue to work on older supported CPython revisions.

One important compatibility limitation applies to projects that publish only new wheel versions alongside
source distributions. In these cases, users with older installers may need to build from source, which often
presents challenges. To address this, projects may need to consider dual-publishing strategies during the initial
migration period.

## Rejected Ideas

Several alternative approaches were considered but ultimately rejected:

### Keeping the Current Wheel Format
While the wheel format has served well for over a decade, modern Python packages have evolved
significantly. The current format can't efficiently handle larger packages with Rust or C extensions,
nor can it properly express modern hardware acceleration requirements. Better compression and finer-grained
compatibility tagging are needed to handle scaling of package indices.

### Tying Wheel Changes to CPython Releases
This approach would make adoption predictable but has significant drawbacks:
- Slower adoption for users on LTS systems
- Unfair or unclear to users of alternative Python implementations
- Many wheel improvements don't actually require Python version changes

### Keeping the `.whl` Extension
While appealing, this would cause problems:
- Current installers would try to install incompatible wheels, raising confusing errors
- The current filename format is too restrictive for future changes such as variants
- Build tag placement makes extending the wheel filename ambiguous

### Version-Specific Extensions (`.whl2`, `.whl3`, etc.)
Using version-specific extensions like `.whl2` or `.whl3` would eliminate the need for `Wheel-Version`
metadata and allow different wheel versions to coexist side by side. However, this approach comes with
significant drawbacks: it would require updating system file associations for each new version, reinforce
the problematic practice of storing metadata in filenames, and could cause confusion if the file extension
and internal version numbers diverge.

### Changing the Outer Container Format
While changing the wheel format to something like `tar.zst` could improve compression, this approach
would require non-standard library dependencies. Additionally, making such a fundamental change to the
container format would make it harder to iteratively adopt changes to wheels. Future improvements can
achieve similar benefits within the existing ZIP format, making this change unnecessary.

### Including Wheel 2.0 Specification
This PEP intentionally focuses solely on establishing compatibility rules, leaving specific
wheel 2.0 features for future proposals. This separation allows for better discussion of
individual features and maintains focus on the core compatibility framework.

## Open issues

- Is it better to break users every change if they can tell new wheels are available?
- Should we delay publication to reduce number of users broken if we don’t ignore incompatible wheels by default?
- How can we emphasize disruptions if wheel updates do “break the world”?
- How best to signal new wheels that are incompatible exist to users?
