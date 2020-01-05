# Testing application on Codefresh
The goal of this step is to ensure that the built images for the application work as intended. This is done by running running unit, feature, and integration tests as necessary. Only applications with a passing test suite can be deployed.

> This doc is work in progress. Please help us with PRs!

## Requirements
* You have at least unit tests for your application
* Your test suite runs locally
* You understand the dependencies to run your test suite
* You successfully [build your applications on Codefresh](build.md)

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

## Dealing with test reports and artifacts
