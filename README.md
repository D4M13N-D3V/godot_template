# Overview
This is a repository that provides a template for you to start your Godot repositories on GitHub. This repository contains a basic project setup along with CI/CD. By default it will build for windows, linux and the web and upload their artifacts, along with deploying a build to GitHub Pages.
# Recommended Flow
To use this properly we recommend creating pull requests into the main branch, and merging your changes in that way. We recommend making a small change to the workflow file in `/.github/workflows/game_ci.yaml` and changing line `169` from `prerelease: false` to `prerelease: true`. Whenever you want to do a proper release, you can go to the list of releases on your repository and change it to be a full release instead of pre-release. This will allow you to have builds for every version of the game, while also maintaining a selection of stable builds.
# Requirments
In order for the workflows in this repository to work with an existing project it requires certain export profiles to be configured. You need to have a windows, linux, and web export configured using the default names: "Windows Desktop","Linux/X11", and "Web".
# Migrating existing project
If you want to add this to an existing GitHub repository, just copy and paste the `.github` folder into the root of your existing repository. As long as your project has these exports and resides in the root of the repository the GitHub workflows should work fine. 
# Versioning
When you push to the `main` branch the workflow will automatically be executed. The workflow will build for Web,Linux, and Windows, and then upload the build artifacts to the workflow. Then a new step will execute which will generate version numbers based on the commits. You can see below which keywords a commit message must start with to increment a major,minor, or patch number. At the end the workflow will create a release and upload the build artifacts to it.

The keywords for versioning are configured in the `GitVersion.yaml` in the root of the repository. This is the default value.

https://gitversion.net/docs/reference/configuration

```
assembly-versioning-scheme: MajorMinorPatch
mode: ContinuousDelivery
branches: {}
ignore:
  sha: []
merge-message-formats: {}
commit-message-incrementing: Enabled
major-version-bump-message: "^(build|chore|ci|docs|feat|fix|perf|refactor|revert|style|test)(\\([\\w\\s-]*\\))?(!:|:.*\\n\\n((.+\\n)+\\n)?BREAKING CHANGE:\\s.+)"
minor-version-bump-message: "^(feat)(\\([\\w\\s-]*\\))?:"
patch-version-bump-message: "^(build|chore|ci|docs|fix|perf|refactor|revert|style|test)(\\([\\w\\s-]*\\))?:"
```
# Alternative Workflows

### Web,Linux,Windows - No GitHub Pages CI
```
```
### Web Only - GitHub Pages CI
```
```
### Linux,Windows Only
```
```
### Linux Only
```
```
### Windows Only
```
```
