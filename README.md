# Overview
This is a repository that provides a template for you to start your Godot repositories on GitHub. This repository contains a basic project setup along with CI/CD. By default it will build for windows, linux and the web and upload their artifacts, along with deploying a build to GitHubDock Pages.

# Docker Image
This uses a open source docker image made for exporting from godot, it can be found here.
https://github.com/abarichello/godot-ci

# How to deploy to GitHub Pages
Go to the settings of your repository, and then on the left hand side click pages. You should then be able to click a drop down near the top of the page to deploy from a branch, deploy from the gh-pages branch once the workflow has executed.
Example Deployment https://D4M13N-D3V.github.io/godot_template/
![image](https://github.com/DamienTehDemon/godot_template/assets/13697702/80a76d39-65be-4de4-ab47-91fbeb3eabd2)

# Changing the Godot version
You can do this by making a small change to the workflow file in `/.github/workflows/game_ci.yaml`. At the top of the file you will see a environment variable for the godot version, just change this to whatever version you want to use to build your game. By default this is 4.2 right now.
# Recommended Flow
To use this properly we recommend creating pull requests into the main branch, and merging your changes in that way. This is to prevent using all of your action runner quota. We recommend making a small change to the workflow file in `/.github/workflows/game_ci.yaml` and changing line `169` from `prerelease: false` to `prerelease: true`. Whenever you want to do a proper release, you can go to the list of releases on your repository and change it to be a full release instead of pre-release. This will allow you to have builds for every version of the game, while also maintaining a selection of stable builds.
# Requirments
In order for the workflows in this repository to work with an existing project it requires certain export profiles to be configured. You need to have a windows, linux, and web export configured using the default names: "Windows Desktop","Linux/X11", and "Web".
# Migrating existing project
If you want to add this to an existing GitHub repository, just copy and paste the `.github` folder into the root of your existing repository, along with the `export_presets.cfg` file. As long as your project has these exports and resides in the root of the repository the GitHub workflows should work fine. 
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
In order to use these just place a file in the `/.github/workflows/` folder with the contents below. Make sure that it has the file extension .yml.
### Web,Linux,Windows - No GitHub Pages CI
```
name: "Godot 4.1.2 CI/CD"

env:
  GODOT_VERSION: 4.1.2

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  web:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    container:
      image: barichello/godot-ci:4.1.2
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Move export templates into position
        run: |
          mkdir -v -p ~/.local/share/godot/export_templates
          mv /root/.local/share/godot/export_templates/${GODOT_VERSION}.stable ~/.local/share/godot/export_templates/${GODOT_VERSION}.stable
      - name: Create staging directory
        run: mkdir -v -p build/web

      - name: Build
        run: godot -v --export-release --headless "Web"

      - name: Add coi-service-worker
        run: |
          git clone https://github.com/gzuidhof/coi-serviceworker.git
          mv coi-serviceworker/coi-serviceworker.js build/web/coi-serviceworker.js
          sed -i '3 i <script src="coi-serviceworker.js"></script>' build/web/index.html

      - name: Zip Web artifacts
        run: zip -r game_web.zip build/web

      - name: Upload Web artifacts
        uses: actions/upload-artifact@v3
        with:
          name: game_web
          path: game_web.zip
  linux:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    container:
      image: barichello/godot-ci:4.1.2

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Move export templates into position
        run: |
          mkdir -v -p ~/.local/share/godot/export_templates
          mv /root/.local/share/godot/export_templates/${GODOT_VERSION}.stable ~/.local/share/godot/export_templates/${GODOT_VERSION}.stable
      - name: Create staging directory
        run: mkdir -v -p build/linux

      - name: Build for Linux
        run: godot -v --export-release --headless "Linux/X11" --path . --output "build/linux/game.x86_64"

      - name: Zip Linux artifacts
        run: zip -r game_linux.zip build/linux

      - name: Upload Linux artifacts
        uses: actions/upload-artifact@v3
        with:
          name: game_linux
          path: game_linux.zip
  windows:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    container:
      image: barichello/godot-ci:4.1.2

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Move export templates into position
        run: |
          mkdir -v -p ~/.local/share/godot/export_templates
          mv /root/.local/share/godot/export_templates/${GODOT_VERSION}.stable ~/.local/share/godot/export_templates/${GODOT_VERSION}.stable
      - name: Create staging directory
        run: mkdir -v -p build/windows

      - name: Build for windows
        run: godot -v --export-release --headless "Windows Desktop" --path . --output "build/windows/game.x86_64"

      - name: Zip Windows artifacts
        run: zip -r game_windows.zip build/windows

      - name: Upload windows artifacts
        uses: actions/upload-artifact@v3
        with:
          name: game_windows
          path: game_windows.zip
  release:
    needs: [web, linux, windows]
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Set up GitVersion
        uses: gittools/actions/gitversion/setup@v0.9.15
        with:
          versionSpec: '5.x'

      - name: Execute GitVersion
        uses: gittools/actions/gitversion/execute@v0.9.15
        with:
          useConfigFile: true
          configFilePath: GitVersion.yml

      - name: Get branch name
        id: get_branch
        run: echo "BRANCH_NAME=$(echo ${GITHUB_REF#refs/heads/})" >> $GITHUB_ENV

      - name: Download artifacts
        uses: actions/download-artifact@v3
        with:
          name: game_web

      - name: Download Linux artifacts
        uses: actions/download-artifact@v3
        with:
          name: game_linux

      - name: Download Windows artifacts
        uses: actions/download-artifact@v3
        with:
          name: game_windows

      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{env.GitVersion_MajorMinorPatch}}
          release_name: ${{env.GitVersion_MajorMinorPatch}}
          body: |
            Release notes for ${{env.GitVersion_MajorMinorPatch}}
          draft: false
          prerelease: false
    
      - name: test
        run: ls

      - name: Upload Web Release Asset
        id: upload-release-asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: game_web.zip
          asset_name: Godot_Game_Web.zip
          asset_content_type: application/zip

      - name: Upload Linux Release Asset
        id: upload-linux-release-asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: game_linux.zip
          asset_name: Godot_Game_Linux.zip
          asset_content_type: application/zip

      - name: Upload Windows Release Asset
        id: upload-windows-release-asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: game_windows.zip
          asset_name: Godot_Game_Windows.zip
          asset_content_type: application/zip
```
### Web Only - GitHub Pages CI
```name: "Godot 4.1.2 CI/CD"

env:
  GODOT_VERSION: 4.1.2

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  web:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    container:
      image: barichello/godot-ci:4.1.2
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Move export templates into position
        run: |
          mkdir -v -p ~/.local/share/godot/export_templates
          mv /root/.local/share/godot/export_templates/${GODOT_VERSION}.stable ~/.local/share/godot/export_templates/${GODOT_VERSION}.stable
      - name: Create staging directory
        run: mkdir -v -p build/web

      - name: Build
        run: godot -v --export-release --headless "Web"

      - name: Add coi-service-worker
        run: |
          git clone https://github.com/gzuidhof/coi-serviceworker.git
          mv coi-serviceworker/coi-serviceworker.js build/web/coi-serviceworker.js
          sed -i '3 i <script src="coi-serviceworker.js"></script>' build/web/index.html

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3.9.3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build/web
          force_orphan: true
          user_name: "github-ci[bot]"
          user_email: "github-actions[bot]@users.noreply.github.com"
          commit_message: "UPDATE GITHUB PAGES"

      - name: Zip Web artifacts
        run: zip -r game_web.zip build/web

      - name: Upload Web artifacts
        uses: actions/upload-artifact@v3
        with:
          name: game_web
          path: game_web.zip
  release:
    needs: [web, linux, windows]
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Set up GitVersion
        uses: gittools/actions/gitversion/setup@v0.9.15
        with:
          versionSpec: '5.x'

      - name: Execute GitVersion
        uses: gittools/actions/gitversion/execute@v0.9.15
        with:
          useConfigFile: true
          configFilePath: GitVersion.yml

      - name: Get branch name
        id: get_branch
        run: echo "BRANCH_NAME=$(echo ${GITHUB_REF#refs/heads/})" >> $GITHUB_ENV

      - name: Download artifacts
        uses: actions/download-artifact@v3
        with:
          name: game_web

      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{env.GitVersion_MajorMinorPatch}}
          release_name: ${{env.GitVersion_MajorMinorPatch}}
          body: |
            Release notes for ${{env.GitVersion_MajorMinorPatch}}
          draft: false
          prerelease: false
    
      - name: test
        run: ls

      - name: Upload Web Release Asset
        id: upload-release-asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: game_web.zip
          asset_name: Godot_Game_Web.zip
          asset_content_type: application/zip
```
### Linux,Windows Only
```name: "Godot 4.1.2 CI/CD"

env:
  GODOT_VERSION: 4.1.2

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  linux:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    container:
      image: barichello/godot-ci:4.1.2

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Move export templates into position
        run: |
          mkdir -v -p ~/.local/share/godot/export_templates
          mv /root/.local/share/godot/export_templates/${GODOT_VERSION}.stable ~/.local/share/godot/export_templates/${GODOT_VERSION}.stable
      - name: Create staging directory
        run: mkdir -v -p build/linux

      - name: Build for Linux
        run: godot -v --export-release --headless "Linux/X11" --path . --output "build/linux/game.x86_64"

      - name: Zip Linux artifacts
        run: zip -r game_linux.zip build/linux

      - name: Upload Linux artifacts
        uses: actions/upload-artifact@v3
        with:
          name: game_linux
          path: game_linux.zip
  windows:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    container:
      image: barichello/godot-ci:4.1.2

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Move export templates into position
        run: |
          mkdir -v -p ~/.local/share/godot/export_templates
          mv /root/.local/share/godot/export_templates/${GODOT_VERSION}.stable ~/.local/share/godot/export_templates/${GODOT_VERSION}.stable
      - name: Create staging directory
        run: mkdir -v -p build/windows

      - name: Build for windows
        run: godot -v --export-release --headless "Windows Desktop" --path . --output "build/windows/game.x86_64"

      - name: Zip Windows artifacts
        run: zip -r game_windows.zip build/windows

      - name: Upload windows artifacts
        uses: actions/upload-artifact@v3
        with:
          name: game_windows
          path: game_windows.zip
  release:
    needs: [web, linux, windows]
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Set up GitVersion
        uses: gittools/actions/gitversion/setup@v0.9.15
        with:
          versionSpec: '5.x'

      - name: Execute GitVersion
        uses: gittools/actions/gitversion/execute@v0.9.15
        with:
          useConfigFile: true
          configFilePath: GitVersion.yml

      - name: Get branch name
        id: get_branch
        run: echo "BRANCH_NAME=$(echo ${GITHUB_REF#refs/heads/})" >> $GITHUB_ENV

      - name: Download Linux artifacts
        uses: actions/download-artifact@v3
        with:
          name: game_linux

      - name: Download Windows artifacts
        uses: actions/download-artifact@v3
        with:
          name: game_windows

      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{env.GitVersion_MajorMinorPatch}}
          release_name: ${{env.GitVersion_MajorMinorPatch}}
          body: |
            Release notes for ${{env.GitVersion_MajorMinorPatch}}
          draft: false
          prerelease: false

      - name: Upload Linux Release Asset
        id: upload-linux-release-asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: game_linux.zip
          asset_name: Godot_Game_Linux.zip
          asset_content_type: application/zip

      - name: Upload Windows Release Asset
        id: upload-windows-release-asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: game_windows.zip
          asset_name: Godot_Game_Windows.zip
          asset_content_type: application/zip
```
### Linux Only
```
name: "Godot 4.1.2 CI/CD"

env:
  GODOT_VERSION: 4.1.2

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  linux:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    container:
      image: barichello/godot-ci:4.1.2

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Move export templates into position
        run: |
          mkdir -v -p ~/.local/share/godot/export_templates
          mv /root/.local/share/godot/export_templates/${GODOT_VERSION}.stable ~/.local/share/godot/export_templates/${GODOT_VERSION}.stable
      - name: Create staging directory
        run: mkdir -v -p build/linux

      - name: Build for Linux
        run: godot -v --export-release --headless "Linux/X11" --path . --output "build/linux/game.x86_64"

      - name: Zip Linux artifacts
        run: zip -r game_linux.zip build/linux

      - name: Upload Linux artifacts
        uses: actions/upload-artifact@v3
        with:
          name: game_linux
          path: game_linux.zip
  release:
    needs: [web, linux, windows]
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Set up GitVersion
        uses: gittools/actions/gitversion/setup@v0.9.15
        with:
          versionSpec: '5.x'

      - name: Execute GitVersion
        uses: gittools/actions/gitversion/execute@v0.9.15
        with:
          useConfigFile: true
          configFilePath: GitVersion.yml

      - name: Get branch name
        id: get_branch
        run: echo "BRANCH_NAME=$(echo ${GITHUB_REF#refs/heads/})" >> $GITHUB_ENV

      - name: Download Linux artifacts
        uses: actions/download-artifact@v3
        with:
          name: game_linux

      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{env.GitVersion_MajorMinorPatch}}
          release_name: ${{env.GitVersion_MajorMinorPatch}}
          body: |
            Release notes for ${{env.GitVersion_MajorMinorPatch}}
          draft: false
          prerelease: false

      - name: Upload Linux Release Asset
        id: upload-linux-release-asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: game_linux.zip
          asset_name: Godot_Game_Linux.zip
          asset_content_type: application/zip
```
### Windows Only
```
name: "Godot 4.1.2 CI/CD"

env:
  GODOT_VERSION: 4.1.2

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  windows:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    container:
      image: barichello/godot-ci:4.1.2

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Move export templates into position
        run: |
          mkdir -v -p ~/.local/share/godot/export_templates
          mv /root/.local/share/godot/export_templates/${GODOT_VERSION}.stable ~/.local/share/godot/export_templates/${GODOT_VERSION}.stable
      - name: Create staging directory
        run: mkdir -v -p build/windows

      - name: Build for windows
        run: godot -v --export-release --headless "Windows Desktop" --path . --output "build/windows/game.x86_64"

      - name: Zip Windows artifacts
        run: zip -r game_windows.zip build/windows

      - name: Upload windows artifacts
        uses: actions/upload-artifact@v3
        with:
          name: game_windows
          path: game_windows.zip
  release:
    needs: [web, linux, windows]
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Set up GitVersion
        uses: gittools/actions/gitversion/setup@v0.9.15
        with:
          versionSpec: '5.x'

      - name: Execute GitVersion
        uses: gittools/actions/gitversion/execute@v0.9.15
        with:
          useConfigFile: true
          configFilePath: GitVersion.yml

      - name: Get branch name
        id: get_branch
        run: echo "BRANCH_NAME=$(echo ${GITHUB_REF#refs/heads/})" >> $GITHUB_ENV

      - name: Download Windows artifacts
        uses: actions/download-artifact@v3
        with:
          name: game_windows

      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{env.GitVersion_MajorMinorPatch}}
          release_name: ${{env.GitVersion_MajorMinorPatch}}
          body: |
            Release notes for ${{env.GitVersion_MajorMinorPatch}}
          draft: false
          prerelease: false

      - name: Upload Windows Release Asset
        id: upload-windows-release-asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: game_windows.zip
          asset_name: Godot_Game_Windows.zip
          asset_content_type: application/zip
```
