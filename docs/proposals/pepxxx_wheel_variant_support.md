# PEP ### - Wheel Variants - Extending Platform Awareness

| Resource             | Link                                                                                                                                     |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `PEP Link`           | `To Be Published`                                                                                                                        |
| `DPO Discussion`     | [Implementation variants: rehashing and refocusing](https://discuss.python.org/t/implementation-variants-rehashing-and-refocusing/54884) |
| `Github Repository`  | <https://github.com/wheelnext/pep_xxx_wheel_variants>                                                                                    |
| `Demo / Wheel Index` | <https://wheelnext.github.io/variants-index/>                                                                                            |

## Abstract

The Python wheel packaging format uses platform
[compatibility tags](https://packaging.python.org/en/latest/specifications/platform-compatibility-tags/) to specify
wheel's supported environments based on Python version, ABI, and platform (operating system, architecture, core system
libraries). These tags are not able to express features of modern hardware. This is particularly challenging for the
scientific computing, artificial intelligence (AI), machine learning (ML), and high-performance computing communities,
where packages are often built with specific hardware accelerations (e.g., NVIDIA CUDA, AMD ROCm), specialized CPU
instructions (e.g., AVX512_BF16), or other system dependencies.

This PEP proposes "Wheel Variants," a backward-compatible extension to the wheel specification
(PEP [427](https://peps.python.org/pep-0427/) & [491](https://peps.python.org/pep-0491/)). This extension introduces a
mechanism for package maintainers to declare multiple build variants for the same version and standard
[compatibility tags](https://packaging.python.org/en/latest/specifications/platform-compatibility-tags/) (as defined
by [PEP 425](https://peps.python.org/pep-0425/), and later extended by PEPs [513](https://peps.python.org/pep-0513/),
[571](https://peps.python.org/pep-0571/), [599](https://peps.python.org/pep-0599/),
[600](https://peps.python.org/pep-0600/), [656](https://peps.python.org/pep-0656/),
[730](https://peps.python.org/pep-0730/), [738](https://peps.python.org/pep-0738/)) while allowing installers to
automatically select the most appropriate variant based on system hardware and software capabilities and
characteristics.

To enable fine-grained package selection without fragmenting the Python ecosystem, this PEP proposes:

- An evolution of the wheel format called **Wheel Variant** that allows wheels to be distinguished by hardware or
software attributes.

- A **variant provider plugin** interface allowing to dynamically detect platform attributes and recommend the most
suitable wheel.

- A **hash-based identification mechanism** for wheel variants, ensuring not breaking the current packaging ecosystem
while allowing quick visual identifications of which Python wheel artifact corresponds to what.

The goal is to simplify the user experience to a familiar `pip install <package>`, while ensuring optimal
performance and compatibility.

This approach allows seamless package resolution without requiring intrusive changes to installers, ensures backward
compatibility, and minimizes the burden on package maintainers.

## Definitions

Most of the definitions are borrowed from [PEP 440](https://peps.python.org/pep-0440/#definitions)

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
[RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119.html).

- **Projects** are software components that are made available for integration. Projects include Python libraries,
frameworks, scripts, plugins, applications, collections of data or other resources, and various combinations thereof.
Public Python projects are typically registered on the [Python Package Index](https://pypi.org/).

- **Releases** are uniquely identified snapshots of a project.

- **Distributions** are the packaged files which are used to publish and distribute a release.

- **Build tools** are automated tools intended to run on development systems, producing source and binary distribution
archives. Build tools may also be invoked by integration tools in order to build software distributed as sdists rather
than prebuilt binary archives.

- **Index servers** are active distribution registries which publish version and dependency metadata and place
constraints on the permitted metadata.

- **Publication tools** are automated tools intended to run on development systems and upload source and binary
distribution archives to index servers.

- **Installation tools** are integration tools specifically intended to run on deployment targets, consuming source and
binary distribution archives from an index server or other designated location and deploying them to the target system.

- **Automated tools** is a collective term covering build tools, index servers, publication tools, integration tools and
any other software that produces or consumes distribution version and dependency metadata.

## Motivation

The Python packaging ecosystem has evolved to support increasingly diverse computing environments. The current software
ecosystem often relies on platform specific features to pick which binaries are compatible with a particular computer.
Unfortunately the current wheel format cannot adequately express the features  of modern hardware. This limitation
forces package authors into suboptimal distribution strategies and creates friction for users attempting to install
performance-critical packages.

Existing approaches to handling Python packages with more complex platform requirements are suboptimal (explored below
in greater details). Some methods include maintaining separate package indexes for different hardware configurations,
bundling all potential variants into a single "mega-wheel" / "monolithic wheel" or using separate package names
(`mypackage-gpu`, `mypackage-cpu`). Each of these approaches has significant drawbacks, such as excessive binary size,
dependency confusion, and inefficient dependency resolution, complex documentation, etc.

According to the [2024 Python Developers Survey](https://lp.jetbrains.com/python-developers-survey-2024/#purposes-for-using-python),
a significant portion of respondents over the last years have been successively using Python for scientific computing
purposes, covering such areas as Data analysis (steadily over 40% respondents), Machine learning (grown to 40% in 2024),
Data engineering (around 30%), and more. Many of these use cases are directly impacted by suboptimal packaging.

This issue is often crossing the boundaries of scientific computing - as highlighted in the following issue:
[manylinux_2_34 x86-64 builds produce binaries that are not compatible with all x86-64 CPUs](https://github.com/pypa/manylinux/issues/1725),
where `manylinux_2_34_x86_64` now implicitly requires `x86_64-v2` with no support for other x86 version. This
lack of support results in complexity in managing platform-specific dependencies and compatibility. That complexity
affects the installation process for users and increases the maintenance burden for package authors. The `x86_64`
compiler flags further emphasizes the urgent need for a more expressive and efficient solution in the Python packaging
ecosystem.

This PEP proposes a systematic and scalable approach to selecting optimized wheels based on platform characteristics,
which will help Python’s usage expand across diverse computing environments, from cloud computing to embedded systems
and AI accelerators.

### Rationale and User Stories

- A user wants to install a version of NumPy that is specialized for their CPU architecture.

- A user wants to install a version of PyTorch that is specialized for their GPU architecture.

- A user wants to install a version of mpi4py that has certain features enabled
  (e.g. specific MPI implementations for their hardware).

- A library maintainer wants to build their library for wasm32-wasi with and without pthreads support.

- A library maintainer wants to build their library for an Emscripten platform for Pyodide with extensions for graphics
compiled in.

- A library maintainer wants to provide packages of their game library using different graphics backends.

- SciPy wants to provide packages built against different BLAS libraries, like OpenBLAS and Accelerate on macOS. This is
something they indirectly do today

### The Limitations of Platform Compatibility Tags

The current wheel format encodes compatibility through three platform tags:

- **Python tag:** [The Python tag indicates the implementation and version required by a distribution](https://packaging.python.org/en/latest/specifications/platform-compatibility-tags/#python-tag)
(e.g., `cp313`)  
- **ABI tag:** [The ABI tag indicates which Python ABI is required by any included extension modules](https://packaging.python.org/en/latest/specifications/platform-compatibility-tags/#abi-tag)
(e.g., `cp313`)
- **Platform tag**: [Operating system and architecture - In its simplest form: sysconfig.get_platform()](https://packaging.python.org/en/latest/specifications/platform-compatibility-tags/#platform-tag)
(e.g., `linux_x86_64`)

While these tags effectively handle traditional compatibility dimensions, they cannot express modern requirements:

**GPU Accelerated Frameworks:** A wheel filename like `torch-2.8.0-cp313-cp313-manylinux_2_28_x86_64.whl`
provides no indication whether it contains NVIDIA CUDA support, AMD ROCm support, or is CPU-only. Users cannot determine
compatibility with their GPU hardware or drivers.

**CPU Instruction Sets:** A wheel filename like
`numpy-2.3.2-cp313-cp313-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl` provides no indication whether it contains
CPUs optimized instructions ranging from basic x86-64 to modern processors with AVX512, SHA-NI, and other specialized
instructions. Packages cannot indicate whether they require or benefit from specific CPU features. In turns having to
rely on the lowest common denominator forces to leave performance on the table.

**Runtime Dependencies:** Scientific computing packages often depend on specific BLAS implementations (OpenBLAS vs
Intel MKL), MPI providers (OpenMPI vs MPICH), or other system libraries that affect both functionality and performance.
The current Wheel format is not able to encode that dependency.

This lack of flexibility has led many projects to find sub-optimal - yet necessary - workarounds. Such as this manual
selector provided by the PyTorch team. This complexity represents a fundamental scalability issue with the current tag
system.

![PyTorch Wheel Selector](site:assets/wheel_variants/pytorch_selector.png)

**Source:** [https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/) (screen capture date: 2025/08/22)

This problem is not unique to PyTorch. Projects like JAX, NumPy, SciPy, Scikit-Learn and many others in the scientific
Python ecosystem face similar hurdles. The core issue is that wheel tags, while successful, are not extensible enough
to handle the combinatorial complexity of build options.

### Current Workarounds and Their Limitations

Package maintainers have developed various strategies to work around aforementioned limitations. However, each approach
has significant drawbacks.

These workarounds place a significant burden on both end-users and package maintainers. Users must understand their
hardware and software environment in detail to select the correct installation command. Maintainers must manage complex
build matrices and provide extensive documentation on how to install their packages correctly.

The Wheel Variants proposal aims to solve this problem at a fundamental level within the packaging ecosystem, providing
a standardized, automated, and user-friendly solution.

#### "Separate Package Indexes" as Variants

Projects like [PyTorch](https://pytorch.org/get-started/locally/), [RAPIDS](https://docs.rapids.ai/install/#selector)
and other packages currently distribute packages that approximate "variants" through separate package indexes with
custom URLs.

```bash
pip install torch --index-url="https://download.pytorch.org/whl/cu129"
```

**This approach requires:**

- Manual selection of artifacts based on hardware and software factors  
- Complex installation instructions  
- Separate infrastructure maintenance  
- Potential security issues when combining multiple indexes  
- Separate index for every combination of compatible features (e.g. GPU variants with different levels of CPU
optimizations)

**Induced Security Risk:** This approach has unfortunately led to supply chain attacks - More details on
[PyTorch Blog](https://pytorch.org/blog/compromised-nightly-dependency/).  It’s a non-trivial problem to address which
has forced the PyTorch to create a complete mirror of all their dependencies. Which is one of the core motivations
behind [PEP 766](https://peps.python.org/pep-0766/).

The complexity of configuration often leads to projects providing ad-hoc installation instructions rather than covering
permanent settings. This can lead to users being unable to cleanly upgrade the packages, or the upgraded packages being
reverted to the default variant on upgrades.

#### "Package Name" as Variants

Some packages use different names for variants (e.g.,
[`xgboost` - NVIDIA CUDA accelerated, `xgboost-cpu`](https://xgboost.readthedocs.io/en/stable/install.html)). However,
this creates dependency management challenges when multiple packages require the same underlying library with different
acceleration support.

Commonly, these packages install overlapping files. Since Python packaging does not support expressing that two packages
are mutually exclusive, installers can install both of them to the same environment, with the package installed second
overwriting files from the one installed first. This could lead to unpredictable behavior, including the possibility of
incidentally switching between variants depending on upgrade ordering.

Additional limitation of this approach is that publishing a new release synchronously across multiple package names is
not currently possible. [PEP 694](https://peps.python.org/pep-0694/) proposes adding such a mechanism for multiple
wheels within a single package, but extending it to multiple packages is not a goal.

**Induced Security Risk:** proliferation of suffixed variant packages leads users to expect these suffixes in other
packages, making name squatting much easier. For example, one could create a malicious `numpy-cuda` package that users
will be lead to believe it’s a CUDA variant of NumPy.

```bash
pip install xgboost      # NVIDIA GPU variant
pip install xgboost-cpu  # CPU-only variant
```

[cupy](https://github.com/cupy/cupy) for diverse reasons had to build a total of 52 different packages - all with
different names - which clearly highlights the limit of such an approach. End users need to carefully read the `CuPy`
installation documentation to figure out which package they need. And for maintainers it’s labor-intensive to
continuously have to create new PyPI packages, ask for limit increases, and keep their wheel build infrastructure and
documentation in sync with those new package names.

```bash
cupy-cuda100 cupy-cuda101 cupy-cuda102 cupy-cuda110 cupy-cuda111 cupy-cuda112 cupy-cuda113 cupy-cuda114 cupy-cuda115 
cupy-cuda116 cupy-cuda117 cupy-cuda118 cupy-cuda119 cupy-cuda11x cupy-cuda120 cupy-cuda121 cupy-cuda122 cupy-cuda123 
cupy-cuda124 cupy-cuda125 cupy-cuda126 cupy-cuda127 cupy-cuda128 cupy-cuda129 cupy-cuda12x cupy-cuda13x cupy-cuda70 
cupy-cuda75 cupy-cuda80 cupy-cuda90 cupy-cuda91 cupy-cuda92 cupy-rocm-4-0 cupy-rocm-4-1 cupy-rocm-4-2 cupy-rocm-4-3 
cupy-rocm-4-4 cupy-rocm-4-5 cupy-rocm-5-0 cupy-rocm-5-1 cupy-rocm-5-2 cupy-rocm-5-3 cupy-rocm-5-4 cupy-rocm-5-5 
cupy-rocm-5-6 cupy-rocm-5-7 cupy-rocm-5-8 cupy-rocm-5-9 cupy-rocm-6-0 cupy-rocm-6-1 cupy-rocm-6-2 cupy-rocm-6-3
```

#### "Extra-Dependency" as Variants

[JAX](https://docs.jax.dev/en/latest/installation.html) uses a plugin-based approach. The central `jax` package provides
a number of extras that can be used to install additional plugins, e.g. `jax[cuda12]` or `jax[tpu]`. This is far from
ideal as `pip install jax` (with no extra) would provide a broken install for everybody and consequently dependency
chains, a fundamental expected behavior in the Python ecosystem is dysfunctional.

JAX includes 12 extra selectors to cover all use cases - many of which overlap and could be misleading to users if they
don’t read in detail the documentation.

It should be noted that most of these "extras" are technically mutually exclusive, though it is currently impossible to
correctly express this incompatibility within the package metadata.

```yaml
Provides-Extra: minimum-jaxlib
Provides-Extra: cpu
Provides-Extra: ci
Provides-Extra: tpu
Provides-Extra: cuda
Provides-Extra: cuda12
Provides-Extra: cuda13
Provides-Extra: cuda12-local
Provides-Extra: cuda13-local
Provides-Extra: rocm
Provides-Extra: k8s
Provides-Extra: xprof
```

#### Bundled Universal Packages - Monolithic Builds

Including all possible variants in a single wheel is another option, but this leads to excessively large artifacts,
wasting bandwidth and slower installation times for users who only need one specific variant. In some cases, such
artifacts cannot be hosted on PyPI because they exceed its size limits.

#### Wheel variant selection via source distribution

[Flash-attention](https://github.com/Dao-AILab/flash-attention) does not publish wheels on PyPI at all, but instead
publishes a customized source distribution that performs platform detection, downloads the appropriate wheel from
upstream server, and then provides it to the installer. Such an approach can provide a good end-user experience by
selecting the most optimal variant automatically. However, it prevents `--only-binary` installs from working and
requires downloading source distribution and running its build phase. In this case, it also requires running with
`--no-build-isolation`. It requires hosting wheels separately, and does not provide for uniform experience across the
ecosystem.

**Induced Security Risk:** similarly to regular source builds, this model requires running arbitrary code at install
time.

#### Ecosystem Fragmentation

The lack of standardized variant support has led to ecosystem fragmentation:

**Inconsistent User Experience**: Each package uses different installation methods, creating confusion and reducing
discoverability.

**Development Tool Complications**: Installation tools, IDEs, and CI/CD systems struggle to handle non-standard
installation requirements.

**Documentation Burden**: Maintainers must create and maintain complex installation guides and users must read them. If
they don’t know or don’t take the time to read it - almost certainly their install will be dysfunctional.

### Impact on Scientific Computing and AI/ML Workflows

**TODO: Let’s insert as many quotes as possible from the community**

The packaging limitations particularly affect scientific computing and AI/ML applications where performance optimization
is critical.

#### Heterogeneous Computing Environments

Research institutions and cloud providers often manage heterogeneous computing clusters with different architectures
(CPU, Hardware accelerators, ASICS, etc.). The current system requires environment-specific installation procedures,
making reproducible deployment difficult. This situation also contributes to making "scientific papers" difficult to
reproduce.

#### Artificial intelligence, Machine learning, and Deep learning

The recent advances in modern AI workflows increasingly rely on GPU acceleration, but the current packaging system makes
deployment complex and adds a significant burden on open source developers of the entire tool stack (from build backends
to installers, not forgetting the package maintainers).

## Motivation Summary

As highlighted in the previous section, the current Python packaging system cannot adequately serve the needs of modern
heterogeneous computing environments. These aforementioned limitations force package authors into complex workarounds
that create friction for users, increase maintenance burden, and fragment the ecosystem.

**Wheel Variants provide a standardized solution that:**

- Enables automatic hardware-appropriate package selection  
- Maintains full backward compatibility with existing tools (by guaranteeing to not break non-variant aware installers,
tools, and indexes)  
- Reduces package maintenance complexity by providing a unified and flexible answer to the problem  
- Improves user experience through a consistent experience that requires little to no user inputs.  
- Supports the full spectrum of modern computing hardware  
- Provides a future-proof and flexible system that can evolve with the ecosystem and future use cases.

### Out-of-scope features

This PEP tries to present the minimal scope required and leaves aspects to tools to evolve. A non-exhaustive list:

- The format of the static variants file, and how to include them in a pylock.toml  
- The list of variant provider that is vendored or re-implemented, as well as opt-in mechanisms  
- How to instruct build backends to emit variants through the PEP 517 mechanism. For backwards compatibility, build
backends have to default to non-variant builds

## Wheel Variant Glossary

This section focuses specifically on the vocabulary used by the proposed "Wheel Variant" standard:

- **Variant Wheels**: Wheels that share the same distribution name, version, build number, and platform compatibility
tags, but are distinctly identified by an arbitrary set of variant properties.

- **Variant Namespace**: An identifier used to group related features provided by a single plugin (e.g., `nvidia`,
`x86_64`, `arm`, etc.).

- **Variant Feature**: A specific characteristic (key) within a namespace (e.g., `version`, `avx512_bf16`, etc.) that
can have one or more values.

- **Variant Property**: A 3-tuple (`namespace :: feature-name :: feature-value`) describing a single specific feature
and its value. If a feature has multiple values, each is represented by a separate property.

- **Variant Label**: A string added to the wheel filename to uniquely identify variants. A string up to 16 characters.

- **Null Variant**: A special variant with zero variant properties and the reserved label `null`. Always considered
supported but has the lowest priority among wheel variants, while being preferably chosen over non-variant wheels.

- **Variant Provider (Plugin)**: A provider of supported and valid variant properties for a specific namespace, usually
in the form of a Python package that implements system detection.

## Prior Art - Existing Solution - Within and  Beyond Python

This problem is not unique to the Python ecosystem, different groups and ecosystems have come up with various answers to
that very problem. This section will focus on highlighting the strengths and weaknesses of the different approaches
taken by various communities.

### Conda - Conda-Forge

The project that will come to most people’s mind is [conda / conda-forge](https://conda.org/), TO BE FOLLOWED BY MICHAEL

### Spack / Archspec

TO BE ADDED

### Docker / Kubernetes / Container

TO BE ADDED

### Homebrew: Bottle DSL (Domain Specific Language)

TO BE ADDED: [https://docs.brew.sh/Bottles#bottle-dsl-domain-specific-language](https://docs.brew.sh/Bottles#bottle-dsl-domain-specific-language)

### Nix / Nixpkgs

<TO BE ADDED>

### Linux Distro - The Gentoo Perspective

[Gentoo Linux](https://www.gentoo.org) is a source-first distribution with support for extensive package customization.
The primary means of this customization are so-called [USE flags](https://wiki.gentoo.org/wiki/Handbook:AMD64/Working/USE):
boolean flags exposed by individual packages and permitting fine-tuning the enabled features, optional dependencies and
some build parameters. For example, a flag called `jpegxl` controls the support for JPEG XL image format,
`cpu_flags_x86_avx2` controls building SIMD code utilizing AVX2 extension set, while `llvm_slot_21` indicates that the
package will be built against LLVM 21.

Gentoo permits using [binary packages](https://wiki.gentoo.org/wiki/Handbook:AMD64/Working/Features#Binary_package_support)
both as a primary installation method and a local cache for packages previously built from source. Among the metadata,
binary packages store the configured USE flags and some other build parameters. Multiple binary packages can be created
from a single source package version, in which case the successive packages are distinguished by monotonically
increasing build numbers. The dependency resolver uses a combined package cache file to determine whether any of the
available binary packages can fulfill the request, and falls back to building from source if none can.

The interesting technical details about USE flags are:

1. Flags are defined for each package separately (with the exception of a few special flags). Their meanings can be
either described globally or per package. The default values can be specified at package or profile (a system
configuration such as "amd64 multilib desktop") level.  
2. Global flags can be grouped for improved UX. Examples of groups are `CPU_FLAGS_X86` that control SIMD code for x86
processors, and `LLVM_SLOT` that select the LLVM version to build against.  
3. With the exception of a few special flags, there is no automation to select the right flags. For `CPU_FLAGS_X86`,
Gentoo provides an external tool to query the CPU and provide a suggested value, but it needs to be run manually, and
rerun when new flags are added to Gentoo. The package managers also generally suggest flag changes needed to satisfy the
dependency resolution.  
4. Dependencies, package sources and build rules can be conditional to use flags:  
   - `flag? ( … )` is used only when the flag is enabled  
   - `!flag? ( … )` is used only when the flag is disabled  
5. Particular states of USE flags can be expressed on dependencies, using a syntax similar to Python extras:
`dep[flag1,flag2…]`.  
   - `flag` indicates that the flag must be enabled on the dependency  
   - `!flag` indicates that the flag must be disabled on the dependency  
   - `flag?` indicates that it must be enabled if it is enabled on this package  
   - `flag=` indicates that it must have the same state as on this package  
   - `!flag=` indicates that it must have the opposite state than on this package  
   - `!flag?` indicates that it must be disabled if it is disabled on this package  
6. Constraints can be placed upon state of USE flags within a package:  
   - `flag` specifies that the flag must be enabled  
   - `!flag` specifies that the flag must be disabled  
   - `flag? ( … )` and `!flag? ( … )` conditions can be used like in dependencies  
   - `|| ( flag1 flag2 … )` indicates that at least one of the specified flags must be enabled  
   - `^^ ( flag1 flag2 … )` indicates that exactly one of the specified flags must be enabled  
   - `?? ( flag1 flag2 … )` indicates that at most one of the specified flags must be enabled

This syntax has been generally seen as sufficient for Gentoo. However, its simplicity largely stems from the fact that
USE flags have boolean values. This also has the downside that multiple flags need to be used to express enumerations.

## Linux Distro - Debian / Ubuntu Perspective

TO BE ADDED: [https://wiki.debian.org/CategoryMultiarch](https://wiki.debian.org/CategoryMultiarch)

## Specification

This PEP proposes a set of backward-compatible extensions to the wheel format (PEP [427](https://peps.python.org/pep-0427/)
& [491](https://peps.python.org/pep-0491/)) and the packaging ecosystem version while maintaining complete backward
compatibility with existing package managers and tools. The design was made with the intent to protect
non-variant-aware tools from failure when a new type of wheel appears that they don’t know how to manage.

### Overview

Wheel variants introduce a more fine-grained specification of built wheel characteristics beyond what wheel tags
provide. When evaluating wheels to install, the installer must determine whether variant properties are compatible with
the system in addition to determining the tag compatibility. In order to choose the most suitable wheel to install, the
installer must order wheels according to the priorities of their variant properties first, and their tags second.

Usually, providers are implemented as third-party Python packages providing the API specified in this document, called
provider plugins. These plugins provide routines for validating variant properties while building variant wheels, and
for determining wheel compatibility with the given system.

When it is necessary to query the platform in order to determine wheel compatibility, provider plugins need to be called
while installing the wheel. Otherwise, their use can be limited to build time or disabled entirely, in which case the
list of supported variant properties is encoded into the variant metadata.

Package managers must not install or run untrusted variant providers without the user explicitly opting to that.
Provider packages must not specify any dependencies, and the installer must ensure that no dependencies are installed if
specified in the provider package metadata.

It is recommended that the most commonly used plugins are either vendored, reimplemented, and/or locked to specific
wheels after verifying their trustworthiness, to enable the ability to securely install variant wheels out-of-the-box.
To reduce the maintenance costs, repositories of such vetted plugins could be maintained collaboratively and shared
between different package managers.

For plugins not in such a pre-approved list, a trust-on-first-use mechanism for every version is recommended. In
interactive sessions, the package manager can explicitly ask the user for approval. In non-interactive sessions, the
approval can be given using command-line interface options. It is important that the user is informed of the risk before
giving such an approval.

### Overview of Changes

The Wheel Variant PEP introduce three key components:

1. **Extended Wheel Filenames**: Variant wheels include a variant label in their filename to ensure:  
   1. that every distinct variant has an unique filename  
   2. that variant wheels are not  accidentally installed by non-variant-aware tools.  

2. **Variant Metadata Format**: Standardized metadata describing variant properties and provider requirements.  
   1. Metadata specification at "project level" inside `pyproject.toml`  
   2. Metadata specification of "built packages" inside two JSON files:  
      1. `**.dist-info/variant.json*`: Individual wheel variant metadata.  
      2. `*-variants.json`: Variant metadata file aggregated on the package index.  

3. **Provider Plugin System**: Plugin interface to allow detection of system capabilities and validate variant
compatibility.

4. **Environment Markers: New environment markers to declare dependencies that are applicable to a subset of variants only.**

### Extended Wheel Filename Format

One of the core requirements of the design is to ensure that installers predating this PEP will ignore wheel variant
files. We propose to achieve this intent by appending a `-{variant label}` just before the `.whl` file extension.

The variant label is separated using the same "-" character as other wheel filename components to be rejected by
filename verification algorithms currently used by installers. Wheel filenames have two optional components now: the
`build tag` (at the third position - see below), and the `variant label` (at the last position - see below).

**The variant label serves two objectives:**

- It guarantees a unique filename for different variants sharing identical tags.  
- It provides a human-readable identifier that helps to visually distinguish different variants.

The label length is strictly limited (16 characters max) to prevent the wheel filenames from becoming much longer than
they are now, and causing issues on systems with restrictions on total path length.

#### Variant label validation

- Must adhere to the following rules:  
    - Lower case only (to prevent case sensitivity issues)  
    - Between 1-16 characters  
    - Using only `0-9`, `a-z` or `.` or `_` characters

- Equivalent regex: `r"[0-9a-z._]{1,16}"`

#### Build Tag and Variant Label

- If both are present, the wheel will be rejected by installers and package indexes since the filename has too many
components.

- If only the variant label is present, it will be rejected by installers and package indexes since the python tag is
misinterpreted as the build number, and the build number must start with a digit. This assumes that no Python tags
starting with a digit will be introduced in the foreseeable future.

This critical behavior to ensure backward compatibility was confirmed by a survey of wheel filename verification methods
used by different package managers and packaging tooling ([auditwheel](https://github.com/pypa/auditwheel/blob/6839107e9b918e035ab2df4927a25a5f81f1b8b6/src/auditwheel/repair.py#L61-L64),
[packaging](https://github.com/pypa/packaging/blob/78c2a5e4f5c04fd782a5729d93892c3a3eafe365/src/packaging/utils.py#L94-L134),
[pdm](https://github.com/pdm-project/pdm/blob/main/src/pdm/models/requirements.py#L260-L287),
[pip](https://github.com/pypa/pip/blob/c46141c29c3646a3328bc4e51d354cc732fb1432/src/pip/_internal/models/wheel.py#L38-L46),
[poetry](https://github.com/python-poetry/poetry/blob/1c04c65149776ae4993fa508bef53373f45c66eb/src/poetry/utils/wheel.py#L23-L27),
[uv](https://github.com/astral-sh/uv/blob/f6a9b55eb73be4f1fb9831362a192cdd8312ab96/crates/uv-distribution-filename/src/wheel.rs#L182-L299),
[warehouse](https://github.com/pypi/warehouse/blob/main/warehouse/utils/wheel.py#L78-L81)).

Currently the wheel filename follows the following format - as defined by [PEP 427](https://peps.python.org/pep-0427/#file-name-convention)

```re
{distribution}-{version}(-{build tag})?-{python tag}-{abi tag}-{platform tag}.whl
```

The Wheel Variant PEP extends this filename format following this template:

```re
{distribution}-{version}(-{build tag})?-{python tag}-{abi tag}-{platform tag}(-{variant label})?.whl
```

**A few examples:**

- **Regular wheel (non variant):** `numpy-2.3.2-cp313-cp313t-musllinux_1_2_x86_64.whl`  
- **Variant Label:**               `numpy-2.3.2-cp313-cp313t-musllinux_1_2_x86_64-x86_64_v3.whl`  
- **Null variant (see below)**     `numpy-2.3.2-cp313-cp313t-musllinux_1_2_x86_64-null.whl`

#### One-to-One relationship

There must be a direct one-to-one relationship guarantee between variant properties and the variant label.

**A variant label must uniquely describe a specific set of variant properties for a given distribution and version.**

In other words, for a given distribution (i.e. package name) and version:

- Two different labels must not refer to the same set of variant properties.  
- The set of variant properties must always point to the same variant label.

### Null Variant

The concept of a null variant was added to make it possible to distinguish a fallback wheel variant from a regular wheel
published for backwards compatibility. For example, a package that features optional GPU support could publish the
following wheels:

- One or more wheel variants built for specific hardware: will be installed on wheel-variant enabled systems with
suitable hardware.

- A CPU-only null variant that is installed on systems with wheel variant support but without suitable hardware.

- A GPU+CPU regular wheel that is installed on systems without wheel variant support (i.e. the “mega-wheel” approach)

The null variant must not have any properties and it must use the variant label `null`.  
Conversely, wheel variants that declare any variant properties must not use the variant label `null`.

In particular, this makes it possible to publish a smaller null variant for systems that do not feature suitable
hardware, with a fallback regular wheel with support for CPU and all GPUs for systems where variants are not supported
and therefore GPU support cannot be determined.

Indeed, not being compatible with any of the available variants gives the installer more information about the system
(e.g. not having specialized hardware) than systems which do not support wheel variants. Consequently, it makes sense
that package maintainers may wish to propose a different “fallback” to their users whether their system is Wheel Variant
enabled or not. Publishing a null variant should be entirely optional. If one is published, a wheel variant enabled
installer must select in priority the null variant. If none is published, fallback on the non-variant wheel instead.
The non-variant wheel is also used if variant support is explicitly disabled by an installer flag.

### Opt-in vs Opt-out

Wheel variants should be supported using an opt-out approach. This ensures that wheel variants work seamlessly out of
the box, providing users with optimal performance and compatibility with minimal maintenance burden. Plugins will be
automatically installed into an isolated environment, enabling variant support without user intervention.

For scenarios requiring explicit control, providers can be marked as optional. In these cases, users must explicitly
enable the providers, but they will still benefit from automatic provider installation.

### Variant Properties System

Variant properties follow a key-value design, where namespace and feature name constitute the key. Namespaces are used
to group features defined by a single provider, and avoid conflicts should multiple providers define a feature with the
same name. These keys are restricted to lowercase letters, digits and underscores, to make it easier to preserve
consistency between different providers. In particular, uppercase characters are disallowed to avoid different spellings
of the same name. The character set for values is more relaxed, to permit values resembling versions.

Variant features can be declared as allowing multiple values. If that is the case, these values are matched as a logical
disjunction, i.e. only a single value needs to be supported. Features are treated conjunctively, i.e. all of them need
to be supported. This provides some flexibility in designating variant compatibility while avoiding having to implement
a complete boolean logic.

**This hierarchical structure enables:**

- Organized property management without naming conflicts  
- Independent development of provider plugins  
- Extensible support for new hardware and software capabilities without requiring changes to tools or a new PEP.  
- Clear ownership and validation responsibilities

#### Variant Property format

Variant properties use a structured 3-tuple format inspired by [PEP 301 for Trove Classifiers](https://peps.python.org/pep-0301/#distutils-trove-classification)

```shell
namespace :: feature-name :: feature-value
```

A few examples could be:

```shell
nvidia :: cuda_version_lower_bound :: 12.8
x86_64 :: level :: v3
aarch64 :: version :: 8.1a
x86_64 :: avx512_bf16 :: on
```

#### Variant Property Validation

**Variant Namespace:** identifies the provider plugin and must be unique within the plugin set used by a single package
version.

- It **must** follow this exact regex: `r"[a-z0-9_]+"`

**Variant Feature Name**: Names a specific “characteristic” within the namespace.

- It **must** follow this exact regex: `r"[a-z0-9_]+"`

**Variant Feature Value**: A single value corresponding to the combination `namespace :: feature`.

- It **must** follow this exact regex: `r"[a-z0-9_.,!>~<=]+"`
- In a “multi-value” feature, a single variant wheel can specify multiple values corresponding to a single feature key.
Otherwise, only a single value can be present.

#### Variant Ordering

In order to choose the best wheel to install, the installer must order different variant wheels according to their
variant properties. This ordering must take precedence over ordering by wheel tags.

The ordering is done according to the following algorithm:

1. Namespaces, features within namespaces and values within features are ordered according to their preference.  
2. Variant properties are ordered. The sort key is a 3-tuple, consisting of indices of namespaces, features within
namespaces and values within features in the sorted lists.  
3. Variant wheels are ordered according to the indices of their properties on the sorted list. They must be ordered so
that a variant featuring a more preferred property sorts before one that does not have the same property.

The sorting algorithm ensures that the variants with features considered more important are preferred over variants with
less important properties (e.g. GPU support over CPU optimizations), and variants featuring more preferable properties
sort before these featuring a subset of them (e.g. a wheel featuring both GPU support and CPU optimizations is preferred
over one with just GPU support). The null variant naturally sorts last, since it doesn’t have any properties.

The initial ordering of features within namespaces and values within features are provided by the provider plugins, in
the form of their ordered lists of supported properties. The initial ordering of namespaces, as well as overrides to the
remaining orderings are provided by the package metadata. Installers should also permit users to override the ordering.

### Metadata - Data Format Standard

This section describes the metadata format used for variant wheels. The format is used in three locations, with slight variations:

- in the source repository, inside the `pyproject.toml` file  
- in the built wheel, as a `*.dist-info/variant.json` file  
- on the package index, as a `{package-name}-{version}-variants.json` file.

All three variants metadata files share a common JSON-compatible structure, with some of its elements shared across all
of them, and some being specific to a single variant, as described further in this section.

These variations fit into the common wheel building pipeline where a source tree is used to build one or more wheels,
and the wheels are afterwards published on an index. The `pyproject.toml` file provides the metadata needed to build the
wheels, as well as the shared metadata needed to install them. This metadata is then amended with information on the
specific variant build, and copied into the wheel. When wheels are uploaded into the index, the metadata from all of
them is read and aggregated into a single JSON file that can be used by the package installer to efficiently evaluate
the available variants without having to fetch metadata from every wheel separately.

#### The metadata tree

The metadata is a dictionary rooted at a specific point, specified for each file separately. The top-level keys of this
dictionary are strings corresponding to specific metadata blocks, and their values are further dictionaries representing
these blocks. The complete structure can be visualized using the following tree:

```textproto
(root)
|
+-- providers
|   +- <namespace>
|      +- requires      : list[str]
|      +- enable-if     : str | None
|      +- plugin-api    : str | None
|      +- optional      : bool = False
|      +- plugin-use    : Literal["install", "build", "none"] = "install"
|
+-- default-priorities
|   +- namespace        : list[str]
|   +- feature
|      +- <namespace>   : list[str]
|   +- property
|      +- <namespace>
|         +- <feature>  : list[str]
|
+-- variants
    +- <variant-label>
       +- <namespace>
          +- <feature>  : list[str]
```

For convenience, validation and reference - [a JSON Scheme file is included with the PEP](site:assets/wheel_variants/variant_schema.json)

#### `pyproject.toml`: variant project-level data table

The pyproject.toml file is the standard project configuration file as defined in
[pyproject.toml specification](https://packaging.python.org/en/latest/specifications/pyproject-toml/#pyproject-toml-spec).
The variant metadata is rooted at the top-level variant table. This format does not include the variant dictionary.

Under a `[variant]` key, it defines the providers and default priorities needed to build and consume the variants.

**Example Structure:**

```toml
[variant.default-priorities]
# prefer `x86_64` plugin over `aarch64`
namespace = ["x86_64", "aarch64", "blas_lapack"]

# prefer aarch64 version and x86_64 level features over other features
# (specific CPU extensions like "sse4.1")
feature.aarch64 = ["version"]
feature.x86_64 = ["level"]

# prefer x86-64-v3 and then older (even if CPU is newer)
property.x86_64.level = ["v3", "v2", "v1"]

[variant.providers.aarch64]
# example using different package based on Python version
requires = [
    "provider-variant-aarch64 >=0.0.1; python_version >= '3.12'",
    "legacy-provider-variant-aarch64 >=0.0.1; python_version < '3.12'",
]
# use only on aarch64/arm machines
enable-if = "platform_machine == 'aarch64' or 'arm' in platform_machine"
plugin-api = "provider_variant_aarch64.plugin:AArch64Plugin"

[variant.providers.x86_64]
requires = ["provider-variant-x86-64 >=0.0.1"]
# use only on x86_64 machines
enable-if = "platform_machine == 'x86_64' or platform_machine == 'AMD64'"
plugin-api = "provider_variant_x86_64.plugin:X8664Plugin"

[variant.providers.blas_lapack]
requires = ["blas-lapack-variant-provider"]
plugin-use = "build"
```

### Note regarding `requires = [...]`

As shown above, the `requires = [...]` is specified as a list.

```toml
[variant.providers.aarch64]
# example using different package based on Python version
requires = [
    "provider-variant-aarch64 >=0.0.1; python_version >= '3.12'",
    "legacy-provider-variant-aarch64 >=0.0.1; python_version < '3.12'",
]
```

This design is necessary to allow future-proofing of the design when a plugin would become unmaintained or deprecated.  
However this list **must** resolve to a single and unique project to be installed. Any situation where two dependency
specifiers were to be simultaneously valid must be considered invalid and rejected.

### `*.dist-info/variant.json`: the packaged variant metadata file

The `variant.json` file is placed inside variant wheels, in the `*.dist-info/` directory containing the wheel metadata.
It is serialized into JSON, with a variant metadata dictionary being the top object. In addition to the shared metadata
imported from `pyproject.toml`, it contains a `variants` object that must list exactly one variant - the variant
provided by the wheel.

The default-priorities and providers for all wheels of the same package version on the same index must be the same and
be equal to value in `{package-name}-{version}-variants.json` hosted on the index and described below.

**The variant.json file corresponding to the wheel built from the example pyproject.toml file for x86-64-v3 would look like:**

```json
{
   "default-priorities": {
      "feature": {
         "aarch64": ["version"],
         "blas_lapack": ["provider"],
         "x86_64": ["level"]
      },
      "namespace": ["x86_64", "aarch64", "blas_lapack"],
      "property": {
         "blas_lapack": {
            "provider": ["accelerate", "openblas", "mkl"]
         },
         "x86_64": {
            "level": ["v3", "v2", "v1"]
         }
      }
   },
   "providers": {
      "aarch64": {
         "enable-if": "platform_machine == 'aarch64' or 'arm' in platform_machine",
         "plugin-api": "provider_variant_aarch64.plugin:AArch64Plugin",
         "requires": [
            "provider-variant-aarch64 >=0.0.1,<1; python_version >= '3.9'",
            "legacy-provider-variant-aarch64 >=0.0.1,<1; python_version < '3.9'"
         ]
      },
      "blas_lapack": {
         "plugin-use": "build",
         "requires": ["blas-lapack-variant-provider"]
      },
      "x86_64": {
         "enable-if": "platform_machine == 'x86_64' or platform_machine == 'AMD64'",
         "plugin-api": "provider_variant_x86_64.plugin:X8664Plugin",
         "requires": ["provider-variant-x86-64 >=0.0.1,<1"]
      }
   },
   "variants": {
      "x8664v3_openblas": {
         "blas_lapack": {
            "provider": ["openblas"]
         },
         "x86_64": {
            "level": ["v3"]
         }
      }
   }
}
```

### `{name}-{version}-variants.json`: the index level variant metadata file.

For every package version that includes at least one variant wheel, there must exist a corresponding
`{name}-{version}-variants.json` file, hosted and served by the package index, where the package name and version are
normalized according to the same rules as wheel files, as found in the
[Binary Distribution Format specification](https://packaging.python.org/en/latest/specifications/binary-distribution-format/#escaping-and-unicode).
The link to this file must be present on all index pages where the variant wheels are linked, to facilitate discovery
and guarantee efficient variant resolution.

This file uses the same structure as `variant.json` described above, except that the variants object is permitted to
list multiple variants, and must list all variants available on the package index for the package version in question.

**The following behaviors must be respected and verified during the generation of the `{name}-{version}-variants.json` file:**

- Wheel Variants must declare strictly identical `default-priorities` and `providers` dictionary entries.  
- Wheel Variants with different labels must not use strictly identical sets of variant properties  
- Wheel Variants with identical labels must use strictly identical sets of variant properties

The `foo-1.2.3-variants.json` corresponding to the package with two wheel variants, one of them listed in the
previous example, would look like:

```jsonc
{
   "default-priorities": {    // Identical to above
      ...
   },
   "providers": {    // Identical to above
      ...  
   },
   "variants": {
      "x8664v3_openblas": {
         "blas_lapack": {
            "provider": ["openblas"]
         },
         "x86_64": {
            "level": ["v3"]
         }
      },
      "x8664v4_mkl": {
         "blas_lapack": {
            "provider": ["mkl"]
         },
         "x86_64": {
            "level": ["v4"]
         }
      }
  }
}
```

#### Integration with `pylock.toml`

The following section is added to the `pylock.toml` specification:

```rst
.. _pylock-packages-variants-json:

``[packages.variants-json]``
----------------------------

- **Type**: table
- **Required?**: no; requires that :ref:`pylock-packages-wheels` is used,
  mutually-exclusive with :ref:`pylock-packages-vcs`,
  :ref:`pylock-packages-directory`, and :ref:`pylock-packages-archive`.
- **Inspiration**: uv_
- The URL or path to the `variants.json` file.
- Only used if the project uses :ref:`wheel variants <wheel-variants>`.

.. _pylock-packages-variants-json-url:

``packages.variants-json.url``
''''''''''''''''''''''''''''''

See :ref:`pylock-packages-archive-url`.

.. _pylock-packages-variants-json-path:

``packages.variants-json.path``
'''''''''''''''''''''''''''''''

See :ref:`pylock-packages-archive-path`.

.. _pylock-packages-variants-json-hashes:

``packages.variants-json.hashes``
'''''''''''''''''''''''''''''''''
```

If there is a `[packages.variants-json]` section, the installer should resolve  
variants to select the best wheel file.

## Plugin API - Standardized Variant Provider Plugin Interface

### Purpose

This section describes the API used by variant provider plugins. The plugins are a central point of the variant
specification, defining the valid metadata, and providing routines necessary to install and build variants.

This document provides the API described both in text and using Python Protocols for convenience.

### High level design

Every provider plugin must operate within a single namespace. This namespace is used as a unique key for all
plugin-related operations. All the properties defined by the plugin are bound within the plugin's namespace, and the
plugin defines all the valid feature names and values within that namespace.

It is recommended that providers choose namespaces that can be clearly associated with the project they represent, and
avoid namespaces that refer to other projects or generic terms that could lead to naming conflicts in the future.

Within a single package, only one plugin can be used for a given namespace. Attempting to load a second plugin sharing
the same namespace must cause a fatal error. However, it is possible for multiple plugins using the namespace to exist,
which implies that they become mutually exclusive. For example, this could happen if a plugin becomes unmaintained and
needs to be forked into a new package.

To make it easier to discover and install plugins, they should be published in the same indexes that the packages using
them. In particular, packages published to PyPI must not rely on plugins that need to be installed from other indexes

Plugins are implemented using a Python class. Their API can be accessed using two methods:

1. An explicit object reference, as the "plugin API" metadata.

2. An installed entry point in the variant_plugins group. The name of the entry point is insignificant, and the value
provides the object reference.

Both formats use the object reference notation from the
[entry point specification](https://packaging.python.org/en/latest/specifications/entry-points/). That is, they are in
the form of:

```python
importable.module:ClassName
```

The resulting plugin is instantiated by the equivalent of:

```python
import importable.module

plugin_instance = importable.module.ClassName()
```

The explicit "plugin API" key is the primary method of using the plugin. It is part of the variant metadata, and it is
therefore used while building and installing wheels.

The entry point method is provided to increase the convenience of using variant-related tools. It is therefore normally
used with plugins that are installed to the user's main system. It can be used e.g. to detect and print all supported
variant properties, to help user configure variant preferences or provide defaults to `pyproject.toml`.

### Behavior stability and versioning

It is recommended that the plugin’s output remains stable within the plugin’s lifetime, and that packages do not pin to
specific plugin versions. This ensures that the installer can vendor or reimplement the newest version of the plugin
while ensuring that variant wheels created earlier would still be installable.

If a need arises to introduce a breaking change in the plugin's output, it is recommended to add a new API endpoint to
the plugin. The old endpoints should continue being provided, preserving the previous output.

### Helper classes

#### Variant feature config

The variant feature config class is used to define a single variant feature, along with a list of possible values.
Depending on the context, the order of values may be significant. It is defined using the following protocol:

```python
from abc import abstractmethod
from typing import Protocol
from typing import runtime_checkable


@runtime_checkable
class VariantFeatureConfigType(Protocol):
    """A protocol for VariantFeature configs"""

    @property
    @abstractmethod
    def name(self) -> str:
        """Feature name"""
        raise NotImplementedError

    @property
    def multi_value(self) -> bool:
        """Does this property allow multiple values per variant?"""
        raise NotImplementedError

    @property
    @abstractmethod
    def values(self) -> list[str]:
        """Ordered list of values, most preferred first"""
        raise NotImplementedError
```

A "variant feature config" must provide two properties or attributes:

- `name` specifying the feature name, as a string.

- `multi_value` specifying whether the feature is allowed to have multiple corresponding values within a single variant
wheel. If it is `False`, then it is an error to specify multiple values for the feature.

- `values` specifying feature values, as a list of strings. In contexts where the order is significant, the values must
be orderred from the most preferred to the least preferred.

All features are interpreted as being within the plugin's namespace.

**Example implementation:**

```python
from dataclasses import dataclass


@dataclass
class VariantFeatureConfig:
    name: str
    values: list[str]
    multi_value: bool
```

### Plugin class

#### Protocol

The plugin class must implement the following protocol:

```python
from abc import abstractmethod
from typing import Protocol
from typing import runtime_checkable


@runtime_checkable
class PluginType(Protocol):
    """A protocol for plugin classes"""

    @property
    @abstractmethod
    def namespace(self) -> str:
        """Get provider namespace"""
        raise NotImplementedError

    @property
    def is_build_plugin(self) -> bool:
        """Is this plugin valid for `plugin-use = "build"`?"""
        return False

    @abstractmethod
    def get_all_configs(self) -> list[VariantFeatureConfigType]:
        """Get all valid configs for the plugin"""
        raise NotImplementedError

    @abstractmethod
    def get_supported_configs(self) -> list[VariantFeatureConfigType]:
        """Get supported configs for the current system"""
        raise NotImplementedError
```

### Properties

The plugin class must define the following properties or attributes:

- `namespace: str` specifying the plugin's namespace.

- `is_build_plugin: bool` indicating whether the plugin is valid for `plugin-use = "build"`. If that is the case,
`get_supported_configs()` must always return the same value as `get_all_configs()` (modulo ordering), which must be a
fixed list independent of the platform on which the plugin is running. Defaults to `False` if unspecified.

**Example implementation:**

```python
class MyPlugin:
    namespace = "example"
```

#### `def get_supported_configs(...):`

- Purpose: get features and their values supported on this system

- Required: yes

**Prototype:**

```python
    @abstractmethod
    def get_supported_configs(self) -> list[VariantFeatureConfigType]:
        ...
```

This method is used to determine which features are supported on this system. It must return a list of "variant feature
configs", where every config defines a single feature along with all the supported values. The values should be ordered
from the most preferred value to the least preferred.

The method must return a fixed list of supported features.

**Example implementation:**

```python
class MyPlugin:
    namespace = "example"

    # defines features compatible with the system as:
    # example :: version :: v2 (more preferred)
    # example :: version :: v1 (less preferred)
    # (a wheel with no "example :: version" is the least preferred)
    #
    # the system does not support "example :: something_else" at all
    def get_supported_configs(self) -> list[VariantFeatureConfig]:
        return [
            VariantFeatureConfig(
               name="version", 
               values=["v2", "v1"], 
               multi_value=False
            ),
        ]
```

#### `def get_all_configs(...):`

- Purpose: get all valid features and their values

- Required: yes

**Prototype:**

```python
    @abstractmethod
    def get_all_configs(self) -> list[VariantFeatureConfigType]:
        ...
```

This method is used to validate available features and their values for the given plugin version. It must return a list
of "variant feature configs", where every config defines a single feature along with all its valid values. The list must
be fixed for a given plugin version, it is primarily used to verify properties prior to building a variant wheel.

Note that the properties returned by `get_supported_configs()` must be a subset of those returned by this function.

**Example implementation:**

```python
class MyPlugin:
    namespace = "example"

    # all valid properties as:
    # example :: accelerated :: yes
    # example :: version :: v4
    # example :: version :: v3
    # example :: version :: v2
    # example :: version :: v1
    def get_all_configs(self) -> list[VariantFeatureConfig]:
        return [
            VariantFeatureConfig(
               name="accelerated", 
               values=["yes"],
               multi_value=False
            ),
            VariantFeatureConfig(
               name="version", 
               values=["v1", "v2", "v3", "v4"],
               multi_value=False
            ),
        ]
```

#### Python version compatible

It is recommended for plugins to avoid using any Python syntax or API not supported by any Python which has not yet
reached [end-of-life support](https://devguide.python.org/versions/). It is best to maximize compatibility by avoiding
new syntaxes whenever possible.

#### Future extensions

The future versions of this specification, as well as third-party extensions may introduce additional properties and
methods on the plugin instances. The implementations should ignore additional attributes.

For best compatibility, it is recommended that all private attributes are prefixed with an underscore (_) character to
avoid incidental conflicts with future extensions.

Variant environment markers

Three new environment markers are introduced in dependency specifications:

1. `variant_namespaces` corresponding to the set of namespaces of all the variant properties that the wheel variant was
built for.  
2. `variant_features` corresponding to the set of `namespace :: feature` pairs of all the variant properties that the
wheel variant was built for.  
3. `variant_properties` corresponding to the set of `namespace :: feature :: value` tuples of all the variant properties
that the wheel variant was built for.  
4. `variant_label` corresponding to the exact variant label that the wheel was built with.

The markers are defined as sets of strings, and therefore MUST be matched via the `in` or `not in` operator, e.g.:

```textproto
dep1; "foo" in variant_namespaces
dep2; "foo :: bar" in variant_features
dep3; "foo :: bar :: baz" in variant_properties
dep4; variant_label == "foobar"
```

Implementations MUST ignore differences in whitespace while matching the features and properties.

Variant marker expressions MUST be evaluated against the variant properties stored in the wheel being installed, not
against the current output of the provider plugins. If a non-variant wheel was selected or built, all variant markers
evaluate to `False`.

## Reference implementation

The [variantlib](https://github.com/wheelnext/variantlib) project contains a reference implementation of the protocol, querying plugins, variant filtering
and ordering, validation, variant metadata reading and writing, and generating the `variants.json` index.

A client for installing variant wheels is implemented in [uv](https://github.com/astral-sh/uv).
