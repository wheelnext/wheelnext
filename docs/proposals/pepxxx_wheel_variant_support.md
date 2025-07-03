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

### Wheel Variants

#### Wheel filename

In order to distinguish different variants, a variant label was added to the filename. The label is added as the very
last component in order to make it easy to distinguish different variants.

The model defaults to using a hash in order to provide unique and reproducible filenames out of the box. However, it
permits explicitly choosing a different label in order to make variants easy to recognize by humans. The label length
is strictly limited in order to prevent the wheel filenames to become much longer than they are now, and causing issues
on systems with smaller filename or path length limits.

The same `-` character is used as the separator in order to reduce the risk of existing package manager implementations
accidentally choosing a potentially unsupported wheel variant instead of a regular wheel. This was based on a survey
of wheel filename verification methods used by different package managers and libraries (packaging, poetry, pip, uv).
Both the current specification and some implementations are very permissive about different components, yet they all
reject wheels if there are more than six components, or the build tag does not start with a digit. A limitation of this
choice is that it assumes that the python tag will never start with a digit.

#### Variant properties

Variant properties follow a key-value design, where namespace and feature name constitute the key. Namespaces are used
to group features defined by a single provider, and avoid conflicts should multiple providers define a feature with
the same name. The character sets for all components are restricted in order to make it easier to preserve consistency
between different providers, in particular uppercase characters are rejected to avoid different spellings of the same
name. The character set for values is more relaxed, in order to permit values resembling versions and version
specifications.

Multiple values are permitted as a logical disjunction, while different features are treated conjunctively. This is
meant to provide some flexibility in designating variant compatibility while avoiding having to implement a complete
boolean logic. This flexibility is further extended via the concept of dynamic plugins, permitting the values to
be dynamically interpreted, e.g. as version ranges.

#### Variant hash

Variant hash is used as a stable and unique identifier for every set of variant properties. It is truncated to
8 characters in order to ensure that filenames remain short. SHA256 algorithm was chosen, because it is already widely
used in wheels, in the `RECORD` file and therefore the package managers do not have to implement an additional
algorithm.

To ensure reproducible hash values, properties are sorted before hashing. They are then serialized into a canonical
string form, and each one is terminated with a newline character to ensure their separation.

As a special case, a variant hash of `00000000` is used for the null variant, in order to make it easily distinguishable
from other variants.

#### Null variant

The concept of a null variant was added to make it possible to distinguish a fallback wheel variant from a regular wheel
published for backwards compatibility. For example, a package that features optional GPU support could publish
the following wheels:

1. One or more GPU wheel variants that is installed on systems with wheel variant support and a suitable GPU.

2. A CPU-only null variant that is installed on systems with wheel variant support but without suitable GPU.

3. A GPU+CPU regular wheel that is installed on systems without wheel variant support.

In particular, this makes it possible to publish a smaller null variant for systems that do not feature suitable GPUs,
with a fallback regular wheel with support for CPU and all GPUs for systems where variants are not supported
and therefore GPU support cannot be determined.

### Plugin API

#### General design

