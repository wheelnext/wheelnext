# PEP ### - External Wheel Hosting

| Resource            | Link                                                                                                                                     |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `PEP Link`          | `To Be Published`                                                                                                                        |
| `DPO Discussion`    | [Implementation variants: rehashing and refocusing](https://discuss.python.org/t/implementation-variants-rehashing-and-refocusing/54884) |
| `Github Repository` | <https://github.com/wheelnext/pep_wheel_variants>                                                                                        |

## Abstract

This PEP proposes an evolution of Python Wheels standard to support hardware-specific or platform-dependent package variants.
Current mechanisms for distinguishing Python Wheels (i.e Python ABI version, OS, CPU architecture, and Build ID) are
insufficient for modern hardware diversity, particularly for environments requiring specialized dependencies such as
high performance computing, accelerated computing (GPU, FPGA, ASIC, etc.), etc.

This proposal introduces `Wheel Variants`, a mechanism for publish platform-dependant wheels and selecting the most suitable
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

- A user wants to install a version of NumPy that is accelerated for their CPU architecture.

- A user wants to install a version of PyTorch that is accelerated for their GPU architecture.

- A user wants to install a version of mpi4py that has certain features enabled (e.g. specific MPI implementations for
their hardware).

- A library maintainer wants to build their library for wasm32-wasi with and without pthreads support.

- A library maintainer wants to build their library for an Emscripten platform for Pyodide with extensions for graphics
compiled in.

- A library maintainer wants to provide packages of their game library using different graphics backends.

- Manylinux standard doesn’t cover all use-cases: [github.com/pypa/manylinux/issues/1725](https://github.com/pypa/manylinux/issues/1725)

## Specification

### Wheel Variants

A `Wheel Variant` is a Python Wheel designed to support a specific platform configuration. Wheel Variants will
follow an extended filename pattern, incorporating an additional hash to indicate a specific variant:

```bash
mypackage-0.0.1~abcd1234-py3-none-any.whl
```

This `~abcd1234` hash is computed based on platform attributes detected by the user-installed `provider plugins` and
aggregated by `variantlib`, ensuring uniqueness and scalability while maintaining compatibility with existing tooling.
The `variantlib` library will generate these hashes using `hashlib.shake_128()`, providing a lightweight and
deterministic method of identifying platform-specific variants.

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

9. Platform detection can be expensive, it must be cache-able. The following caching policy might be a good start [ultimately up to the tool]
    - Run once and cache
    - Void at restart
    - Void at plugin update
    - Void manually: `[uv] pip cache --void variant_cache`

## Backward Compatibility

The introduction of `Wheel Variants` does not break existing `installer` [`pip/uv/etc.`] versions, as older versions
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

- `variantlib`, the library handling plugin registration and variant selection.

- Demo `Provider Plugins` capable of detecting `fictional  hardware` and `fictional technology`

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
    - Same logic for CPU, every `AVX512` accelerated package need to highlight that feature in the exact same way.
    - Shall we have a package like `troveclassifiers` to act as a "source of truth" ?

## Conclusion

This proposal outlines a scalable and backward-compatible solution for platform-specific Python Wheel distribution.
By leveraging `Wheel Variants` and `Provider Plugins`, this mechanism simplifies package installation for users while
maintaining efficiency for package maintainers and repository hosts.
