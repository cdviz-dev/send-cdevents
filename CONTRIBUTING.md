# Contributing to send-cdevents

Thank you for your interest in contributing to the send-cdevents GitHub Action!

## Development Setup

This project uses [mise](https://mise.jdx.dev/) for tool management and task automation.

### Prerequisites

1. Install [mise](https://mise.jdx.dev/getting-started.html)
2. Clone the repository
3. Install tools and dependencies:

```bash
mise install
```

This will automatically install:
- `dprint` - for code formatting
- `bun` - for JavaScript tooling
- `github-cli` - for GitHub operations

## Development Workflow

### Code Quality Checks

Before submitting any changes, ensure your code passes all quality checks:

```bash
# Format code
mise run format

# Run all checks (formatting, validation, linting)
mise run check
```

The `check` task runs:
- `dprint check` - validates code formatting
- `bunx @action-validator/core action.yml` - validates GitHub Action schema

### Testing Changes

Test your changes using the test workflow:

```bash
# Push changes to trigger test workflow
git push origin feature-branch

# Or run tests manually via GitHub UI:
# Actions → Test Send CDEvents Action → Run workflow
```

The test workflow validates:
- Basic CDEvent sending
- HTTP endpoint integration
- Custom headers
- Configuration files
- Environment variables
- File input
- Error handling
- Config cleanup

## Making Changes

### Code Style

- Use consistent formatting (enforced by dprint)
- Follow existing patterns in the codebase
- Use proper CDEvents format (see conformance examples)
- Use multiline YAML strings (`|`) for JSON data in examples

### Documentation

- Update README.md if you change functionality
- Use proper CDEvent examples from the conformance spec
- Include GitHub-flavored markdown callouts where helpful
- Update this CONTRIBUTING.md if you change the development process

### Commit Messages

Use conventional commit format:
```
type: description

Examples:
feat: add support for custom timeout configuration
fix: resolve config file permission issues
docs: update README with new environment variables
ci: improve test workflow performance
```

## Release Process

### Preferred: GitHub Workflow (Recommended)

1. Ensure all changes are merged to `main`
2. Go to **Actions** → **Release** → **Run workflow**
3. Enter the version (e.g., `v1.2.3`) following [semantic versioning](https://semver.org/)
4. Click **Run workflow**

The workflow will:
- Run all quality checks
- Create and push version tag
- Create GitHub release with usage examples
- Update major version tag (e.g., `v1`) for GitHub Marketplace

### Alternative: Local Release

If you have the necessary permissions and tools set up locally:

```bash
# Check release status first
mise run release-check

# Create release
mise run release -- v1.2.3
```

> **⚠️ Note**: Local releases require:
> - Write access to the repository
> - GitHub CLI authenticated (`gh auth login`)
> - Clean working directory on `main` branch

### Version Guidelines

Follow [semantic versioning](https://semver.org/):

- **Patch** (`v1.0.1`): Bug fixes, documentation updates
- **Minor** (`v1.1.0`): New features, backward-compatible changes  
- **Major** (`v2.0.0`): Breaking changes, incompatible API changes

### Release Checklist

- [ ] All tests pass
- [ ] Documentation is updated
- [ ] CHANGELOG.md is updated (if exists)
- [ ] Version follows semantic versioning
- [ ] Release notes are meaningful

## Project Structure

```
.
├── action.yml              # GitHub Action definition
├── README.md              # User documentation
├── CONTRIBUTING.md        # This file
├── .mise.toml            # Tool management and tasks
├── dprint.json           # Code formatting configuration
├── .github/
│   ├── workflows/
│   │   ├── ci.yml        # Code quality checks
│   │   ├── test.yml      # Action functionality tests
│   │   └── release.yml   # Release automation
│   ├── dependabot.yml   # Dependency updates
│   └── markdownlint.json # Markdown linting config
└── test-data/           # Sample CDEvent files
```

## Available Tasks

```bash
# Show all available tasks
mise tasks

# Key tasks:
mise run format          # Format all code
mise run check          # Run all quality checks  
mise run release-check  # Check current release status
mise run release -- v1.2.3  # Create new release
```

## Getting Help

- Check existing [Issues](https://github.com/cdviz-dev/send-cdevents/issues)
- Review the [cdviz-collector documentation](https://github.com/cdviz-dev/cdviz-collector)
- Look at [CDEvents specification](https://cdevents.dev/)
- Open a new issue for bugs or feature requests

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). By participating, you agree to uphold this code.

Thank you for contributing! 🎉