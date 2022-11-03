# Migrating from V2 to V3

Migrating from V2 to V3 is straightforward. The breaking changes that are required
for migration are documented here.

1. Change all tag references from `@v2` to `@v3`
2. Remove all usages of `token` and replace them with `platform_token` in the `with:` clause

Additionally, if your workflow was based off of our example workflows you will
need to update the `github/codeql-action/upload-sarif` action to avoid warnings
related to use of an outdated Node.js in your workflow.

We have added a number of additional features, and it is recommended you review
them on our [Advanced Configuration](./advanced-configurations.md) page to
ensure the best experience.
