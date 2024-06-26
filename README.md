# @diotoborg/consequatur-facilis-qui

_Package and deploy CloudFormation templates with a simple CLI_

Pacploy is a command-line utility to deploy
[AWS CloudFormation](https://aws.amazon.com/cloudformation) templates which
supports packaging nested templates and local artifacts. It provides one-liner
commands to manage infrastructure as code without enforcing an opinionated
framework. Written in Node.js, it integrates smoothly with your existing
npm/yarn projects.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

## Table of Contents

- [Getting started](#getting-started)
- [Usage](#usage)
  - [Available commands](#available-commands)
  - [Deploy a stack](#deploy-a-stack)
  - [Download stack outputs](#download-stack-outputs)
  - [Delete a stack](#delete-a-stack)
  - [Retrieve stack status](#retrieve-stack-status)
  - [Display errors](#display-errors)
  - [Cleanup retained resources](#cleanup-retained-resources)
  - [Package local dependencies](#package-local-dependencies)
  - [Zip an artifact](#zip-an-artifact)
- [Configuration](#configuration)
  - [Deployment](#deployment)
  - [Packaging local artifacts](#packaging-local-artifacts)
  - [AWS credentials](#aws-credentials)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Getting started

Install @diotoborg/consequatur-facilis-qui using [`npm`](https://www.npmjs.com/package/@diotoborg/consequatur-facilis-qui):

```bash
npm install --save-dev @diotoborg/consequatur-facilis-qui
```

You can then deploy a local template using the following command (e.g. in a
[npm script](https://docs.npmjs.com/cli/v7/using-npm/scripts)):

```bash
@diotoborg/consequatur-facilis-qui deploy --template-path 'template.yaml' --stack-name 'my-stack'
```

See below for all available commands and configuration.

## Usage

### Available commands

```bash
$ @diotoborg/consequatur-facilis-qui --help
@diotoborg/consequatur-facilis-qui <command>

Commands:
  @diotoborg/consequatur-facilis-qui deploy     Deploy a stack
  @diotoborg/consequatur-facilis-qui sync       Download stack outputs in JSON or .env format
  @diotoborg/consequatur-facilis-qui delete     Delete a stack
  @diotoborg/consequatur-facilis-qui package    Package dependencies to S3 or ECR
  @diotoborg/consequatur-facilis-qui status     Retrieve the status of a stack
  @diotoborg/consequatur-facilis-qui errors     Display the latest errors on a stack
  @diotoborg/consequatur-facilis-qui cleanup    Delete retained resources and artifacts created by a stack
                     but not associated with it anymore
  @diotoborg/consequatur-facilis-qui zip [dir]  Create a zip archive of a package

Options:
  --help     Show help [boolean]
  --version  Show version number [boolean]
  --config   The path to a config file [string]
```

See [deployment configuration](#deployment) for more information on the
`--config` option, and below for details on each command.

### Deploy a stack

```bash
$ @diotoborg/consequatur-facilis-qui deploy --help

Options:
  --template-path               The local path to the template to deploy
                                [string] [required]
  --stack-name                  The stack name or stack id [string] [required]
  --region                      The region in which the stack is deployed
                                [string] [required]
  --force-delete                If set, will not ask for confirmation to delete
                                the stack and associated resources if needed
                                [boolean] [default:false]
  --force-upload                If set, will upload local artifacts even if
                                they are up-to-date [boolean] [default:false]
  --deploy-bucket, --s3-bucket  The name of a S3 bucket to upload the template
                                resources [string]
  --deploy-ecr, --ecr           The URI of an Elastic Container Registry to
                                deploy docker images [string]
  --sync-path                   Path to where stack outputs should be saved
                                [string]
  --no-prune                    If set, will not prune unused packaged files
                                associated with supplied stack from deployment
                                bucket [boolean] [default: false]
  --cleanup                     If set, will delete retained resources
                                associated with the supplied stack [boolean]
                                [default: false]
```

This command bundles several operations to go from a local template to a live
infrastructure in a single command:

- [`package`](#package-local-dependencies) local artifacts to S3 or ECR and
  update the templates with the uploaded location
- create a change set for the stack and execute it, then monitor the deployment
- if `--sync-path` is specified, [`sync`](#download-stack-outputs) the stack's
  outputs locally
- if the stack is in a non-deployable [`status`](#retrieve-stack-status),
  propose to [`delete`](#delete-a-stack) it
- [`cleanup`](#cleanup-retained-resources) deployment bucket (by default unless
  `--no-prune` is passed) and retained resources (if `--cleanup` is set)

Check the [Configuration](#Configuration) section for additional options
available only through config files.

### Download stack outputs

```bash
$ @diotoborg/consequatur-facilis-qui sync --help

Options:
  --stack-name       The stack name or stack id [string] [required]
  --region           The region in which the stack is deployed
                     [string] [required]
  --sync-path, --to  Path to where stack outputs should be saved. Format is
                     determined by file extension (supported: .json, .env)
                     [string]
  --no-override      If provided, the command will not override existing file
                     [boolean] [default: false]
```

Download a stack's outputs (as defined in the template's
[`Outputs`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html)
section) into a local JSON file. This enables to access stack information such
as deployed resource ids.  
You usually won't need to run this command separately as it is included in the
[`deploy`](#deploy-a-stack) command when passing the `--sync-path` option.

### Delete a stack

```bash
$ @diotoborg/consequatur-facilis-qui delete --help

Options:
  --stack-name    The stack name or stack id [string] [required]
  --region        The region in which the stack is deployed [string] [required]
  --force-delete  If set, will not ask for confirmation before deleting the
                  stack and associated resources [boolean] [default: false]
  --no-prune      If set, will not prune unused packaged files associated with
                  supplied stack from deployment bucket [boolean]
                  [default: false]
  --cleanup       If set, will not delete retained resources associated with
                  the supplied stack [boolean] [default: false]
```

Delete a stack, and optionally retained resources (see the
[`cleanup`](#cleanup-retained-resources) command for more info).  
By default, the command assumes that it is running in an interactive
environment and will ask for confirmation before deleting anything. If this is
not the case, use `--force-delete` to skip the confirmation prompts.

### Retrieve stack status

```bash
$ @diotoborg/consequatur-facilis-qui status --help

Options:
  --stack-name  The stack name or stack id [string] [required]
  --region      The region in which the stack is deployed [string] [required]
  -q, --quiet   Don't format the status, only display the command output
                [boolean] [default: true]
```

Retrieve the current status of a stack. This is mostly used internally by other
commands but also made available as is.

### Display errors

```bash
$ @diotoborg/consequatur-facilis-qui errors --help

Options:
  --stack-name  The stack name or stack id [string] [required]
  --region      The region in which the stack is deployed [string] [required]
```

Display the last error messages that occurred during deployment of the stack.
This will recursively parse nested templates and display only resources with a
meaningful error message. This enables to stay in the CLI to debug a failed
deployment instead of switching back and forth with the AWS console and
navigating across stacks to get that information.  
Usually you won't need to run this command as it is automatically invoked as
part of the [`deploy`](#deploy-a-stack) command.

### Cleanup retained resources

```bash
$ @diotoborg/consequatur-facilis-qui cleanup --help

Options:
  --stack-name                  The stack name or stack id [string] [required]
  --region                      The region in which the stack is deployed
                                [string] [required]
  --deploy-bucket, --s3-bucket  The name of a S3 bucket to upload the template
                                resources [string]
  --force-delete                If set, will not ask for confirmation to delete
                                [boolean] [default: false]
  --no-prune                    If set, will not prune unused packaged files
                                associated with supplied stack from deployment
                                bucket [boolean] [default: false]
  --no-retained                 If set, will not delete retained resources
                                associated with the supplied stack [boolean]
                                [default: false]
```

Certain resources are not deleted with the stack, such as S3 buckets which are
often [marked to be retained](https://aws.amazon.com/premiumsupport/knowledge-center/delete-cf-stack-retain-resources/)
as they need to be emptied first. This command automates the deletion of such
resources.  
It also purges unused artifacts from the S3 bucket used to
[`package`](#package-local-dependencies) the specified stack (artifacts not
associated with the specified stack won't be removed). This can reduce
unnecessary storage costs as the bucket grows larger with each artifact update
(the [`package`](#package-local-dependencies) command is able to check if an
artifact already exists, but won't remove previous ones after updates).  
For a list of supported resources, check
[supported.js](./src/Command/cleanup/supported.js).

### Package local dependencies

```bash
$ @diotoborg/consequatur-facilis-qui package --help

Options:
  --template-path               The local path to the template to deploy
                                [string] [required]
  --deploy-bucket, --s3-bucket  The name of a S3 bucket to upload the artifacts
                                [string]
  --deploy-ecr, --ecr           The URI of an Elastic Container Registry to
                                deploy docker images [string]
  --region                      The region of the S3 bucket or the ECR
                                [string] [required]
  --force-upload                If set, will upload local artifacts even if
                                they are up-to-date [boolean] [default:false]
  --zip-to                      If provided, will also zip the packaged
                                template to the specified path [string]
```

Parse the supplied template for resources that point to local artifacts, and
package those to S3 or ECR. See
[ResourceProperty.js](./src/Command/pkg/ResourceProperty.js) for the list of
supported resources and packaging destinations: it extends on
[the list supported by the aws-cli](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cloudformation/package.html).
This command is invoked by [`deploy`](#deploy-a-stack) so you usually won't
need to package your dependencies separately.  
S3 artifacts will be [`zip`](#zip-an-artifact)ped into an archive format
supported by CloudFormation. If the archive hasn't been updated since the last
packaging (based on its md5 hash), it won't be uploaded again unless
`--force-upload` is specified.  
For Docker artifacts (i.e. either a `dockerfile`, a directory containing a
`dockerfile` or a `tar` archive), docker images will be built and uploaded to
the supplied ECR. Only updated layers will be re-uploaded unless
`--force-upload` is specified. Note that your system must have
[docker](https://www.docker.com/) installed to build images (check out
[sys-dependencies](https://github.com/romainv/sys-dependencies) to automate the
installation).  
If the supplied template includes nested templates, those will be parsed as
well and transformed to point to the new location of their dependencies. Nested
templates will in turn be uploaded to S3 and their parent updated to point to
their S3 location, recursively.  
Optionally, with the `--zip-to` option the supplied template can also be zipped
together with its parameters and tags in a format recognized by
[AWS CodePipeline](https://docs.aws.amazon.com/codepipeline/latest/userguide/integrations-action-type.html#integrations-deploy-CloudFormation)
so you can deploy it as part of your CI/CD pipeline.

Check the [Configuration](#packaging-local-artifacts) section for additional
options available only through config files.

### Zip an artifact

```bash
$ @diotoborg/consequatur-facilis-qui zip --help
@diotoborg/consequatur-facilis-qui zip [dir]

Positionals:
  dir  Path to the package to zip [string] [default:pwd]

Options:
  --zip-to, --to  The destination path of the zip file. If omitted, will
                  generate a name based on the package dir [string]
  --bundle-dir    The parent folder under which to add bundled dependencies
                  [string] [default: "node_modules"]
  --format        The archive format to use ("zip" or "tar")
                  [string] [default: "zip"]
```

Create an archive file from the current or supplied directory following the
packaging configuration (see
[Packaging local artifacts](#packaging-local-artifacts) to understand how to
bundle npm dependencies or ignore certain files).  
You typically won't need this command as such, and rather use the
[`package`](#package-local-dependencies) or the [`deploy`](#deploy-a-stack)
commands directly, but this could be useful for debug purposes to verify which
files get packaged.

## Configuration

### Deploying a single stack

Besides the CLI options described above for all
[available commands](#available-commands), you can configure @diotoborg/consequatur-facilis-qui's
deployment with a config file specified as follows, in order of precedence:

- using the `--config` option
- using a `@diotoborg/consequatur-facilis-qui.config.js` or `@diotoborg/consequatur-facilis-qui.config.json` file at the root of your
  project
- using a `@diotoborg/consequatur-facilis-qui` entry in the `package.json` file of your npm/yarn project

The configuration object is a map of options in
[camelCase](https://en.wikipedia.org/wiki/Camel_case).  
For [`deploy`](#deploy-a-stack), you can specify `stackParameters` and
`stackTags` in JSON format besides the options listed in [usage](#usage). For
example:

```json
{
  "region": "us-east-1",
  "stackName": "my-stack",
  "templatePath": "./template.yaml",
  "stackParameters": {
    "foo": "bar"
  },
  "stackTags": {
    "App": "MyApp"
  }
}
```

### Deploying a set of stacks

Pacploy supports managing a set of inter-dependent stacks via a config file.
To enable deploying, deleting or syncing multiple stacks in a single command,
specify a list of individual configuration objects.  
It is possible to define the relationship between stacks by specifying on
which stacks they depend with the `dependsOn` parameter. The deployment and
deletion commands will take this into account to optimize the overall
deployment while respecting the dependencies. Dependencies can be cross-region
as well.  
This also enables to use a dependent stack's outputs as parameters by passing
the `stackParameters` as a function which takes as input an object where
the outputs are mapped to the dependent stack's name.  
For instance the below config file enables deploys a stack and its dependent:

```js
// @diotoborg/consequatur-facilis-qui.config.js

// Sample config which deploys a set of inter-dependent stacks
const region = "us-east-1"
export default [
  // This stack creates a S3 bucket used to deploy resources of other stacks
  {
    region,
    stackName: "stack-deploy",
    templatePath: "deployment_infra.yaml",
  },
  // This stack uses the deployment bucket to package its local artifacts
  {
    region,
    stackName: "stack-A",
    templatePath: "stackA.yaml",
    dependsOn: [{ region, name: "stack-deploy" }],
    deployBucket: ({ "stack-deploy": { bucketName } = {} }) => bucketName,
    syncPath: [".env", "stackA.json"],
  },
  // This stack uses the deployment bucket and the outputs of stack A
  {
    region,
    stackName: "stack-B",
    templatePath: "stackB.yaml",
    dependsOn: [{ region, name: "stack-A" }],
    stackParameters: ({ "stack-A": { output1, output2 } = {} }) => ({
      output1,
      output2,
    }),
    syncPath: ".env", // Will merge with stack-A outputs
  },
]
```

### Packaging local artifacts

As described in the [`package`](#package-local-dependencies) command, you can
point to local artifacts in [supported](./src/Command/pkg/ResourceProperty.js)
resource properties of your template. To customize how these artifacts are
packaged, you can, in order of precedence:

- place a `.@diotoborg/consequatur-facilis-quirc` file in the target directory or higher in the tree
- specify the packaging options directly in your npm/yarn's `package.json`
  (some options overlap with
  [npm's specifications](https://docs.npmjs.com/cli/v7/configuring-npm/package-json))

The options are specified as follows:

```json
{
  "main": "src/index.js",
  "files": ["*.js"],
  "bundledDependencies": ["cfn-response", "node-fetch"],
  "ignore": ["node_modules/aws-sdk"],
  "dockerBuild": {
    "buildargs": {
      "NODE_VERSION": "v14.17.6",
      "ENV_VAR": "${ENV_VAR}"
    }
  },
  "root": "."
}
```

Here is what these options do:

- `main` (optional) is usually specified in npm's `package.json` and will
  always be included in the package
- `files` (optional) is the list of glob patterns that should be included in
  the archive
- `bundledDependencies` (optional) are the list of npm dependencies which
  should be bundled. By default, they will be nested under a `node_modules`
  folder in your package, unless your resource is a
  [Lambda layer](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)
  in which case the dependencies will be nested under a `nodejs/node_modules`
  folder
- `ignore` (optional) specifies the globs to exclude from the archive. If
  omitted, @diotoborg/consequatur-facilis-qui will look for the following files up in the directory, in
  order of precedence: `.@diotoborg/consequatur-facilis-quiignore`, `.dockerignore`, `.npmignore`,
  `.gitignore`
- `dockerBuild` (optional) provides options to build a docker image (see
  [Docker's documentation](https://docs.docker.com/engine/api/v1.37/#operation/ImageBuild)
  for available options). It is possible to reference environment variables in
  buildargs values, using the format `${ENV_VAR}`. Such references will be
  replaced by their value in the environment (`process.env["ENV_VAR"]`)
- `root` (optional) can be specified to change the default root folder used to
  resolve the relative paths specified in `files`. This can be useful to include
  files from parent folders in the package. If `ignore` is specified in
  `.@diotoborg/consequatur-facilis-quirc`, the ignored globs will be evaluated against the specified `root`
  but if `ignore` is not specified and a `.<prefix>ignore` file is used instead,
  the globs will be evaluated relative to that file

### AWS credentials

You will need sufficient permissions to deploy your CloudFormation template,
including packaging local artifacts, performing actions to manage your stack as
well as granting permissions to your stack resources.  
Pacploy uses the official
[AWS JS SDK](https://aws.amazon.com/sdk-for-javascript/) to execute AWS
commands, so it is compatible with all the supported ways to setup credentials
in Node.js: please refer to the
[official doc](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/setting-credentials-node.html)
for more info.

## License

Pacploy is [MIT licensed](./LICENSE)
