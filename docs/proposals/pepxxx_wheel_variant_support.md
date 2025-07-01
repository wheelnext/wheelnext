# PEP ### - Wheel Variants

| Resource            | Link                                                                                                                                     |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `PEP Link`          | `To Be Published`                                                                                                                        |
| `DPO Discussion`    | [Implementation variants: rehashing and refocusing](https://discuss.python.org/t/implementation-variants-rehashing-and-refocusing/54884) |
| `Github Repository` | <https://github.com/wheelnext/pep_xxx_wheel_variants>                                                                                        |

## Abstract

This PEP proposes an evolution of Python Wheels standard to support hardware-specific or platform-dependent package variants.
Current mechanisms for distinguishing Python Wheels (i.e Python ABI version, OS, CPU architecture, and Build ID) are
insufficient for modern hardware diversity, particularly for environments requiring specialized dependencies such as
high performance computing, hardware accelerated software (GPU, FPGA, ASIC, etc.), etc.

This proposal introduces `Wheel Variants`, a mechanism for publishing platform-dependent wheels and selecting the most suitable
package variant for a given platform.

To enable fine-grained package selection without fragmenting the Python ecosystem, this PEP proposes:

- A `Wheel Variant` system that enables multiple wheels for the same Python package version, distinguished by
hardware-specific attributes.

- A `Provider Plugin` system that dynamically detects platform attributes and recommends the most suitable wheel.

- A hash-based identification mechanism for wheel variants, ensuring compatibility while maintaining clarity in package naming.

This approach allows seamless package resolution without requiring intrusive changes to `installers`, ensures backward
compatibility, and minimizes the burden on package maintainers.

## Motivation

Existing approaches to handling platform-specific Python packages are suboptimal. Some methods include maintaining
separate package indexes for different hardware configurations, bundling all potential dependencies into a single
"mega-wheel," or creating separate package names (`mypackage-gpu`, `mypackage-cpu`). Each of these approaches has
significant drawbacks, such as excessive binary size, dependency confusion, and inefficient dependency resolution, etc.

The need for a systematic and scalable approach to selecting optimized wheels based on platform characteristics has
become increasingly urgent as Python usage expands across diverse computing environments, from cloud computing to
embedded systems and AI accelerators.

## Rationale

### User Stories

- A user wants to install a version of NumPy that is specialized for their CPU architecture.

- A user wants to install a version of PyTorch that is specialized for their GPU architecture.

- A user wants to install a version of mpi4py that has certain features enabled (e.g. specific MPI implementations for
their hardware).

- A library maintainer wants to build their library for wasm32-wasi with and without pthreads support.

- A library maintainer wants to build their library for an Emscripten platform for Pyodide with extensions for graphics
compiled in.

- A library maintainer wants to provide packages of their game library using different graphics backends.

- SciPy wants to provide packages built against different BLAS libraries, like OpenBLAS and Accelerate on macOS. This is
something they [indirectly do today](https://github.com/wheelnext/wheelnext/pull/2#discussion_r1957200935)

- Manylinux standard doesn’t cover all use-cases: [github.com/pypa/manylinux/issues/1725](https://github.com/pypa/manylinux/issues/1725)

## Specification

### Wheel Variants

Wheel Variants provide the ability to parametrize built wheels beyond the scope currently permitted by wheel tags.
Every variant is described by zero or more properties that are defined and controlled by provider plugins. The plugins
provide a standardized Python API to determine which variants are supported by the system, and to order them according
to preference in installing.

### Wheel filename

This specification extends the wheel filename to include an optional variant label. The complete filename follows
the following pattern:

```
{distribution}-{version}(-{build tag})?-{python tag}-{abi tag}-{platform tag}(-{variant label})?.whl
```

Files not featuring the `variant label` part are regular wheels. Variant wheels use it to uniquely identify each
variant. It can either be a 8-character variant hash, or a custom string of 1 to 8 ASCII characters from the range
`[a-z0-9._]`.

For example, the following are valid wheel variant names:

```
mypackage-0.0.1-py3-none-any-fa7c1393.whl
mypackage-0.0.1-cp310-abi3-manylinux_2_28_x86_64-fast.whl
mypackage-0.0.1-3-py3-none-any-fa7c1393.whl
```

### Variant properties

Each variant is described using zero or more properties. A property is a string of the following form:

```
namespace :: feature :: value
```

The `namespace` is defined by the provider plugin, and all properties defined by the provider use the same namespace.
The `feature` specifies the property name, and `value` the corresponding property value. Both `namespace` and `feature`
are ASCII strings of characters in the range `[a-z0-9_]`, while `value` of characters in the range `[a-z0-9_.,!>~<=]`.

A single variant can include multiple features from a namespace, and multiple values for the feature.
For a feature to be considered compatible with the sytem, the provider must indicate that *at least one* of its values
is compatible. For a wheel to be considered compatible, *all* of its features must be compatible.

For example, the following set of features:

```
myprovider :: version :: 1.1
myprovider :: version :: 1.2
myprovider :: accelerated :: yes
```

indicates that `myprovider :: version :: 1.1` *or* `1.2` must be supported, *and* that `myprovider :: accelerated :: yes`
must be supported.


### Variant hash

Variant hash is computed using the following algorithm, where `properties` is given as a list of property tuples:

```python
import hashlib
import typing


def variant_hash(properties: typing.Iterable[tuple[str, str, str]]) -> str:
    if not properties:
        return "00000000"
    hash_obj = hashlib.new("sha256")
    for namespace, feature, value in sorted(properties):
        hash_obj.update(f"{namespace} :: {feature} :: {value}\n".encode())
    return hash_obj.hexdigest()[:8]
```


### Null variant

A null variant is a special case of a wheel variant. It has no properties, and its variant label is always `00000000`.
It is distinct from non-variants, as it still requires the package manager to explicitly support variants, and therefore
it can be used to provide distinct fallbacks for package managers predating variant support and for systems where none
of the other variants is supported.


### Provider Plugins

A `Provider Plugin` detects the characteristics of the host platform and provides relevant metadata to `variantlib`.
Each plugin must implement a standard entry point:

```toml
[project.entry-points."variantlib.plugins"]
my_plugin = "my_plugin.plugin:MyVariantPlugin"
```

The plugin itself follows a minimal interface:

```python
from variantlib.config import ProviderConfig
from my_plugin import __version__

class MyVariantPlugin:
    __provider_name__ = "my_plugin"
    __version__ = __version__

    def run(self) -> ProviderConfig | None:
        """
        Detects platform attributes and returns a ProviderConfig if applicable.
        """
```

### Integration with `installers`

Upon package installation, `pip/uv/etc.` will:

1. Query `variantlib` for installed `Provider Plugins`.

2. Generate a list of possible variant hashes based on detected attributes.

3. Search package repositories for matching wheels.

4. Select the most relevant variant based on predefined priority rules.

5. Install the selected wheel, or fall back to a generic version if no variant is found.

6. Users must have control over which plugins are active, with the ability to disable or prioritize them via `pip.conf`
or `variant.conf`.

7. A `[uv] pip install --no-variant package` option should be available to force installation of generic wheels.

8. A `[uv] pip install --variant=abcd1234 package` option should be available to force installation of generic wheels.

9. Platform detection can be expensive, it must be cache-able. The following caching policy might be a good start
(ultimately up to the tool):
    - Run once and cache
    - Void at restart
    - Void at plugin update
    - Void manually: `[uv] pip cache --void variant_cache`

## Backward Compatibility

The introduction of `Wheel Variants` does not break existing `installer` (`pip`, `uv`, etc.) versions, as older versions
will simply ignore variant wheels. This is ensured by modifying the standard wheel filename regex ensures that legacy
`pip` versions do not mistakenly install incompatible variants.

## Security Implications

The `Provider Plugin` mechanism introduces potential security risks, such as untrusted plugins providing misleading
platform data. To mitigate this:

- Plugins should be sourced from trusted repositories. It's essentially remote code execution at install time.

## How to Teach This

User documentation will be updated to:

- Explain the concept of `Wheel Variants` and how they improve package selection.
- Provide guidance on writing and publishing `Provider Plugins`.
- Detail configuration options, such as overriding variant selection or disabling the feature entirely.
- Include debugging instructions for users encountering unexpected behavior.

## Reference Implementation

A prototype implementation has been developed, demonstrating:

- [`variantlib`](https://github.com/wheelnext/variantlib), the library handling plugin registration and variant selection.

- Demo [`Provider Plugins`](https://github.com/wheelnext/pep_xxx_wheel_variants) capable of detecting [`fictional  hardware`](https://github.com/wheelnext/provider_fictional_hw) and [`fictional technology`](https://github.com/wheelnext/provider_fictional_tech).

- A modified version of `pip` integrating variant-aware package resolution.

## Rejected Ideas

Several alternative approaches were considered and ultimately rejected:

1. **Explicit Package Naming (`mypackage-gpu`, `mypackage-cpu`)**
    - Leads to dependency resolution issues and combinatorial explosion of package variants.

2. **Bundling All Dependencies into a Single Wheel**
    - Results in unnecessarily large downloads and inefficiencies.

3. **Modifying `pip` Internals Directly**
    - Would impose significant maintenance burden on `pip` maintainers and slow adaptation to new hardware platforms.

4. **One index per configuration**
    - Significant user complexity. Force the user to carefully read the documentation.
    - Totally breaks the dependency tree: `transformers => pytorch`

![pytorch selector](../assets/images/pytorch_variant_selector.webp)

## Open Issues

1. **Dependency Management**
    - How should dependencies be expressed when a package depends on a variant of another package?

2. **Lockfile Support**
    - Should lockfiles store variant hashes or remain variant-agnostic?

3. **Platform Detection Edge Cases**
    - How should plugins handle ambiguous or incomplete hardware information?
    - Probably should stay up to the plugin maintainer to decide.

4. **How to maintain a source-of-truth**
    - Everybody who build CUDA-accelerated Wheel Variants need to use **exactly** the metadata that will
    be provided to installers.
    - Same logic for CPU, every `AVX512` specialized packages need to highlight that feature in the exact same way.
    - Shall we have a package like `troveclassifiers` to act as a "source of truth"?

## Conclusion

This proposal outlines a scalable and backward-compatible solution for platform-specific Python Wheel distribution.
By leveraging `Wheel Variants` and `Provider Plugins`, this mechanism simplifies package installation for users while
maintaining efficiency for package maintainers and repository hosts.