The plugin API was largely inspired by [PEP 517](https://peps.python.org/pep-0517/). However, it was extended to support
classes that are instantiated, in order to facilitate single initialization and clean caching of plugin state between
multiple method calls. For the convenience of plugin authors, both class-level and module-level (with global variables
and functions) API implementations are supported.

For the primary use in building packages and installing wheel variants, the plugin API endpoint is either specified
explicitly or inferred from requirements. Support for the latter was added as the need to explicitly guess the correct
`build-backend` value was noted as a significant shortcoming of PEP 517. However, for the convenience of package
developers, plugins are recommended to install entry points as well. Thanks to that, the developer can install
the relevant provider plugins to their system, and variant-related tooling will be able to automatically discover it
and obtain the correct API backend values.

The API means to be absolutely minimal. The plugin declares its namespace globally and does not include it in return
values to avoid potential problems if returned namespace mismatched. All methods are only passed properties
in the plugin namespace to reduce the risk of mistakenly processing properties from another namespace.
The `validate_property()` method operates on one property at a time to simplify the return value, since calling it
multiple times is not considered a bottleneck.

The types used in the API are defined using abstract protocols, in order not to force a specific implementation —
especially that such an implementation could end up relying on models deprecated in a future version of Python.
For example, the relevant data types can be implemented using data classes, named tuples, `argparse.Namespace`
or an entirely custom class.

#### Static and dynamic plugins

The split into static and dynamic plugins was introduced to handle diverse use cases for wheel variants. In particular,
it was pointed out that the static design cannot handle use cases where compatible values cannot be predicted up front
and restricted to a fixed list. For these cases, the API permits the plugin to intelligently process the actual property
values specified at build time, for example as version ranges.

At the same time, the support for the static approach to plugins was preserved in order to facilitate better caching
and the ability to pin variants easier for the use cases that do not need dynamic processing.

Both versions of the API use the same prototypes to avoid maintaining two divergent API documentations, and to make it
easier to convert plugin from one type to another. The only difference is in the value of `known_properties` argument
to `get_supported_configs()`, that explicitly takes `None` for static plugins to avoid accidentally depending on this
data in static plugins.


## Specification

### Wheel Variants

Wheel Variants provide the ability to parametrize built wheels beyond the scope currently permitted by wheel tags.
Every variant is described by zero or more properties that are defined and controlled by provider plugins. The plugins
provide a standardized Python API to determine which variants are supported by the system, and to order them according
to preference in installing.

#### Wheel filename

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

#### Variant properties

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


#### Variant hash

Variant hash is computed using the following algorithm, where `properties` are given as a list of property tuples:

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


#### Null variant

A null variant is a special case of a wheel variant. It has no properties, and its variant label is always `00000000`.
It is distinct from non-variants, as it still requires the package manager to explicitly support variants, and therefore
it can be used to provide distinct fallbacks for package managers predating variant support and for systems where none
of the other variants is supported.


### Provider Plugins

Provider plugins define the valid variant properties and provide the logic for detecting which properties are compatible
with the user's system. They are Python packages providing an API for installers to call into. Each plugin defines
a namespace for all its properties.


#### API endpoint

The API can be either implemented as top-level variables and functions in a Python module, or as a class. The API
endpoint is specified using the same syntax as object references in the [entry points specification](
https://packaging.python.org/en/latest/specifications/entry-points/), that is:

```
{import path}(:{object path})?
```

where import path specifies the module to import, as for the Python `import` statement, and object path specifies
the class to use. If object path is omitted, the whole module is used as the endpoint.

If the API endpoint specifies a callable, it is called to instantiate the provider object. Otherwise, it is used as-is.

An API endpoint specification is equivalent to the following Python pseudocode:

```python
import {import path}

if {object path}:
    obj = {import path}.{object path}
else:
    obj = {import path}

if callable(obj):
    obj = obj()
```

Additionally, a plugin provider can install an entry point in the `variant_plugins` group that can be used
by development tools to discover available providers. However, wheels must be installable without the presence of entry
points.

#### Plugin API

The plugin API needs to conform to the `PluginType` protocol as declared in the following snippet:

```python
from abc import abstractmethod
from typing import Protocol
from typing import runtime_checkable


# Type aliases for readability
VariantNamespace = str
VariantFeatureName = str
VariantFeatureValue = str


@runtime_checkable
class VariantFeatureConfigType(Protocol):
    """A protocol for VariantFeature configs"""

    @property
    @abstractmethod
    def name(self) -> VariantFeatureName:
        """Feature name"""
        raise NotImplementedError

    @property
    @abstractmethod
    def values(self) -> list[VariantFeatureValue]:
        """Ordered list of values, most preferred first"""
        raise NotImplementedError


@runtime_checkable
class VariantPropertyType(Protocol):
    """A protocol for variant properties"""

    @property
    @abstractmethod
    def namespace(self) -> VariantNamespace:
        """Namespace (from plugin)"""
        raise NotImplementedError

    @property
    @abstractmethod
    def feature(self) -> VariantFeatureName:
        """Feature name (within the namespace)"""
        raise NotImplementedError

    @property
    @abstractmethod
    def value(self) -> VariantFeatureValue:
        """Feature value"""
        raise NotImplementedError


@runtime_checkable
class PluginType(Protocol):
    """A protocol for plugin classes"""

    @property
    @abstractmethod
    def namespace(self) -> VariantNamespace:
        """Plugin namespace"""
        raise NotImplementedError

    @property
    @abstractmethod
    def dynamic(self) -> bool:
        """
        Is this a dynamic plugin?

        This property / attribute should return True if the configs
        returned `get_supported_configs()` depend on `known_properties`
        input.  If it is False, `known_properties` will be `None`.
        """
        raise NotImplementedError

    @abstractmethod
    def get_supported_configs(
        self, known_properties: frozenset[VariantPropertyType] | None
    ) -> list[VariantFeatureConfigType]:
        """Get supported configs for the current system"""
        raise NotImplementedError

    @abstractmethod
    def validate_property(self, variant_property: VariantPropertyType) -> bool:
        """Validate variant property, returns True if it's valid"""
        raise NotImplementedError
```

The plugin must implement the following attributes or properties:

- `namespace` stating the namespace used by the provider

- `dynamic` indicating whether the plugin is dynamic

It must also implement two methods or functions:

- `get_supported_configs()` that returns a list of feature names and values that are compatible with the current
  environment, in their order of preference (i.e. a wheel with such a property can be installed)

- `validate_property()` that checks whether the specified property name and value is valid (i.e. a wheel can be built
  with a such a property)

#### `get_supported_configs()`

The `get_supported_configs()` method is used to obtain the list of configurations that are supported by the current
environment. Its exact semantics depends on whether the provider plugin is declared as dynamic or not.

If the provider is static (`dynamic = False`), the list of supported configurations is expected to be fixed.
The `known_properties` parameter is always `None`, and the method must return all configurations supported
by the system. The package manager may cache that list and reuse it for other packages using the same plugin.

If the provider is dynamic (`dynamic = True`), `known_properties` is an unordered container of all properties found
in installable wheel variants for the package or packages in question. The provider must verify whether each
of the listed properties is supported, and return configurations that include these of them that are. Since the return
value may depend on `known_properties`, package managers cannot cache it across different packages, and instead must
call the method separately for every value of `known_properties`.

The `known_properties` option will be passed a type meeting the `VariantPropertyType` prototype, that is having three
attributes or properties: `namespace` with the property namespace, `feature` with its feature name, and `value` with
its value. Only properties using the provider's namespace must be passed.

The return value is an unordered list of "feature configuration types". These types must implement two attributes
or properties: `name` stating the feature name, and `value` being an ordered list of supported values. The values
should be ordered from the most preferred to the least preferred. When selecting variants to install, variants with
property values of higher precedence will be preferred.

#### `validate_property()`

The `validate_property()` method is used to determine whether the specified property is valid. It is passed a type
matching the `VariantPropertyType` prototype, and must return `True` if it is valid or `False` if it is not. When
a wheel variant is being built with multiple properties from a given namespace, the function will be called separately
for each of them. It will only be called with properties whose namespace matches the plugin's namespace.


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

### Alternative approaches

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

### Wheel Variants

#### Wheel filename

- Adding the variant label as a third component (before the build tag) with additional `~` characters. This approach
  was ultimately rejected, as adding it as a last component made it easier to distinguish different variants,
  and achieved the same goals.

- Using labels in addition to the variant hash. This approach was rejected because it caused unnecessary increase
  of filename length. As the specification evolved and made it unnecessary to include the hash in the filename,
  it was suggested to replace it with a human-readable label instead.

- Automatically generating human-readable labels by providers. This idea did not fit well with very limited variant
  label length.

#### Variant properties

- Originally, only a single value was permitted for a property. This assumed that variants designate a specific
  property of the wheel (e.g. a CPU version it was built for), while the provider plugins indicate which of these
  properties are supported by the system. However, during discussion the need for an opposite approach was indicated,
  where the wheel designates its compatibility (e.g. as a multiple compatible runtime versions), and the plugin verifies
  whether it matches the system.

- Interpreting the value as a version specifier, and matching the supported values (as versions) against it. It was
  rejected as not very generic and it was hard to define a single good variant precedence sorting. Instead, support
  for dynamic plugins was added, that makes it possible for plugins to implement a similar logic if they need one,
  and implement it in the way best fitted to their particular needs.

#### Variant hash

- Originally, the SHAKE-128 algorithm was used, as it permitted choosing an arbitrary hash length. However, it was
  pointed out that the same result can be achieved by using a more common hash algorithm, and truncating it.

- The initial implementation lacked separation between serialized variant properties. As a result, different
  combinations of properties could have yielded the same hash value (`a :: b :: c` + `de :: f :: g` = `a :: b :: cd` +
  `e :: f :: g`).

### Plugin API

- It was proposed that plugins could be implemented as callable executables (or scripts) instead. This would provide
  natural process isolation, and make it easier to implement plugins in programming languages other than Python.
  However, this idea was ultimately rejected in favor of following the approach more consistent with PEP 517.

- Originally, provider plugins relied entirely on entry points for discovery, assuming that all plugins installed
  in the environment would be used. However, this was deemed not explicit enough, and the inconsistency with PEP 517
  was pointed out.

- The original static API used a `get_all_configs()` function that provided all valid property values for the purpose
  of validation. To facilitate dynamic plugins and to avoid unnecessary divergence in API, it was replaced by a simpler
  `validate_property()` function.

- Use of more basic Python types (such as tuples) for passing variant configurations and properties was considered.
  However, dataclass-like protocols provided much better readability at a minimal cost.

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
