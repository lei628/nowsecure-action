# Getting Started

To use this action an active NowSecure Platform account is required. If you **_are not_** an existing NowSecure Platform customer, please [contact us](https://info.nowsecure.com/github-request).

## Prerequisites

- A NowSecure Platform token
  - To generate a token in the UI, go to the "Profile & Preferences" page and click “Create Token”
- A NowSecure Group ID
  - To find your group ID in the UI, go to your app’s “Package Details” page and copy the Group ID by hovering over the group name
- (Optional for GHAS) An active GitHub account (cloud or on-prem) with an active Advanced Security feature

## Basic Configuration

### For a New Workflow

For the easiest setup, see our [example annotated workflow](../workflows/basic.yml).

### For an Existing Workflow

> Note: For line-of-code identification, `ripgrep` must be available in the runner. For Ubuntu images, add a step for `apt-get install -y ripgrep`.

After the stage that builds your application (e.g. called `build`), create a new stage called `scan`:

```yml
scan:
  runs-on: ubuntu-latest
  outputs:
    report_id: ${{ steps.upload.outputs.report_id }}
  # The stage that builds the application.
  needs: build
  steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    # Replace with whatever pulls the application file before we upload.
    - name: Download application
      uses: actions/download-artifact@v2
      with:
        # Generated in the "build" stage.
        name: app

    - id: upload
      name: NowSecure upload app
      uses: nowsecure/nowsecure-action/upload-app@v3
      with:
        platform_token: ${{ secrets.NS_TOKEN }}
        # TODO: Replace application path.
        app_file: "example.apk"
        # TODO: Replace the Group ID.
        group_id: "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
```

Then we can introduce another stage, called `process` which retrieves the results from the NowSecure Platform and converts the results to SARIF for GHAS:

```yml
process:
  if: ${{ needs.scan.outputs.report_id }}
  runs-on: ubuntu-latest
  # The above stage we introduced.
  needs: scan
  steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: NowSecure download report
      uses: nowsecure/nowsecure-action/convert-sarif@v3
      timeout-minutes: 60
      with:
        report_id: ${{ needs.scan.outputs.report_id }}
        platform_token: ${{ secrets.NS_TOKEN }}
        # TODO: Replace the Group ID.
        group_id: "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"

    - name: Upload SARIF file
      uses: github/codeql-action/upload-sarif@v1
      with:
        sarif_file: NowSecure.sarif
```
