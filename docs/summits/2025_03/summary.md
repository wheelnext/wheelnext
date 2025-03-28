# 2025 WheelNext Community Summit - Summary

## **Slides and Resources:** [Link](site:summits/2025_03/slidedecks_and_resources)

## **Key Highlights**

- **Wheel Variants**
    - Critical for the scientific computing / machine and deep learning ecosystems.
    - This ecosystem represent a very significant of python users, package downloads, etc.
- **Index Group Priority** Priority-based resolution to mitigate supply chain risks.
- **Wheel Evolution** Modernize metadata and compression without breaking users.
- **Governance** Push for a Packaging Council to unify standards.
- **Tooling** `uv` and `pixi` emerging as key innovators.

## 1. Lightning Talks - Key Points

### **NVIDIA** - Andy Terrel

#### **CUDA Platform**

- Enables interoperability among tools like PyTorch and JAX

#### **Python Packaging Challenges**

- **Growth and Community**
    - Python's popularity and 6 million CUDA users
    - Increased community engagement desired
        - There is a call for increased engagement from the AI and scientific computing communities within the
        broader Python language community.
        - Participation in conferences, language summits, and CPython core sprints is encouraged.
        - More support for organization such as PSF and NumFocus.

- **Technical Challenges**
    - Difficulty to ship and distribute packages containing large binaries.
    - JIT becomes increasingly demanded - especially as binaries grow in size.
    - No install-time hardware/system detection (drivers, accelerator, networking, CPU instructions, system libraries).
    - Address security concerns (e.g. dependency confusion attacks)
    - No ability to "bypass build isolation" - difficulty to extend libraries depending on specific ABIs.
    - ZStandard compression would be a significant step forward

#### **Action Items**

- Keep being a driving force in the WheelNext Open-Source initiative.
- Keep driving forward the state of scientific computing, machine and deep learning.
- Commitment to support the Python Ecosystem at large and support the open source organizations behind it.

### **META/PyTorch** - Eli Uriegas

#### **PyTorch Foundation**:

- Aims to build a strong foundation for ML applications, encouraging community collaboration

#### **Challenges**:

- Space limitations on PyPI
- Currently need for a dedicated package index to support multiple hardware accelerators (we should do better!)
- Lack of features in packaging and installers (e.g. default extras, index priority, package variants, etc.)

#### **Future Improvements**:

- Variants proposal to improve packaging experience
- Index prioritization to manage multiple indices
- Default extras to enhance user experience

#### **Current Successes and Areas for Improvement**:

- Adoption of ManyLinux 228 standard
- Need for improved build systems (e.g. conda-like system)
- Ongoing progress in handling multiple indices, pending completion of certain PEPs

### **Google/JAX** - Skye Wanderman-Milne

#### **Key Challenges**:

- Managing extra dependencies and ensuring compatibility during upgrades
- Issues with PyPI indices and package sizes, especially for nightlies

#### **Improvement Opportunities**:

- Enhancing distribution of C++ libraries via Python wheels on PyPI
- Developing more efficient packaging solutions to reduce package sizes

#### **Ecosystem & Community Opportunities**:

- Expanding JAX's ecosystem of libraries to support additional application logic
- Increasing community engagement and fee~dback to drive improvements and adoption

### **Quansight** - Ralf Gommers

#### **Key Challenges**:

- Outdated packaging standards and lack of deterministic builds
- Managing CUDA environments and packaging for specialized libraries (e.g. BLAS, OpenMPI, etc.)
- Large binary sizes and lack of distro support

#### **Improvement Opportunities**:

