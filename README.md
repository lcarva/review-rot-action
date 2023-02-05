# review-rot-action

GitHub actions for [lcarva/review-rot](https://github.com/lcarva/review-rot).

## Run Action

Use this action to execute review-rot for a given configuration.

Example:

```yaml
- name: Run review-rot
  uses: lcarva/review-rot-action/run@main
  with:
    config: config.yaml
    output: output.json
```

Example config.yaml:

```yaml
arguments:
  format: json
git_services:
- host: github.com
  repos:
  - tektoncd/chains
  token: "${GITHUB_TOKEN}"
  type: github
```

`envsubst` can be used to replace the placeholder token with the token from the workflow, e.g.:

```yaml
- name: Process config
  run: |-
    set -euo pipefail
    < ./config.yaml envsubst > config-with-token.yaml
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

NOTE: The default `GITHUB_TOKEN` may not be sufficient due to rate-limiting. Use a fine-grained
personal GitHub token if that's the case.

## Web Action

The web action is a convenient way to fetch the web app files from review-rot used to display
the results in a web friendly format.

Example:

```yaml
- name: Fetch review-rot web
  uses: lcarva/review-rot-action/web@main
  with:
    output: web
```

`output` specifies an existing directory where the HTML, JavaScript, and CSS files are copied to.

## Publish Page Example

The example below is a full example of a workflow that fetches data with review-rot, and publishes
the result as a GitHub page.

```yaml
name: Publish

on:
  push:
    branches:
      - main
  schedule:
    # At every minute past every hour on every day-of-week from Monday through Friday
    - cron: '* */1 * * 1-5'
  workflow_dispatch:

jobs:
  publish:
    runs-on: ubuntu-latest

    permissions:
      pages: write
      id-token: write

    steps:
      - name: Check out repo
        uses: actions/checkout@v3

      - name: Process config
        run: |-
          set -euo pipefail
          < ./config.yaml envsubst > config-with-token.yaml
        env:
          GITHUB_TOKEN: ${{ secrets.CUSTOM_GITHUB_TOKEN }}

      - name: Run review-rot
        uses: lcarva/review-rot-action/run@main
        with:
          config: config-with-token.yaml
          output: output.json

      - name: Process data
        run: |-
          set -euo pipefail
          echo '=== START OF FULL DATA ==='
          cat output.json
          echo '==== END OF FULL DATA ===='

          mkdir web

          # You can also filter the data here.

          cp output.json web/data.json

      - name: Fetch review-rot web
        uses: lcarva/review-rot-action/web@main
        with:
          output: web

      - name: Configure pages
        uses: actions/configure-pages@v2

      - name: Upload pages artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: web

      - name: Deploy pages
        uses: actions/deploy-pages@v1
        id: deployment
```

NOTE: The workflow above requires setting the source to "GitHub Actions" in the repository's
page settings.
