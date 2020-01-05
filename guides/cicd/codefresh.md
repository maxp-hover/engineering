# Codefresh Conventions @HOVER
* Store your yaml files in the repo
  * in `codefresh` or a `.codefresh` repo
  * for monorepos with multiple pipelines, store the yaml files in the corresponding directory - either in the root or in a sub directory
  * name file same as pipeline
  * use `_`_
  * use `yml` as file ending
* Don't rebuild your image in different pipelines
  * This would break the pattern of ensuring an image is fine

#### Tags
* Don't use the branch name as tag, use something uniq to the current build like the SHA

#### Naming things
* stage: a verb like `test`, `deploy`, `build`, ... - optionally followed by a noun like `run_unit_tests`, `notify_slack`
* step name: Usually a noun that indicates the purpose of this step
* title: A description of what is happening using present tense
* description: Free form

#### Build policies
* concurrency
* Once a build is created terminate previous builds from the same branch.
* Once a build is terminated, terminate all child builds initiated from it.

#### Pipelines vs steps
When does what make sense?


#### Parallization
