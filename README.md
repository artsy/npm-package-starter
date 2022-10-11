# @artsy/npm-package-starter

A starter repo for publishing `@artsy` packages to NPM, which includes common CI setup and PR-label based semantic releases via [Auto](https://intuit.github.io/auto/index).

## Commands

```bash
yarn test
yarn type-check
yarn lint
yarn compile
yarn watch
```

## Overview

There are a few key tools that we use for our publishing pipeline. Notably

- [Auto](https://github.com/intuit/auto): Generate releases based on semantic version labels on pull requests
- [Artsy's CircleCI Orbs](https://github.com/artsy/orbs): Orbs are packages of CircleCI configuration that can be shared across projects

Combining these things together, we get seamless DX around releases that can be triggered directly from PRs via labels, and minimal CI config for running common script commands in an ideal way. Additionally, an automated changelog is generated for every release.

## Setup

Since much of this functionality relies on Artsy's Github and CircleCI org membership, there are a few setup steps before everything will work:

- Create a new repo on Github under the Artsy namespace and copy the files from this example repo into it. Commit and push to `main`.
- Go to https://github.com/artsy/your-repo-name/settings/access, click `Add Teams`, and then add `Engineering`
- Then [go to CircleCI](https://app.circleci.com/projects/project-dashboard/github/artsy/), find the new repo, and click the little "gear" icon to set up
- As CircleCI should automatically pick up the [`.circleci/config.yml` file](https://github.com/artsy/npm-package-starter/blob/main/.circleci/config.yml), select the `Fastest` option and enter `main` for the branch. Click `Set up Project`
- Add `NPM_TOKEN` and `GH_TOKEN` environment variables via the [settings page](https://app.circleci.com/settings/project/github/artsy/your-app-name/environment-variables) (details are in 1pass).
- This will allow Auto to write a message to the GitHub PR notifying devs that the package has been published, along with the version number, as well as increment the version number in `package.json`
- Update the package name in [`package.json`](https://github.com/artsy/npm-package-starter/blob/main/package.json#L2)
- Done!

Once these steps are completed the new repo should be able to access NPM authentication, our shared Auto config and more.

## Typical Developer Workflow

- Open PR
- Tag PR with correct semantic-release label (by default, `patch` is applied)
- Merge PR
- Release is sent to NPM and the merged PR is updated with a comment indicating the version that was just published
- If needing to release a `canary` version for testing, tag PR with a `canary` label. In a few moments the PR description will update with a link to the canary that was just published

### Deeper Dive

For those looking to understand how these pieces fit together within our Github Org, read on!

- The file responsible for connecting label-based publishing together is [`.autorc`](https://github.com/artsy/npm-package-starter/blob/main/.autorc)
- Within `.autorc` is `"extends": "@artsy"`, which points at our [shared Auto config](https://github.com/artsy/auto-config). This is where we define the various labels that will perform various functions
- For running tests and other chores on CI, [see `.circleci/config.yml`](https://github.com/artsy/npm-package-starter/blob/main/.circleci/config.yml)
- Inside of that file you'll see jobs like [`yarn/test`](https://github.com/artsy/npm-package-starter/blob/main/.circleci/config.yml#L20).
- Notice how there's nothing pointing to a script in `package.json`? This is due to how we've setup [sensible defaults](https://github.com/artsy/orbs/blob/main/src/yarn/yarn.yml#L110) via CircleCI Orbs. It is assumed that one has these scripts already setup in their `package.json`.
- All of the various authentication-token needs are also managed from within here, in the `pre-release` [job](https://github.com/artsy/orbs/blob/main/src/yarn/yarn.yml#L65-L71)
