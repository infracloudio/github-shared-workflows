## docker build push

This workflow builds and pushes a docker image to a docker registry

#### workflow requires few parameters to be passed to it

Inputs
- DOCKER_REG_USERNAME: username to log into docker registry
- DOCKER_REG_PASSWORD: password to log into docker registry 
- dockerfile-path: path to Dockerfile, if not mentioned looks for Dockerfile in root directory 
- image-name: name of the image
- build-args: build arguments, need to be provided in CSV format as inputs currently doesnt support lists, for e.g. BUILD_ARG1=VALUE1,BUILD_ARG2=VALUE2

Outputs
- imageid: returns the built image id

To login to docker repo below action is used

```yaml
- name: Login to DockerHub
  uses: docker/login-action@v2
  with:
    username: ${{ inputs.DOCKER_REG_USERNAME }}
    password: ${{ inputs.DOCKER_REG_PASSWORD }}
```

Following steps sets the Dockerfile path and converts build arguments in required format i.e. each argument seperated by new line

```yaml
- name: set environment variable for dockerfile path
      run: |
        if [[ ${{ inputs.dockerfile-path }} == "" ]]; then
          echo "DOCKERFILE_PATH=." >> $GITHUB_ENV
        else
          echo "DOCKERFILE_PATH=${{ inputs.dockerfile-path  }}" >> $GITHUB_ENV
        fi
      shell: bash
    - name: set environment variable for build args
      run: |
        echo ${{ inputs.build-args }} | tr -s ',' '\n' > build-args.txt
        echo 'BUILD_ARGS<<EOF' >> $GITHUB_ENV
        cat build-args.txt >> $GITHUB_ENV
        echo 'EOF' >> $GITHUB_ENV
      shell: bash

```
Following step is used to build and push docker image

```yaml
- name: Build and push
  id: build-and-push
  uses: docker/build-push-action@v3
  with:
    context: ${{ env.DOCKERFILE_PATH }}
    push: true
    build-args: |
      ${{ env.BUILD_ARGS }}
    tags: ${{ inputs.DOCKER_REG_USERNAME }}/${{ inputs.image-name }}:latest # image versionining not implemented currently
```

### To call this workflow caller workflow can directly implement the below:

```yaml
on:
  push:
    branches:
      - main

jobs:
  sentinel-image-job:
    runs-on: ubuntu-latest
    name: A job to build and push sentinel cli docker image
    steps:
      - name: checkout current repo
        uses: actions/checkout@v3
      - name: checkout shared actions repo
        uses: actions/checkout@v3
        with:
          repository: infracloudio/github-shared-workflows
          path: ./github-shared-workflows
      - name: docker build push
        uses: ./github-shared-workflows/.github/actions/docker-build-push
        id: build-and-push
        with:
          DOCKER_REG_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
          DOCKER_REG_USERNAME: ${{ secrets.DOCKERHUB_PASSWORD }}
          dockerfile-path: ./docker-images/sentinel-cli-script
          image-name: sentinel
      - run: echo ${{ steps.build-and-push.outputs.imageid }}
```

