# Publish npm package/library (Angular) to Azure DevOps Artifacts Feed using Azure Devops Pipeline (YAML)

You have some reusable Node.js code like an Angular component and you like to make it available as an npm package throughout your company which uses Azure DevOps?
Here is the solution on how to build an Angular library and publish it to Azure Devops Artifacts Feed where it can be consumed by your colleagues.

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

Push to the master branch. The pipeline should start automatically - build the package and deploy it to the artifacts feed.

## Consume your package
Go to the Azrue DevOps Artifacts Feed, find your Package and see further instructions there.

For not windows users:

1. create a file called ```.npmrc``` in your project folder. The content of this file can be copied form the Azrue DevOps Artifacts Feed instructions
2. the secound ```.npmrc``` file which contains the token etc. need to go to your users root folder - just follow the Azrue DevOps Artifacts Feed instructions to create its content

The first installation is not very nice for non windows users but once installed it works quite well - also for other projects and packages.
