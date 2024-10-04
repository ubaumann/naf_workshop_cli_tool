
The goal of this lab is to consolidate all the concepts you've learned. You are encouraged to think outside the box and create your own use case or idea for the CLI tool.

Start by creating a new Python project using Poetry. Use Typer to build a CLI tool. Below is an example use case:

- The CLI tool should include a --no-dry-run option, which will only execute commands on the actual network when dry-run mode is disabled.
- It should support increasing verbosity by using the -v flag multiple times.
- The CLI tool should provide an option to specify the Nornir configuration file.
- It should also include GET and SET options to retrieve and update the chosen YANG model.