- Adoption of new standards and best practices (e.g. WheelNext, [PEP 739](https://peps.python.org/pep-0739/))
- Enhanced accelerator support and binary size management
- Increased community collaboration and knowledge sharing

#### **Planned Contributions**:

- Active involvement in WheelNext development and PEP reviews
- Updates to [PyPackaging native](https://pypackaging-native.github.io/) website and completion of [PEP 725](https://peps.python.org/pep-0725/)
- Community building and outreach to improve packaging practices and standards

### **Red Hat** - Jeremy Eder

#### **Key Challenges**:

- Ensuring supply chain security and compliance (FIPS environments)
- Optimizing environments for business goals (cost, performance, stability)

#### **Improvement Opportunities**:

- Developing tools like [Fromager](https://github.com/python-wheel-build/fromager) for gap analysis and
community learning
- Centralizing capabilities based on community needs

#### **Growth Opportunities**:

- Collaborating with community (Quansight, PEPs) to address impedance matching and flexibility
- Prioritizing customer feedback and business model alignment

#### **Lessons from Kubernetes**:

- Importance of customer feedback and business model alignment
- Evolution of features to address metadata problems and optimize deployments
- Scalability and adaptability: Kubernetes' success in scaling and adapting to changing requirements can inform
strategies for other projects and communities

### **AMD** - Jeff Daily

#### **Key Challenges**:

- Lack of ROCm packages for Python, leading to a significant gap in offerings
- Dependency management and compression algorithm issues
- User experience problems with package installations and environment conflicts

#### **Improvement Opportunities**:

- Developing variant packages for seamless user experience and smart defaults
- Increasing community presence and contributions from the packaging team

#### **Growth Opportunities**:

- Enhancing documentation and support for ROCm compatibility and usage
- Addressing compatibility queries and determining appropriate architecture combinations

#### **Future Directions**:

- Expanding ROCm package offerings and improving user experience through variant packages and community engagement
- Collaborating with experts (e.g. Viya, Anoush) to improve documentation and compatibility support

### **Astral.sh** - Charlie Marsh

#### **Key Projects**:

- [Ruff](https://github.com/astral-sh/ruff): a static analysis tool chain with 45 million downloads per month
- [UV](https://github.com/astral-sh/uv): a Python package and project manager with 12.5% of PyPI download requests
- [Python Build Standalone](https://github.com/astral-sh/python-build-standalone): creating portable,
relocatable Python distributions

#### **Improvement Opportunities**:

- Developing declarative configuration and lock file consistency for simplified package installations
- Enhancing speed and user experience in Python packaging and installation
- Addressing challenges in hardware-accelerated dependencies and consistent metadata

#### **Growth Opportunities**:

- Collaborating with the CPython community to upstream changes and simplify future development
- Improving packaging and distribution processes to increase project availability and adoption
- Exploring ways to improve metadata handling and dependency variance

#### **Action Items**:

- Continue developing and testing prototypes for hardware-accelerated dependencies
- Engage with users to gather feedback and identify areas for improvement
- Discuss further collaboration opportunities with the CPython community and explore ways to improve packaging
and distribution processes

### **Anaconda** - Stanley Seibert

#### **Key Focus Areas**:

- Serving data science, machine learning, AI, and scientific computing users
- Cross-platform support (Windows, Mac, Linux) and numerical performance
- Interoperability between Conda and other Python packaging tools

#### **Challenges and Concerns**:

- Variant handling and unclear user intent
- External channels and indices, including security implications
- Specifying non-Python dependencies for build and runtime

#### **Growth Opportunities**:

- Sharing experiences and aligning efforts with Python packaging
- Exploring package signing and verification methods
- Helping the community by sharing the learning accumulated with years of experience in variant related topics
in the `conda` ecosystem.

#### **Action Items**:

- Creating self-service wheel channels/indices for experimentation
- Improve interoperability between Conda and other Python packaging tools
- Investigate declaring non-Python dependencies in wheels
- Address security implications and user communication regarding channel priorities
- Facilitate self-service wheel channels on anaconda.org for experimentation

### **GPU Mode** - Mark Saroufim

#### **GPU Mode Community**:

- 15,000 kernel developers focused on ML systems
- Weekly YouTube lectures and working groups for project collaboration
- Kernel competition platform with 6,000 participants

#### **Challenges and Concerns**:

- Interoperability issues between PyTorch and custom CPP code
- Multi-backend Python packaging challenges
- JIT compilation time concerns for LLMs
- PyTorch binary size and CUDA toolkit installation issues

#### **Growth Opportunities**:

- Encouraging adoption of Wheelnext and growing its user base
- Addressing challenges and concerns to improve user experience
- Providing resources and support for GPUmode community members

#### **Action Items**:

- Organizing talks with the GPU Mode community to prove the ground of ideas defined within wheelnext.
- Explore solutions for interoperability issues between PyTorch and custom CPP code
- Investigate ways to reduce JIT compilation time for LLMs
- Provide guidance on CUDA toolkit installation via Conda and address PyTorch binary size concerns

<!-- ------------------------------------------------------------------------------------------------------ -->

## 2. Python Packaging Governance

**Presenter:** Barry Warsaw

### Supporting PEP 772: Establish a Packaging Council

<https://peps.python.org/pep-0772/>

1. Set packaging standards and roadmaps.
2. Improve diversity of input (beyond PyPA).
3. Partner with Python Steering Council.

### Expected outcomes

- A Packaging Summit at PyCon.
- Increased community involvement in packaging decisions.

### Transition from PEP 609 to PEP 772

- **Background**:
    - [PEP 609](https://peps.python.org/pep-0609/) (2019) defined Python Packaging Authority (PyPA) infrastructure and policies.
    - Limitations of [PEP 609](https://peps.python.org/pep-0609/):
        - Lack of a structured decision-making body.
        - Limited authority of PyPA.

- **Proposed Changes**:
    - Establish a five-member Packaging Council to make binding decisions on packaging standards.
    - Improve continuity with two-year terms and a focus on packaging standards.

- **Importance of Packaging**
    - **Role of PyPI**:
        - Essential for Python's popularity.
        - Provides necessary "batteries" for Python's functionality.
    - **Benefits of Diverse Decision Making**:
        - Leads to better solutions.
        - Increases community involvement and representation.

- **Community Involvement and Representation**
    - **Initial Voting Membership**:
        - Includes current PyPA members, packaging working group members, and core developers.
        - Discussion on including voices from:
            - Nonprofit organizations.
            - Commercial entities involved in packaging.

    - **Communication and Feedback**:
        - Open PR for comments on the WheelNext repo.
        - Effective communication channels to inform the community about packaging standards and updates.

- **Action Items**
    - Follow up on Packaging Summit details at PyCon.
    - Draft governance model (similar to PEP 13).
    - Solicit feedback on initial council membership.
    - Participate on the discourse thread: <https://discuss.python.org/t/pep-772-packaging-governance-process/79724>

<!-- ------------------------------------------------------------------------------------------------------ -->

## 3. Wheel Variants

**Presenter:** Jonathan Dekhtiar

**Key Actors & Contributors:** NVIDIA, Quansight, META/PyTorch

**Details on the [Full Proposal](site:proposals/pepxxx_wheel_variant_support)**

- **Takeaways**
    - Key points:
        - Python packaging limitations.
        - Wheel Variant concept in Python packaging.
        - Importance of feedback in development.

    - Challenges:
        - Current packaging solutions are unsufficient to support the scientific computing / AI / ML / DL ecosystems.
        - Python package managers are totally hardware-unaware.
        - No ability to describe a wheel beside extremely high level "tags" (python ABI, GCC version, OS, CPU Arch Family)

    - Future considerations:
        - Future-proof design for variants.
        - Combination of arbitrary metadata.
        - Shall not Impact on non-variant users.
        - Compatibility with existing installers.

- **Variant Concept in Python Packaging**
    - Key characteristics:
        - Variants should be arbitrary and combinable.
        - Design should be future-proof.

    - Benefits:
        - Addresses limitations of current Python packaging.
        - Allows for more specific hardware and software configurations.

- **Feedback Importance**
    - Value of feedback:
        - User feedback is valuable.
        - Feedback helps identify design issues.

    - Importance in development:
        - Crucial in identifying small issues.
        - Improves design and functionality of the packaging system.

- **Compatibility with Existing Installers**
    - Requirements:
        - Ensure compatibility with current installers.
        - Avoid breaking existing systems.

    - Considerations:
        - New variant system should not break existing installers.
        - Compatibility with non-variant aware installers.

- **Technical Proposal for Variants**
    - Definition:
        - Variants are wheels with additional metadata.
        - Metadata describes hardware and software configurations.

    - Implementation:
        - Allows for arbitrary and combinable specifications.
        - Enables more specific and flexible packaging solutions.
