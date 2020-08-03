# create-aws-cdk-app
> Create an AWS CDK typescript project with sane defaults

After working with the AWS CDK for a bit I've noticed that some parts of it are really non-idiomatic on the js community, so I've decided to create this project with the goals of developing a solution similar to `create-react-app` for AWS CDK projects, especifically typescript ones. At the moment I'll simply use this repo to make a list of all things that could be improved and once I believe the list is good enough I'll write the code and publish it on npm.

Note that some of the things on this list can be explained by the fact that the CDK was made to be used with a wide set of languages, so I believe that it'll be easy to improve on them by just strictly targeting typescript.

## Things to improve
### Global install
All the documentation on the cdk requests that you install the package globally and just use that, however this is a nightmare because it can easily lead to compatibility probolems between different people working on the same project, furthemore it is better to isolate the dependencies on a per-project basis.

A problem that is also connected to this is the fact that currently some of the commands available are npm scripts (eg:`npm run build`) while some others make use of the global cdk installation (eg:`cdk synth`). The convention is to have all commands as npm scripts, which also make it easier for newcomers into the project to learn what is available by just looking at `package.json`.

Something that is also heavily counter-intuitive is that if you run the local version of the cdk with `npm run cdk` the build step (`npm run build`) is not executed, whereas if you use the global version it is.

### Typescript artifacts
The typescript configuration generated by `cdk init` makes typescript generate its build artifacts right next to the source code files, so your folder structure ends up heavily polluted by .js and .d.ts files which should not be edited given that they were generated. The end result is that 66% of the files in your source directory are just build artifacts.

Furthermore, to prevent these artifacts from being checked in to source control, the `.gitignore` file contains these patterns:
```
*.js
*.d.ts
```
Which make it impossible to have declaration files or javascript files mixed with your code.

### Jest tests
The tests set up by `cdk init` use a different `expect` and don't follow the conventions of jest, resulting in code like the following:
```typescript
import { expect as expectCDK, haveResource } from '@aws-cdk/assert';
import * as cdk from '@aws-cdk/core';
import * as CdkWorkshop from '../lib/cdk-workshop-stack'

test('SNS Topic Created', () => {
  const app = new cdk.App();
  const stack = new CdkWorkshop.CdkWorkshopStack(app, 'MyTestStack');
  expectCDK(stack).to(haveResource("AWS::SNS::Topic"));
});
```

I believe it'd be better to set up the new matchers as a jest preset, which would transform the previous code to the following:
```typescript
import * as cdk from '@aws-cdk/core';
import * as CdkWorkshop from '../lib/cdk-workshop-stack'

test('SNS Topic Created', () => {
  const app = new cdk.App();
  const stack = new CdkWorkshop.CdkWorkshopStack(app, 'MyTestStack');
  expectCDK(stack).toHaveResource("AWS::SNS::Topic");
});
```

### Folder structure
A new generated CDK project has the following folders:
```
bin       jest.config.js  package.json       test
cdk.json  lib             package-lock.json  tsconfig.json
cdk.out   node_modules    README.md
```

As you can see, these don't really follow established conventions. What is the bin folder supposed to contain? Why is there no `src` folder? ...

### Working directory requirements
`cdk synth` requires the working directory to be the one where `cdk.json` is located.

### npm test can't be executed without a previous npm run build
Otherwise it results in the error:
```
ENOENT: no such file or directory, stat '/home/.../dist'
```
