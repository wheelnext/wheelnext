
# Welcome to WheelNext

## What is WheelNext and Why should you participate !?

WheelNext is an open-source initiative (<https://github.com/wheelnext> & <https://wheelnext.dev/>) aiming to improve the
user experience in the Python packaging ecosystem, specifically around the scientific computing and machine/deep learning space.
The current state of Python packaging was designed and implemented at a time before the increasingly complex needs these use-cases demand were prevalent.
Through WheelNext, we aim to evolve Python packaging standards to better address the needs of this important segement, while retaining its effectiveness for the overall ecosystem.

Many of the problems and gaps are summarized on the following website: <https://pypackaging-native.github.io/>
Objective: To improve the state of packaging for Python native code, making it easier to build, distribute, and install
Python packages with complex native dependencies.

### Key Issues Identified:

- **Complexity:** Due to the wide variety of microarchitectures, execution environments, and highly specialized dependencies, building and packaging native code in Python is complex and often inconsistent across different platforms.
- **Tooling Fragmentation:** Multiple tools and standards exist, leading to confusion and inefficiency.
- **Dependency Management:** Managing dependencies, especially for native code, is challenging.
- **Installation Issues:** Users often face difficulties installing packages with native code due to environment
specific issues.
- **Lack of Standardization:** There is no single, standardized way to handle native packaging in Python.
- **No Package Index Prioritization:** The main package installer `pip` is lacking ability to prioritize a given package
over another.
- **Lack of Symlink Support:** Very important to reduce package and better support native code. Yet it is lacking in the
packaging standard.
- **No ability to build a binary for a specific microarchitecture:** We are lacking the ability to describe with specificity the platform,
microarchitecture, or other execution environment details is supported by a given binary (ARMv7, ARMv8, etc.)
- **No clear path to support non-default package indexes:** Using private / corporate package indexes is common
practice, yet the tooling ecosystem does not have a solid and reliable way to support these workflows.

### Proposed Solutions:

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

- Initial kickoff effort was led by [NVIDIA](http://nvidia.com/), now aggregating efforts from many companies.
- Many ideas, formulations and this template are inherited from [PyPackaging Native](https://github.com/pypackaging-native/pypackaging-native)
