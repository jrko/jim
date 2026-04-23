# Contributing to Jim

Thank you for considering contributing to Jim! We appreciate all kinds of contributions, from bug reports to pull requests.

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
* Follow existing patterns and conventions
* Add tests for new functionality
* Update documentation if necessary
* Reference any related issues

## Setup Instructions

1. Clone the repository:

    ```bash
    git clone git@github.com:JamSuite/jim.git
    cd jim
    ```

2. Use Jim as a Claude Code plugin:

    ```bash
    claude --plugin-dir /path/to/jim
    ```

## Contribution Workflow

Jim uses its own SDLC workflow for development (yes, Jim is self-hosting). The typical flow is:

1. **Spec** — Define the work: `/jim:spec`
2. **Plan** — Design the implementation: `/jim:plan`
3. **Build** — Implement via TDD: `/jim:build`

See [WORKFLOW.md](WORKFLOW.md) for the detailed process.

## Code Review Process

1. Submit your pull request
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
