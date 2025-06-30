# Contributing to the Gradle Cookbook

[![a](https://img.shields.io/badge/slack-%23docs-brightgreen?style=flat&logo=slack)](https://gradle.org/slack-invite)


The Gradle Cookbook is under active development.
Any contributions are welcome!

## Discuss

- `#docs` on the [Gradle Community Slack](https://gradle.org/slack-invite)
- [GitHub Issues](https://github.com/gradle/community/issues)

## Development Environment

Follow [this documentation](https://community.gradle.org/CONTRIBUTING/) for the community site.

## HOWTOs

### Editing and Adding Pages

A few tips:

- All the pages on this site are written in Markdown.
- If needed, we have various tools available, such as code templates and macros, and we can add more MkDocs plugins if necessary.
- The Table of Contents is currently located in [mkdocs.yml](https://github.com/gradle/cookbook/blob/main/mkdocs.yml).
When adding new pages, please update the ToC to ensure they are discoverable.

### Adding New Categories

At the moment, we add categories based on consensus.
If you plan to add a new major category, it's better to discuss it in advance.

## Previews

For any pull request with the approved GitHub Actions run,
a preview site will be deployed.
The build both will share the link in the comments to the pull request.

## Building the Cookbook PDF

Run `BUILD_PDF=1 mkdocs build` to generate the PDF file in [_site/pdf/cookbook.pdf](./../_site/pdf/cookbook.pdf).

## References

- [Main Contributor Guide](https://community.gradle.org/contributing/) - describes how to contribute to Gradle
