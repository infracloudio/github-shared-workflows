## Checkov - Composite Action

This action can be consumed by the GitHub action workflow by providing the following inputs.

| Name | Type | Description | Required |
|:----|:-----|:------| :------ |
| AWS_REGION | String | Name of AWS Region | Yes |
| OIDC_ROLE_ARN | String | AWS OIDC role arn. You can use existing one or create as per your need | Yes |
| GITHUB_TOKEN | Secrets | This is used to write comment on PR | Yes |

### Implementation

Following example show how to use this action in your workflow.

```yaml
name: checkov-code-scan

on: 
  pull_request:
  
jobs:
  plan:
    name: checkov-scan
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      pull-requests: write
          
    steps:
      - name: Check out code
        uses: actions/checkout@v2
        
      - name: checkout checkov compositeAction
        uses: actions/checkout@v3
        with:
          repository: infracloudio/github-shared-workflows
          path: ./github-shared-workflows
        
      - name: run checkov scan against plan
        uses: ./github-shared-workflows/.github/actions/checkov
        with:
          AWS_REGION: ${{secrets.AWS_REGION}}
          OIDC_ROLE_ARN: ${{ secrets.OIDC_ROLE_ARN }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```