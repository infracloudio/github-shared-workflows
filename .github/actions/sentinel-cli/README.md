## sentinel cli

This workflow checks organization level sentinel policies against your terraform code.

#### workflow requires few parameters to be passed to it

Inputs
- varset : name of the variable set containing aws credentials
- organization : terraform cloud org name
- workspace: terraform cloud workspace name
- TF_TOKEN : terraform cloud token having permission for the workspace
- terraform-code-path: path to terraform code against you want to run policy checks
- sentinel-policy-path: path to your sentinel policies and its dependecies
- GITHUB_TOKEN: token required to authenticate  write to the pr

To checkout other private repositories deploy keys can been used, a ssh key pair is used for authentication. Public key has been added to the private repo being checked out as deploy key and private key has been added as actions secret to the repo calling workflow. policies can be checked out like mentioned below

```yaml
- name: checkout sentinel policies
        uses: actions/checkout@v3
        with:
          repository: infracloudio/sentinel-policy-as-code
          ssh-key: ${{ secrets.PRIVATE_SSH_KEY }}
          path: 
```

Following step creates a docker container to run a shell script, which uploads the terraform code configuration to terraform cloud, creates a workspace if does't exists, adds the workspace to varset if isn't already present, creates a plan only run, downloads mock data and run sentinel cli to validate the policies.

```yaml
- name: run docker container containing script
  uses: addnab/docker-run-action@v3
  with:
    image: ruchabhange/sentinel:latest
    options: -v ${{ inputs.sentinel-policy-path }}/common-functions:/opt/sentinel/common-functions -v ${{ inputs.sentinel-policy-path }}/policies:/opt/sentinel/policies  -v ${{ inputs.terraform-code-path }}:/opt/sentinel/config-code -e TF_TOKEN=${{ inputs.TF_TOKEN }} -e organization=${{ inputs.organization }} -e workspace=${{ inputs.workspace }} -e varset=${{ inputs.varset }}
    shell: bash
    run: /opt/sentinel/script.sh

```
Following steps will add policies results to the pr, to add multiline comment an environment variable is created and used like shown in example below

```yaml
- name: pass policy check result to env variable
      if: ${{ github.event_name == 'pull_request' }}
      run: |
        echo 'MESSAGE_ENV<<EOF' >> $GITHUB_ENV
        cat ${{ inputs.sentinel-policy-path }}/policies/message.txt >> $GITHUB_ENV
        echo 'EOF' >> $GITHUB_ENV
      shell: bash
- name: write policy check result to pr
      if: ${{ github.event_name == 'pull_request' }}
      uses: mshick/add-pr-comment@v1
      with:
        message: |
          ${{ env.MESSAGE_ENV }}
        repo-token: ${{ inputs.GITHUB_TOKEN }}
```

The common functions, policies and terraform code are being mounted as volumes to the container. The container expects following environment variables:

- TF_TOKEN     : terraform cloud token having permission for the workspace
- organization : terraform cloud org name
- workspace    : terraform cloud workspace name
- varset       : name of the variable set containing aws credentials


### To call this workflow caller workflow can directly implement the below:

```yaml
on:
  push:
    branches:
      - main
  pull_request:

jobs:
  sentinel-cli-job:
    runs-on: ubuntu-latest
    name: sentinel cli policy checks
    env:
      current-dir: /home/runner/work/${{ github.event.repository.name }}/${{ github.event.repository.name }}
    steps:
      - name: checkout terraform code
        uses: actions/checkout@v3
        with:
          path: ./config-code
      - name: checkout sentinel policies
        uses: actions/checkout@v3
        with:
          repository: infracloudio/sentinel-policy-as-code
          ssh-key: ${{ secrets.PRIVATE_SSH_KEY }}
          path: ./sentinel-policy-as-code
      - name: checkout shared actions
        uses: actions/checkout@v3
        with:
          repository: infracloudio/github-shared-workflows
          path: ./github-shared-workflows
      - name: sentinel policy check
        uses: ./github-shared-workflows/.github/actions/sentinel-cli-script
        with:
          varset: 
          organization: 
          workspace: 
          TF_TOKEN: ${{ secrets.TF_TOKEN }}
          terraform-code-path: ${{ env.current-dir }}/config-code
          sentinel-policy-path: ${{ env.current-dir }}/sentinel-policy-as-code
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

Two secrets are required namely `TF_TOKEN` and `GITHUB_TOKEN` to be passed to the workflow, `GITHUB_TOKEN` can be directly used without defining in the actions secret.

