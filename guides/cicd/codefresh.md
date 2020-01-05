# Codefresh Conventions @HOVER

## Creating pipelines
### Store your pipeline yaml files with the application code
This allows us to track changes to our pipelines. Please obay the following rules:
* Store your pipelines in the root or in a directory called `codefresh`.
* Use `yml` as file extension

If your pipeline is triggered from multiple projects, consider storing it in one of our pipeline-specific repos. See #TODO

### Naming your pipelines
Pipeline name correspond to verbs. 

Pipelines yaml files use `.yml` as file extension
Period. This is the default that Codefresh uses and we follow this rule.

* name file same as pipeline
* use `-` as a word delimiter_
* use `yml` as file ending

* Don't rebuild your image in different pipelines

### Pipelines for Monorepos
For monorepos with multiple pipelines, store the yaml files in the corresponding application directory. That means either in the root of the monorepo or in a sub directory of the app/service/image your are creating the pipelines for.

If there is only a single pipeline, simply name the pipeline `codefresh.yml`. If there are multiple pipelines, stick to the convention of using a `codefresh` directory within the subdirectory.

A monorepo should have all its pipelines in the same Codefresh project. Suffix or prefix your pipelines accordingly. w

For example:
* A codefresh pipeline to build an image: `images/curl/codefresh.yml`
* A codefresh pipeline to build the frontend of the monorepo: `frontend/codefresh/build.yml`

### Always use a Git trigger to run your pipelines


If a git trigger is not appropriate for the pipeline you are building, you are most likely building a [generic pipeline](#generic-pipelines).

## Generic pipelines

## Building Container Images
### Tags
Tags should be uniq so that we can identidy the images
* Don't use the branch name as tag, use something uniq to the current build like the SHA
* Tag all images with the same name

### Use discerete image names


## Using custom images for steps
When you need a custom tool to perform a step in your pipeline, check Dockerhub if there is an _official_ image that ships with the required tool(s). If that is not the case, DO NOT USE a public image unless HOVER explicitly trusts the maintainer.

Usually that means you can safely use images provided by vendors we are working with like Codefresh, AWS, Google, ... You can NOT use an image provided by some "random" person on the internet. Using these images is a severe security risk as an attacker could gain access to GitHub, Codefresh, Kubernetes, and other resources.

Good:
* Using images without a namespace like `alpine:3.10` or `ruby:2.6`
* Create your own image as part of [`hoverinc/cd-tools`](https://github.com/hoverinc/cd-tools)
* Use images of 3rd parties we trust like `codefresh/cli:latest`

Bad:
* Using images that we don't have control over and we do not know the vendor like `banst/awscli` or `appropriate/curl`

## Naming things
* stage: a verb like `test`, `deploy`, `build`, ... - optionally followed by a noun like `run_unit_tests`, `notify_slack`
* step name: Usually a noun that indicates the purpose of this step
* title: A description of what is happening using present tense
* description: Free form

## Build policies
* concurrency
* Once a build is created terminate previous builds from the same branch.
* Once a build is terminated, terminate all child builds initiated from it.

## Sub-pipelines
When does what make sense?

## Plugins