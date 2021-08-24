# Publish npm package/library (Angular) to Azure DevOps Artifacts Feed using Azure Devops Pipeline (YAML)

## Create new project

```
ng new my-workspace --create-application=false
cd my-workspace
ng generate library my-package
```
see: https://angular.io/guide/creating-libraries

Than build you library/package
```
ng build my-package
```
You should see the the package now build in the dist folder

Than add a test app to run your library in:
```
ng g application test-app
```
and replace the whole default generated content from `projects/test-app/src/app/app.component.html` with `<lib-my-package></lib-my-package>` or whatever your package's component selector is.

Run your test application locally:
```
ng serve
```

The test knows where to grab you built package because it looks for it - configuered automatically in the `tsconfig.json`.


## Azure DevOps Pipeline (YAML)

1. install node
2. install packages
3. build own package
4. publish own package

The pipeline needs no extra auth steps to publish packages if Azure DevOps is set up properly but it took me a few attemps to until it worked.

``` YAML
trigger:
- master

pool:
  vmImage: ubuntu-latest

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '14.x'
  displayName: 'Install Node.js'

- script: |
    npm install
  displayName: 'npm install'

- script: | # TODO: change package name
    ng build my-package 
  displayName: 'npm build'

- task: Npm@1
  inputs:
    command: 'publish'
    workingDir: 'dist/my-package' # TODO: change package name
    publishRegistry: 'useFeed'
    publishFeed: '000a000a-000a-000a-0000-0a00a00aaa0' # TODO: change id
```
