# Step 3 - Test
The goal of this step is to ensure that the built images for the application work as intended. This is done by running running unit, feature, and integration tests as necessary. Only applications with a passing test suite can be deployed.

## Requirements
* You have at least unit tests for your application
* Your test suite runs locally
* You understand the dependencies to run your test suite

## Tasks
### Run your tests
This step of moving towards continuous delivery is highly dependent on the application itself. In a nutshell, you want to:
* Add [freestyle steps](https://codefresh.io/docs/docs/codefresh-yaml/steps/freestyle/#dynamic-freestyle-steps) to run arbitrary commands for unit testing, linting, etc.
* Use [compositions](https://codefresh.io/docs/docs/codefresh-yaml/steps/composition/) to run dependencies alongside your applications
* Optional: Format and export test results using [Allure](https://docs.qameta.io/allure/) and copy the report to s3
* Optional: Report a link to the results back to GitHub

Things to keep in mind
* For performance reasons, you might want to split up your tests into [multiple pipelines](https://hoverinc.atlassian.net/wiki/spaces/EN/pages/931823743/Working+with+Multiple+Pipelines)
* Check out the [Decision Path For Creating Test Automation](https://hoverinc.atlassian.net/wiki/spaces/EN/pages/931758882/Decision+Path+For+Creating+Test+Automation+For+New+Features) 
* Follow the [HOVER guidelines](https://hoverinc.atlassian.net/wiki/spaces/EN/pages/919896413/Guide+to+Writing+Integration+Tests) for running integration tests
* You might have to wait for your dependencies to become ready before you can run your tests. See the [Examples](#examples) for recommended ways to do this
* Codefresh changes the working directory in your images! Make sure to set it to whatever you need: https://codefresh.io/docs/docs/codefresh-yaml/what-is-the-codefresh-yaml/#working-directories

### Tag your image
Once the test suite passes, the image needs to be tagged in order to allow the creation of a release in [step 4](04_release). In order to apply additional tags to an image, you need to use a [push step](https://codefresh.io/docs/docs/codefresh-yaml/steps/push/).

The following tags need to be applied to the image:
* The short version of git sha (`CF_SHORT_REVISION`)
* The git sha (`CF_REVISION`)
* The branch name (`CF_BRANCH_TAG_NORMALIZED_LOWER_CASE`)

```
  push:
    type: push
    title: Tagging image
    stage: deploy
    candidate: ${{build}}
    tags:
      - "${{CF_SHORT_REVISION}}"
      - "${{CF_REVISION}}"
      - "${{CF_BRANCH_TAG_NORMALIZED_LOWER_CASE}}"
    image_name: my-app
    registry: cfcr
```
