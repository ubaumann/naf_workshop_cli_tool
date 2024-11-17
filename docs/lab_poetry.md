
To get hands-on experience, we'll create a new Python project. However, for the remainder of the lab, we'll be using an existing Poetry setup.

If you haven't installed Poetry yet, please follow the [installation instructions](https://python-poetry.org/docs/#installation).

## Setup a new project and add dependencies and development dependencies.

- Create a new project with Poetry. Our example use case is a small MTU tool.
- Add dependencies to the project
    - rich
    - nornir
    - nornir-napalm
    - typer
- Add development dependencies
    - pytest
    - black
- Install the created project
- Open a shell with loaded virtual environment
- Display outdated packages

??? example "Solution"

    ```bash
    poetry new mtu-tool
    poetry add rich nornir nornir-napalm typer
    poetry add --group=dev pytest black
    poetry install
    poetry shell
    poetry show --outdated

    ```


## Links:

- [Poetry basic usage](https://python-poetry.org/docs/basic-usage/)


## Additional Task

Use the test Python repository on PyPI to publish your project using Poetry.

## Next Steps

Explore the [pipx](../pipx) chapter or review the differences between `uv` and Poetry.
