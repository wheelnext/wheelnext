# WheelNext

## Contributing

All contributions are very welcome and appreciated! Ways to contribute include:

- Improving existing content on the website: extending or clarifying
  descriptions, adding relevant references, diagrams, etc.
- Providing feedback on existing content
- Proposing new topics for inclusion on the website, and writing the content for them
- ... and anything else you consider useful!

## Using the DevContainer

This repository provides a [devcontainer](https://containers.dev/).
Instructions to install this feature and start the `devcontainer` can be found for:

- VSCode: https://code.visualstudio.com/docs/devcontainers/containers
- Github CodeSpace: https://docs.github.com/en/codespaces/setting-up-your-project-for-codespaces/adding-a-dev-container-configuration/introduction-to-dev-containers
- JetBrains: https://www.jetbrains.com/help/idea/start-dev-container-inside-ide.html

The `devcontainer` automatically launches an HTTP server on port `8080` and serving the live-built documentation: http://localhost:8080/

## Building the documentation locally

To build previews of the documentation locally, start by [installing
uv](https://docs.astral.sh/uv/getting-started/installation/).  This will give you the `uvx` command,
which is an alias for `uv run`.

Now you can build the docs like this:

```sh
uvx --with-requirements requirements.txt mkdocs build
```

or build the docs locally and serve them over local HTTP:

```sh
uvx --with-requirements requirements.txt mkdocs build
```

## Code of Conduct

Everyone interacting in the WheelNext project's codebases, issue trackers,
and communication channels is expected to adhere to the
[NumFOCUS Code of Conduct](https://numfocus.org/code-of-conduct).
