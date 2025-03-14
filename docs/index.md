
# Welcome to WheelNext

## What is WheelNext and Why should you participate !?

WheelNext is an open-source initiative (<https://github.com/wheelnext> & <https://wheelnext.dev/>) aiming to improve the
user experience in the Python packaging ecosystem, specifically around the scientific computing and machine/deep
learning space. We also anticipate benefits in other domains that heavily rely on performance of compiled Python
extension modules - the benefit of utilizing one's hardware more optimally is not exclusive to any single domain.

The current state of Python packaging was designed and implemented at a time predating the increasingly complex needs
these use cases demand were prevalent. Through WheelNext, we aim to evolve Python packaging standards to better address
the needs of this important segment, while retaining its effectiveness for the overall ecosystem.

Many of the problems and gaps are summarized on the following website: <https://pypackaging-native.github.io/>
The WheelNext objective is to improve the state of packaging for Python native
code, making it easier to build, distribute, and install Python packages with
complex native dependencies.

### Key Issues Identified:

Due to the wide variety of GPUs, CPU microarchitectures, execution environments, and highly specialized dependencies,
building and packaging native code as Python wheels is complex and often inconsistent across different platforms.

*Authors* of Python packages that need to support:

- GPUs and other hardware accelerators
- CPU capabilities beyond the CPU family (SIMD instruction sets)
- dependencies meant to be used as a single runtime or shared library rather than vendored (e.g., OpenMP, MPI, BLAS, LAPACK)
- dependencies that rely on symlinks

are struggling with how to produce and distribute wheels.

*Users* of such Python packages get a very suboptimal user experience, because the wheels often are not (or cannot)
be hosted on PyPI as of today, and they have to use package installers that don't offer ways to select wheels that
match their local hardware or installation preferences.

WheelNext aims to solve these wheel format and usability issues to the extent possible. Concrete issues already being
worked on include:

- **Lack of Symlink Support:** Very important to reduce package and better support native code. Yet it is lacking in the
packaging standard.
- **No ability to build a binary for a specific microarchitecture:** We are lacking the ability to describe with
specificity the platform, microarchitecture, or other execution environment details is supported by a given binary
(ARMv7, ARMv8, etc.)
- **No Package Index Prioritization:** The main package installer `pip` is lacking ability to prioritize a given package
index over another.
- **No clear path to support non-default package indexes:** Using private / corporate package indexes is common
practice, yet the tooling ecosystem does not have a solid and reliable way to support these workflows. Namely,
since it's difficult to safely and precisely specify how multiple enabled indices interoperate to resolve the requested
top-level and transitive dependencies

### Approach to Proposed Solutions:

- **Unified Approach:** Developing a more unified and standardized approach to packaging native code in Python.
- **Improved Tooling:** Enhancing existing tools and creating new ones to simplify the process.
- **Better Documentation:** Providing comprehensive and clear documentation to help developers navigate the packaging landscape.
- **Engineering Focus:** Putting the work and the effort to build and propose clear solutions that fixes the
aforementioned issue in the most holistic manner.

### Community Involvement:

Encourages collaboration and input from the Python community to ensure that the new standards and tools meet the needs
of a wide range of users and use cases.

## Contributing

All contributions are very welcome and appreciated! Ways to contribute include:

- Improving existing content on the website: extending or clarifying
  descriptions, adding relevant references, diagrams, etc.
- Providing feedback on existing content
- Proposing new topics for inclusion on the website, and writing the content for them
- ... and anything else you consider useful!

The content for this website is [maintained on GitHub](https://github.com/wheelnext/wheelnext).

## Acknowledgements

- Initial kickoff effort was led by [NVIDIA](http://nvidia.com/), now aggregating efforts from many companies,
 maintainers of open source projects, and other contributors.
- Many ideas, formulations and this template are inherited from [PyPackaging Native](https://github.com/pypackaging-native/pypackaging-native)
