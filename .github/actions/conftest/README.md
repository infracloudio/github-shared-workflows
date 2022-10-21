## Open Policy Agent 

This workflow checks organization level OPA policies against your terraform code.

#### workflow requires few parameters to be passed to it

Inputs

| Name | Type | Description |
|:------|:------|:-------|
| varset | String | name of the variable set containing aws credentials |
| TF_TOKEN | String | terraform cloud token having permission for the workspace |
| TF_ROOT | String | path to terraform code against you want to run policy checks |
| conftest_policy_path | String | path to your OPA policies and its dependecies |
| GITHUB_TOKEN | String | token required to authenticate  write to the pr |
| image-name | String | OPA policies script full image name with tag |

To checkout other private repositories deploy keys can been used, a github token is used for authentication. policies can be checked out like mentioned below

```yaml
   - name: checkout OPA policies
        uses: actions/checkout@v3
        with:
          repository: infracloudio/policy-as-code
          token: ${{ secrets.token }}
          path: ./policy-as-code
```


Following steps will add policies results to the pr, to add multiline comment an environment variable is created and used like shown in example below

```yaml
- name: pass policy check result to env variable
      if: ${{ github.event_name == 'pull_request' }}
      run: |
        echo 'MESSAGE_ENV<<EOF' >> $GITHUB_ENV
        cat ${{ inputs.TF_ROOT }}/msg.md >> $GITHUB_ENV
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



### To call this workflow caller workflow can directly implement the below:

```yaml
on:
  pull_request:

jobs:
  conftest-job:
    runs-on: ubuntu-latest
    name: conftest policy checks
    strategy:
      matrix: 
        dir: ["MySQL_db","Oracle_db","Postgres_db"] 
    env:
      current-dir: /home/runner/work/${{ github.event.repository.name }}/${{ github.event.repository.name }}
    steps:
      - name: checkout terraform code
        uses: actions/checkout@v3
        with:
          path: ./config-code
      - name: checkout OPA policies
        uses: actions/checkout@v3
        with:
          repository: infracloudio/policy-as-code
          token: ${{ secrets.token }}
          path: ./policy-as-code
      - name: checkout shared actions
        uses: actions/checkout@v3
        with:
          repository: infracloudio/github-shared-workflows
          path: ./github-shared-workflows
      - name: conftest policy check
        uses: ./github-shared-workflows/.github/actions/conftest
        with:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
          TF_ROOT: ${{ env.current-dir }}/config-code/examples/${{ matrix.dir }}
          conftest_policy_path: ${{ env.current-dir }}/config-code/conftest-policy
          image-name: ruchabhange/opa_conftest:latest
          GITHUB_TOKEN : ${{ secrets.GITHUB_TOKEN }}
          TOKEN: ${{ secrets.TOKEN }}
```

Two secrets are required namely `TOKEN` and `GITHUB_TOKEN` to be passed to the workflow, `GITHUB_TOKEN` can be directly used without defining in the actions secret.

### Note: 

The terraform code path should be specifies as : terraform-aws-rds/examples { reponame/examples/ }

The value for matrix : dir should be the folder names inside your example directory for example {MySQL_db , Oracle_db , Postgres_db}

values.tfvars is mandatory in every exmaple terraform code

