# Building deployable artifacts @HOVER
The goal of this step is to build the Docker image(s) for your application on Codefresh. These images will later be used to test and deploy the application.

Applications are deployed as container images. We build these images on a CI/CD System. The following paragraphs give you an overview of how to do this and what conventions to follow.

## Requirements
* You have a working Dockerfile and your Docker image can be built locally
* You know the build secrets you need to provide
* You have read the conventions on how to use [Codefresh](/conventions/codefresh)

## Tasks
### Create a `codefresh/build.yml`
In your `codefresh/build.yml`, you need to clone your repository:
```
version: '1.0'
steps:
  main_clone:
    title: 'Cloning main repository'
    type: git-clone
    repo: '${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}'
    revision: '${{CF_REVISION}}'
```

### Build your image
You can use the [Codefresh build step](https://codefresh.io/docs/docs/codefresh-yaml/steps/build/) to create your images.

There are some rules around building images:
* Don’t prepend a `hoverinc/`  before the image name - this happens automatically!!!
* Tag your image with `build-${CF_SHORT_REVISION}`
  * The resulting tag will contain the short version of the Git SHA (`CF_SHORT_REVISION`), for example `build-b4e5ab2`
  * The `build-` prefix here is important because we rely on naming conventions to created releases. The `build-` prefix ensures we don't accidentally create a release for an image that hat has not been tested yet.
* The image name must match your repository name for the image that will be deployed
* If you are building different images from multiple `Dockerfile`s, make sure the `Dockerfile` and resulting images are relatable. Example:
  * Repository: `cool-app`
  * Dockerfile: `Dockerfile.client`
  * Image name: `cool-app-client`
  * Tag: `build-${CF_SHORT_REVISION}`
* If your `Dockerfile` has multiple build stages that you will use during a pipeline run, prefix the resulting images for each build stage with the stage name followed by the `CF_SHORT_REVISION`
  * Repository: `cool-app`
  * Build stage: `test`
  * Image name: `cool-app`
  * Tag: `test-${CF_SHORT_REVISION}`
* The image should have a label/version file that can be used to identify the version
  * Use the `LABEL` instruction in combination with `ARG` to label the image with the version (`CF_SHORT_REVISION`): `LABEL version=${VERSION}`
  * Write a `version.txt` file to the image: `RUN echo ${VERSION} > public/version.txt`
* It is recommended that you disable the Codefresh Build optimizations via `no_cf_cache: true`.

#### Examples

##### Simple Codefresh build
`codefresh/build.yml`:
```yaml
version: '1.0'
steps:
  main_clone:
    title: 'Cloning main repository'
    type: git-clone
    repo: '${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}'
    revision: '${{CF_REVISION}}'

  build:
    title: 'Building Docker image'
    type: build
    image: ${{CF_REPO_NAME}}
    tag: build-${{CF_SHORT_REVISION}}
````

##### Access to private GitHub repos via SSH
This can be achieved securely via Buildkit:
* https://docs.docker.com/develop/develop-images/build_enhancements/
* https://codefresh.io/docs/docs/codefresh-yaml/steps/build/#buildkit-support

Example `codefresh/build.yml`:
```yaml
version: '1.0'
steps:
  main_clone:
    title: "Cloning main repository"
    type: git-clone
    repo: "${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}"
    revision: '${{CF_REVISION}}'

  setup_ssh_keys:
    title: "Setting up ssh key"
    image: alpine:latest
    stage: build
    commands:
      - echo "${GITHUB_SSH_KEY}" | tr  ',' '\n' > "${{CF_VOLUME_PATH}}/github_id_rsa"

  build:
    title: "Building Docker image"
    type: build
    image: ${{CF_REPO_NAME}}
    tag: build-${{CF_SHORT_REVISION}}
    ssh:
      - "github=${{CF_VOLUME_PATH}}/github_id_rsa"
```

Example `Dockerfile`:
```dockerfile
# syntax=docker/dockerfile:experimental
FROM node:10.15.3-alpine AS base
COPY package.json yarn.lock .
# Mount the SSH key to install additional dependencies from private repos
RUN --mount=type=ssh,id=github yarn install --no-progress --only=production
COPY . .
CMD ["node", "app.js"]
```

You can make the `GITHUB_SSH_KEY` environment variable available to your pipeline by attaching the shared configuration "github_ssh_key"

##### Access to private GitHub repos via HTTPS
This can be achieved via a developer token and [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/). The multi-stage part is important to ensure the key is not going to be part of the final image.

> While this generally works, it is recommended to use SSH and Buildkit to access private repositories.

> *TODO*: Rewrite this section to use Buildkit and a secrets file

Example `codefresh/build.yml`:
```yaml
version: '1.0'
steps:
  main_clone:
    title: "Cloning main repository"
    type: git-clone
    repo: "${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}"
    revision: '${{CF_REVISION}}'

  build:
    title: "Building Docker image"
    type: build
    image: ${{CF_REPO_NAME}}
    tag: build-${{CF_SHORT_REVISION}}
    build_arguments:
      - BUNDLE_GITHUB__COM=${{GITHUB_ACCESS_TOKEN}}
```

Example `Dockerfile`:
```dockerfile
FROM ruby:2.6.3-alpine3.9 AS base

FROM base AS gems
ARG BUNDLE_GITHUB__COM
COPY Gemfile Gemfile.lock /usr/src/app/
RUN bundle install && bundle clean

FROM base AS final
COPY --from=gems /usr/local/bundle /usr/local/bundle
COPY . .
CMD ["ruby", "app.rb"]
```

You can make the `GITHUB_ACCESS_TOKEN` environment variable available to your pipeline by attaching the shared configuration "github_access_token"

##### Create a project and repo on Codefresh
* The project name has to match your repository name
* The main pipeline that builds your image has to be called `build`
* In case you need multiple pipelines, please adhere to the following conventions:
  * Pipeline names are all lower case
  * Words are separated using underscores (`_`)
  * Place the YAML file in the `codefresh` directory
  * The name of the respective Codefresh YAML is identical to the pipeline name. For example a pipeline named `fancy_stuff` should use the following file:`codefresh/fancy_stuff.yml`
* Add a trigger that runs on commits and calls the `build` pipeline
  * Set the trigger name to the repo_name including the organization. For example `hoverinc/cool-app`
* Select that you want to use the `codefresh/build.yml` from the repo and auto select the branch

### Trigger a pipeline run
Once you have created the pipeline, you can trigger a run via
* the [codefresh-cli](https://codefresh-io.github.io/cli/installation/) on your machine:
  ```
  codefresh run repo_name/repo_name --branch=master --trigger hoverinc/cool-app --local
  ```
* a git trigger: Push a commit and watch the build on Codefresh

If the build fails, please ensure building the image locally works. If that is the case, make sure you supplied all required build arguments.

### Setup access to CFCR
You will most likely need access to the built images at some point in the process. In order to be able to pull the images onto your workstation, your need to login to the Codefresh registry:
* Start by creating a login token if you don’t have one: [Codefresh Docker Registry · Codefresh | Docs](https://codefresh.io/docs/docs/docker-registries/codefresh-registry/)
* Login:
  ```
  docker login r.cfcr.io -u CF_USER_NAME
  ```
* Pull the image:
  ```
  docker image pull r.cfcr.io/hoverinc/<repo_name>:<git_sha>
  ```

