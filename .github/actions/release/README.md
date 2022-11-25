## Release - Composite Action

This composite action is used for Automatically creating [SemVer](https://semver.org/) compliant releases based on
PR labels and it can be consumed by the GitHub action workflow by providing the following inputs.

| Name | Type | Description | Required |
|:----|:-----|:------| :------ |
| RELEASE_BRANCH | String | Required Branch to tag | Yes |
| GITHUB_TOKEN | Secrets | Used to authenticate | Yes |

### Implementation

Following example show how to use this action in your workflow.

```yaml
name: "Release workflow"
on:
  pull_request:
    types: [closed]
    branches:
      - 'main'
    
jobs:
  terraform:
    name: 'Terraform'
    runs-on: ubuntu-latest

    steps:
      - name: Check out code
        uses: actions/checkout@v3
        
      - name: checkout composite action
        uses: actions/checkout@v3
        with:
          repository: infracloudio/github-shared-workflows
          path: ./github-shared-workflows
                    
      - name: release action
        uses: ./github-shared-workflows/.github/actions/release
        with:
          RELEASE_BRANCH: main
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```
