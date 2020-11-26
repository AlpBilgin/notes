# NPM

## default install path

/usr/local/lib/node_modules/npm/bin

## Cache bust
`npm cache clean --force` to clean out unnecessary copies of older package versions. NPM claims that it manages everything properly but they obviously never migrated multiple projects from a public repo to a private repo.

## Adding a repo

npm config set registry <REPO-URL>  =>  introduce new registry to npm

npm adduser => add a new user for the new registry

 "publishConfig": {
    "registry": "<REPO-URL>"
  },  =>.  add to your package.jsons so NPM knows
