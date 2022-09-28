# github shared workflows

This repository contains reusable github workflows and built using composite actions. Workflows have been added under actions' sub directories as `.github/actions/<shared-workflow-name>/action.yml`, each sub directory <b>must</b> contain action.yml file containing the workflow.

## composite actions structure

```yaml
name:
description:
inputs:
  <input-name>:
    required: # true or false
    default: 
    description: 

outputs:
  <output-name>:
    value: ${{ steps.<step-id>.outputs.<output-name> }}
runs:
  using: "composite"
  steps:
    - uses: # can call any action
    - run: # can run a command in shell
      shell: # shell where the command runs and is required to provide shell with run 
```

composite actions can define inputs to be used within the workflow, like `${{ inputs.<input-name> }}`, and define output like `echo "::set-output name=<output-name>::<output-value>"`

Caller repositories can checkout the shared actions and call them locally by providing the action.yml path

```yaml
name: Reusable Github Workflow

on:
  push:
    branches:
      - main
  pull_request: 

jobs:
  SharedJob:
    runs-on: ubuntu-latest
    name:
    steps:
    - uses: actions/checkout@v3 # checkout current repo
    - uses: actions/checkout@v3 # checkout shared actions repo
      with: 
        repository: infracloudio/github-shared-workflows
        path: ./<checkout-path>
    - uses: <path-to-action>
      id: # can use id of this step to get outputs in another step; like ${{ steps.id.outputs.<output-name> }}
      with:
        <input-name>: <input-value>
    
```

