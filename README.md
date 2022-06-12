# gha-workflows-examples

> Examples of GHA workflows, oriented towards real world use cases.

---

- [gha-workflows-examples](#gha-workflows-examples)
  - [Directories](#directories)
    - [Directory Descriptions](#directory-descriptions)
      - [actions](#actions)
      - [complete](#complete)
      - [containers](#containers)
      - [ios](#ios)
      - [node](#node)
      - [ruby](#ruby)
      - [util](#util)
  - [File Naming](#file-naming)
    - [Naming Structure](#naming-structure)
    - [Common Naming Usage](#common-naming-usage)

This repository contains a variety of Github Actions workflows that cover everything from testing to deploying. Examples for common (and uncommon) tasks across many languages and tools.
## Directories

Directories in this repo are mostly organized by general language or technology being interacted with, such as `containers` or `node`. Within these directories, you will find subdirectories organized by use case, such as `test` or `deploy`.

### Directory Descriptions

#### actions
Here you will find workflows related to Github Actions, such as tests for composite workflows.

In fact, the workflow in this category is tests for [a composite workflow I built](https://github.com/slaughtr/cloudwatch-build-metrics).

* `composite-workflow-test-on-push.yml` - test the composite workflow for failure and success

#### complete
These are complete end to end workflows, generally doing large tasks across many apps. Generally they might fit in another directory, but are complex enough to justify being separated.

* `deploy-multi-app-multi-method-on-pr-merge.yml`
  * A huge multi-app deployent with infra via CDK
* `deploy-new-statsd-host-on-pr-merge.yml`
  * This will deploy a statsd host to EC2 and give you the IP address in a comment
* `test-using-sqlite-on-push.yml`
  * Run tests using a sqlite database
* `trigger-bitrise-build-on-pr-comment.yml`
  * Trigger a build of your app in the CI/CD platform Bitrise

#### containers
Operations with containers such as Convox (third party AWS ECS CLI tool) or Github Codespaces.

* `codespace-prebuild-on-pr-merge.yml`
  * Prebuild a Codespace container with the latest changes
* `convox-deploy-ruby-run-migrations-on-pr-merge.yml`
  * Deploy your Rails app to ECS with Convox and then run migrations
* `convox-deploy-ruby-run-migrations-on-dispatch.yml`
  * Intended as a compliment to the PR merge workflow, this deploys directly to prod manually

#### ios
Workflows that do iOS things. One workflow, anyway!

* `run-xcode-tests-on-dispatch.yml`
  * With prettified ouput
 
#### node
Anything related node: apps, React SPA deployments, npm package releases, etc.

* `serverless-deploy-pr-merge.yml`
  * Deploy your Serverless Framework app
* `build-deploy-react-to-s3-on-pr-merge.yml`
* `deploy-apollo-app-on-pr-merge.yml`
  * Deploy your Apollo GraphQL app
* `deploy-multiple-apps-on-pr-open-update-merge.yml`
* `release-npm-package-flaky-tests-on-pr-merge.yml`
  * Deal with flaky tests by re-running them
* `release-npm-package-on-pr-merge.yml`
  * Release your npm package
* `run-tests-yarn-on-pr-open-update.yml`
  * This uses the traditional actions/cache method instead of the one-liner method in other workflows
* `serverless-deploy-npm-node-lambda-on-pr-merge.yml`
  * Use `sls` to deploy a Lambda function
* `serverless-deploy-yarn-node-lambda-on-pr-merge.yml`
  * And then do it using yarn instead of npm

#### ruby
Rails-oriented, these are workflows that interact with Ruby/Rails - tests, deployments, migrations.

* `bisect-tests-on-pr-comment.yml`
  * Use rspec's `--bisect` flag to identify problematic tests
* `run-tests-with-postgres-and-elasticsearch-on-pr-merge.yml`
  * Run tests using a Postgres and Elasticsearch
* `run-tests-with-mysql-selenium-on-commit.yml`
  * Or toss in Selenium and use MySQL instead

#### util
A collection of non-stack-specific workflows and examples of doing strange spooky CI/CD business. You might find workflows that interact with Github, validate PR metadata, or show how to hack around issues that you might encounter. 

* `bypass-buggy-failure.yml`
  * An example of a way to bypass a buggy failure condition, in this case the Serverless CLI timing out
* `comment-on-pr-when-specific-files-change.yml`
  * Create a PR comment when specific files in a repo change, such as anything under the `db` directory
* `create-release-notes-on-tag.yml`
  * Automatically create a release and release notes when a new tag is added
  * `release-note-configuration.json`
    * Configuration file for the release notes
* `run-task-on-cron-schedule.yml`
  * Run a task at a specified time interval
* `trigger-repository-dispatch-in-another-repo.yml`
  * Kick off a repository_dispatch trigger in a different repo
* `validate-semantic-pr-name.yml`
  * My favorite: enforce conventional commits and PR titling. A beautiful commit history is worth it!
 
## File Naming

### Naming Structure

Files in this repo are named following a loose convention: `$USE-$EVENT` where `$USE` is WHAT the workflow does and `$EVENT` is WHEN the workflow is executed. With this approach, we can quickly deduce what a workflow does and when without having to open the file. This also guides us to not create overly complex workflows that do many things. If a workflow is utilizing many events and if statements to control what happens for each event, you likely want to break it out into smaller pieces.

For an example: a workflow named `deploy-docker-on-pr-merge.yml` would have `$USE` of `deploy-docker` and `$EVENT` of `on-pr-merge`. Note that the `$EVENT` descriptor does not necessarily have to match the literal event type in the workflow, due to some non-intuitive things such as PR merges not having an event. These are outlined in the events list below.

> Caveat: the workflow files in this repo are intentionally named in overly verbose and descriptive ways. `serverless-deploy-npm-node-lambda-on-pr-merge.yml` is helpful in this context, but you would generally not need to specify everything quite so much - you would likely name this something like `deploy-lambda-on-pr-merge.yml`. Usually you're not deploying things in multiple ways (`serverless-deploy`), mentioning the dependency installation tool (`npm`), or specifying the language used (`node-lambda`). Use your best judgement!

### Common Naming Usage

* `on-push` - runs when new commit(s) pushed to Github
* `on-pr-merge` - runs when a PR is merged into `main`, usually using `on: push: branches: main`
  * **Why use the `push` event for these?** There is no `merged` type for `pull_request`, only `closed`. This requires you to add `if: github.event.pull_request.merged` statements to validate the workflow started from a merge.
    * You should NOT be pushing to `main`, and generally should have branch protections for you repo that prevent this.
* `on-pr-comment` - runs using the `issue_comment` event (PRs are Issues in Github's APIs) as a trigger, usually with some `if` statements to look for specific comments
* `on-pr-open-update` - runs when a PR is `opened` or `synchronize`d (commits pushed to Github are sync'd from your local)
* `on-dispatch` - these run when triggered by events `workflow_dispatch` (manually in the Github Actions UI for your repo) or `repository_dispatch` (workflows in other repos/anywhere via simple HTTP requests)

## Callouts

Calling out some specific usage in various files.

### Secrets Manager Usage

* [Doppler](https://doppler.com) is used in `build-deploy-react-to-s3-on-pr-merge.yml`
  * Doppler is a secrets management service, and here is used to load environment variables (passwords, config values, etc) into the excution context of the command after `--`
  * Usage of other secrets management services such as [chamber](https://github.com/segmentio/chamber) or [1password](https://1password.com) is usually _very_ similar, so these examples should be easy to translate if you're using something else.
  
### Best Practices

* [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) is a great way to add some structure and meaning to your commit messages, and is a wonderful standard for Github pull request naming - see how that's enforced in `util/validate-semantic-pr-name.yml`
  * Your git commit history will literally make you breakfast in bed if you follow conventional commits!

### Watching Specific Changed Files

* In `util/comment-on-pr-when-specific-files-change.yml` you can see a nifty way of creating some visibility of specific changes in the PR review process. 
      * This workflow utilizes [tj-actions/changed-files](https://github.com/tj-actions/changed-files) to get a list of the files changed in the PR, and perform an action when certain ones have been changed. This is used instead of the `paths` filter provided by Github Actions inentionally - the steps it includes would usually be part of a larger `test-on-push` workflow.
        * You could also do things like limiting building and/or deployment of specific parts of your app to only when they change. Save time and money where you can! 
      * There are an ever-increasing number of ways to comment on a PR in your workflows - the [`github-script`](https://github.com/actions/github-script) method used here is particularly nice, in my opinion. Not only is it provided by Github directly, it also gives you the ability to add logic and dynamicism to your comment creation via Javascript.
        * `github-script` can do a LOT more than just add comments to PRs - being able to use JS (and npm!) and easily interact with Github APIs gives you a lot of power!

## Contributing

### Pull Requests

PRs to this repository are welcome! If you have something that would make a helpful reference for others please do share. 

### Issues

If you see swamp thing, say swamp thing! 

Open an issue if you tried to use something in this repo and it didn't work, notice some bad syntax, or just see a bad practice happening. 

Issues should clearly describe the problem and contain any relevant logs or outputs. Help me help you!