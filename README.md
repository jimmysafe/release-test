# Generate Changelog with GH Actions

Uses commitlint to keep commit msg standard Conventional Commits.

### Install dependencies

```
yarn add -D husky release-it @release-it/conventional-changelog @commitlint/cli @commitlint/config-conventional
```

### package.json

add the following at the bottom of you package.json file

```
"release-it": {
	"git": {
		"requireBranch": "master",
		"commitMessage": "chore: release v${version}"
	},
	"hooks": {
		"before:init": [
			"git pull origin master"
		]
	},
	"github": {
		"release": true
	},
	"npm": {
		"publish": false
	},
	"plugins": {
		"@release-it/conventional-changelog": {
			"infile": "CHANGELOG.md",
			"preset": {
				"name": "conventionalcommits",
				"types": [
					{
						"type": "feat",
						"section": "Features"
					},
					{
						"type": "fix",
						"section": "Bug Fixes"
					}
				]
			}
		}
	}
}
```

### Add release script

`"release": "release-it"`

### Create Commitlint config file

in the root of your project create `commitlint.config.ts`

file content:
`export default { extends: ["@commitlint/config-conventional"] };`

### Husky

initialize husky
`npx husky install`

create `commit-msg` husky file
`npx husky add .husky/commit-msg "npx commitlint --edit"`

### Github Action

inside `.github/workflows` create `release.yml` file

file content:

```
name: "release"

on:
	push:
		branches:
			- master

jobs:
	release:
		name: "Release"
		runs-on: "ubuntu-latest"

		steps:
			- name: Checkout
			uses: actions/checkout@v2
			with:
				fetch-depth: 0

			- name: Install Dependencies
			run: yarn install --frozen-lockfile

			- name: Initialize Git User
			run: |
				git config --global user.email "ciaffardini.g@gmail.com"
				git config --global user.name "jimmysafe"

			- name: Run Release
			run: yarn release --ci
			env:
				GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Now on push to master the action should fire and create a changelog file.
