# Workshop - Python Wheel Variants

## Abstract

In this workshop, we will focus on brainstorming about the best way to deliver the
Python [Wheel Variant proposal](site:proposals/pepxxx_wheel_variant_support/), which enables
hardware-specific and platform-dependent package variants to address modern hardware diversity. It proposes a system for
building, packaging, publishing and installing the most suitable package variant for a given platform without
fragmenting the Python ecosystem. It also focuses on how to minimize the maintenance impact for
build frontend (`pip`, `uv`, installer`) maintainers while attempting to enabling the most flexible ecosystem possible.

## Definition

A **Wheel Variant** is a Python Wheel specialized for a given platform at a finer level than what existing tags provide
(hardware, software, etc.).

## Rapid Summary of the Solution

- **Wheel Variants System:** Enables multiple wheels for the same Python package name & version, distinguished by
hardware-specific attributes using hash-based unique identifier.

- **Provider Plugins:** Dynamically detect platform attributes and recommend suitable wheels. Plugins can interface
with any Python installer as long as they implement the interface. They could be in any language as long the interface
is respected.

- **Backward Compatibility and Security:** Ensures compatibility with existing Python tooling by guaranteeing that Wheel
Variants will be ignored by non Wheel Variant enabled installers.

## Points of Discussion

Many aspects of the proposal need some in-depth thinking and discussions.
We would like to keep the discussion "reasonably non technical and more focused on user experience".

- **[UX: Package Installation Workflow]:**
    - Shall there be a flag to "enable" variants: `pip install --allow_variants <package>`
    - How do you request a specific `variant` to be installed: `I would like an ARMv8.1a package please`
    - How do you build `lockfiles`. Shall it contain any `variant` information ?
    - Shall you "inform the user" a Variant was installed and which one ? Only in `--verbose` ?
    - Shall the installer prioritize the "highest version" available or the "best variant" available (even if it means
    lower version - how much lower ?)
    - How shall we define "best variant match" ?

<br>

- **[UX: Package Building Workflow]:**
    - What would be a "good approach" for a `Provider Plugin` to "inform the build backend" what Variant Metadata to inject.
    - Arbitrary metadata will lead to some chaos. How do we keep a sense of "structure" without
    a centralized entity (shall it be decentralized)?
    - How do we help package maintainers select a "reasonable set of variants" to build instead of every projects doing
    something wildly different.
    - How shall we provide a "build matrix experience" for a given project (build my project for these X many variants).

<br>

- **[UX: Package Indexes & Distribution]:**
    - What metadata should they expose through APIs and through web UI, and any other packaging tools
    (uploaders like twine, auditing and vendoring tools like auditwheel, etc.).

## Expected Deliverables

By the end of the workshop, we aim to converge to some ideas, strategies and consensus to tackle the aforementioned questions.

## Useful Links

- [Wheel Variant Proposal](site:proposals/pepxxx_wheel_variant_support/)
- [Discussion on discuss.python.org](https://discuss.python.org/t/implementation-variants-rehashing-and-refocusing/54884)
- [Tutorial and Working Demo of Wheel Variants](https://github.com/wheelnext/pep_xxx_wheel_variants)
- [PyPackaging Native ~ Packaging projects with GPU code](https://pypackaging-native.github.io/key-issues/gpus/)
- [PyPackaging Native ~ Distributing a package containing SIMD code](https://pypackaging-native.github.io/key-issues/simd_support/ )

## Workshop Organizers

- **Jonathan Dekhtiar:** NVIDIA - jdekhtiar @ nvidia.com
- **Ralf Gommers:** Quansight - rgommers @ quansight.com
- **Eli Uriegas:** Meta - eliuriegas @ meta.com

<hr>

We look forward to your participation and contributions to making Wheel Variants a reality!
