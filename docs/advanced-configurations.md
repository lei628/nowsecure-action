# Advanced Configurations

The NowSecure Action enables a number of customizations and features. These are
documented here.

- [Action Configuration](#action-configuration) (recommended reading)
- [GitHub Summary Page](#github-summary-page)
- [Per-app Filters](#per-app-filters)
- [Configuring Finding IDs Generation](#configuring-finding-ids-generation)
- [SBOM Generation](#sbom-generation)

## Action Configuration

The NowSecure Action is further configurable via an additional file stored in the repo.
By default, it is called `.nsconfig.yml` and is placed in the root of the repo, however the name and path are configurable via the `config` and `config_path` workflow parameters.

This file allows you to configure:

- A minimum severity filter (the default includes critical, high and medium findings)
- A list of check IDs to include
- A list of check IDs to exclude
- Whether warnings should be shown

An example is provided:

```yml
minimum-severity: medium # Can be one of [critical, high, medium, low, info]
include-checks:
  - apk_hardcoded_keys
  - apk_weak_crypto_methods
exclude-checks:
  - android_janus_warn
include-warnings: true
```

## GitHub Summary Page

To view a summary of the results produced by NowSecure in the workflow, enable the top-level `summary` option in the `.nsconfig.yml`.

For example:

```yml
filter:
  example-filter:
    # ...
# Remainder of configuration...
summary: true
```

## Per-app Filters

By default, filters will be applied for all invocations of the action. However, it is possible
to have different configurations for different apps via a `configs` key.

For example, we can move the above configuration under the `example-workflow-config` key:

```yml
configs:
  example-workflow-config:
    filter:
      include-warnings: true
      minimum-severity: medium
```

And then reference it in the workflow:

```yml
- name: NowSecure
  uses: nowsecure/nowsecure-action/convert-sarif@v3
  timeout-minutes: 60
  with:
    # Specify a config in .nsconfig.yml
    config: example-workflow-config
    # ...
```

Filters can either be defined inline as above, or defined by references to a named filter defined under the `filters:` key

```yml
filter:
  example-filter: # Define a filter that can be used in multiple configs
    include-warnings: true
    minimum-severity: medium

configs:
  example-workflow-config:
    # use the filter defined above
    filter: example-filter
```

## Configuring Finding IDs Generation

The actions derive a persistent ID from the finding key, the platform and the package name. This can be controlled by the `key` element in `.nsconfig.yml`, declared either at the root or as part of a `config` element.

> Note: This is a change from the ID derivation in v2.1.1 and prior. To continue using the old key function, set the `v1-key` to the platform and package you are assessing.

For example:

```yml
key:
  platform: false                 # Don't use the platform when deriving an ID
  package: false                  # Don't use the package when deriving an ID
  v1-key: android com.example.app # Use the old function for "com.example.app" on Android
```

## SBOM Generation

To attach NowSecure SBOM data into Dependency Insights, in the `with:` section of the `convert-sarif` action, add `enable_dependencies: true`.
