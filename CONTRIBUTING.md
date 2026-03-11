# Contributing to [Package Name]

Thank you for considering contributing to [Package Name]! We appreciate all kinds of contributions, from bug reports to pull requests.

## Ground Rules

* Be respectful and professional in all communications
* Follow our [Code of Conduct](CODE_OF_CONDUCT.md)
* Give constructive feedback
* Keep discussions focused on the topic at hand

## Response Time Notice

As a small team with multiple priorities, we may not always be able to respond immediately to issues or pull requests. Please allow some time for us to review your contribution.

## Types of Contributions

### Bug Reports

* File an issue using the provided template
* Include:
  * Clear description of the problem
  * Steps to reproduce
  * Expected vs. actual behavior
  * Any error messages

### Pull Requests

* Create a new branch from main
* Follow PEP 8 style guidelines
* Add tests for new functionality
* Update documentation if necessary
* Reference any related issues

## Setup Instructions

We encourage using VS Code and the .devcontainer for development of this package.

1. Clone the repository:

    ```bash
    $ git clone git@github.com:JamSuite/jamsuite-logger.git
    $ cd jamsuite-logger
    $ code .
    ```

2. If not using VS Code .devcontainer, Make sure you have uv installed and install development requirements:

    ```bash
    pip install --upgrade uv
    uv venv --clear .venv
    uv sync --active --all-groups
    ```

<!--
3. Run tests:
   ```bash
pytest
```
-->

## Code Review Process

1. Submit your pull request
<!-- 2. Wait for automated checks to complete -->
2. Address any reviewer feedback
3. Once approved, maintainers will merge your changes

## Communication Channels

* Issues: For bug reports and feature requests
* Pull Requests: For code changes

## Additional Notes

* We welcome all skill levels and backgrounds
* If you're unsure about something, don't hesitate to ask
* We aim to keep the codebase clean and maintainable

## License

By contributing to this project, you agree that your contributions will be licensed according to its current [LICENSE](LICENSE).

---

Thanks again for your interest in contributing! ☮️