## Premises & Requirements
- auto bump version in `package.json`
- auto update `CHANGELOG.md`
  - commit list with commit link attached
- auto tag on `local`
- optional: 
  - auto tag on `upstream`
  - auto generate release note
  - no [conventional commit messages](https://www.conventionalcommits.org/en/v1.0.0/) are needed


## Options

|                                            |  changelog-maker|  semantic-release|  lerna| standard-version|
| -------------------------------------------| ----------------|------------------|-------|-----------------|
| auto bump version in `package.json`        | x               | x                |v      |v                |
| auto update `CHANGELOG.md`                 | x               | v                |v ⚠️     |v                |
| no conventional commit messages are needed | v               | x                |x      |x                |
| auto tag (local)                           | x               | x                |v      |v                |
| auto generate release note                 | x               | v                |x      |x                |

---

### 1. [**changelog-maker**](https://github.com/nodejs/changelog-maker)

**changelog-maker** will look at the git log of the current directory, pulling entries since the last tag.

- Pros: 
  - [`nodejs`](https://github.com/nodejs/node/blob/master/doc/changelogs/CHANGELOG_V13.md) style commits list 
- Cons:
  - only print to stdout
  - no auto bump version
  - no auto tag on `local` and `upstream`
  - [does not work on Windows ](https://github.com/nodejs/changelog-maker/issues/63)
- Notes:
  - no conventional commit messages are needed

### 2. [**semantic-release**](https://github.com/semantic-release/semantic-release)

**semantic-release** automates the whole package release workflow including: determining the next version number, generating changelog, release notes and publishing the package.

- Pros: 
  - [auto generate changelog](https://github.com/semantic-release/changelog)
  - auto tag and release note on `upstream`
- Cons:
  - `CHANGELOG.md` will be updated for **each release**
  - no auto bump version in `package.json`
- Notes:
  - conventional commit messages are needed
- References:
  - [repo for demo](https://github.com/taylrj/semantic-release-playground)

### 3. [**lerna**](https://github.com/lerna/lerna)

**Lerna** is a tool that optimizes the workflow around managing multi-package repositories with git and npm.

- Pros: 
  - auto tag on `local` 
  - auto bump version in `package.json`
- Notes:
  - auto generate **broken** `CHANGELOG.md` 
  - conventional commit messages are needed
- References:
  - [repo for demo](https://github.com/taylrj/lerna-playground)

### 4. [**standard-version**](https://github.com/conventional-changelog/standard-version)

Automate versioning and `CHANGELOG.md` generation

- Pros: 
  - auto tag on `local`
  - auto bump version in `package.json`
- Notes:
  - conventional commit messages are needed

