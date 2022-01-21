# K8s Service Deployment Pulumi Component Provider (Go)

This repo contains the component resource and generated SDKs for a Pulumi package that can be used to deploy K8s Deployments and Services onto a cluster.

## How Was This Created?
This repo was created using the boilerplate here: https://github.com/pulumi/pulumi-component-provider-go-boilerplate

And then it was updated as per this walkthrough: https://www.youtube.com/watch?v=_RXvNS5N8A8 

The component resource that this repo is based on is the ServiceDeployment.go component resource found here: https://github.com/pulumi/examples/blob/master/kubernetes-go-guestbook/components/serviceDeployment.go 

# Steps of Note to Create the Package and SDKs
As stated above, this package was created by following the tutorial here: https://www.youtube.com/watch?v=_RXvNS5N8A8  

To augment that tutorial, this section identifies any steps that may require additional color commentary or information.

* The README for the boilerplate implies that you just change "xyz" to the name of your package. But there are places where the import is using `github.com/pulumi/pulumi-xyz` where you need to change out that `pulumi` repo name.
  * At least I'm assuming it's required. But maybe that `pulumi` "repo" name is not really a repo name and so you can use `pulumi` but then you might conflict with official `pulumi` stuff? 
* The component resource we are using (see link above) declares a `Deployment` and a `DeploymentArgs` structure and a funcion `NewServiceDeployment`. So the references to the actual component resource should reference `Deployment` and not `NewServiceDeployment` even though the main code will call `NewServiceDeployment`. See `schema.yaml` and `provider.go` for places where the component resource is referenced.
* The input args for the component resource need to have `pulumi:"<lowercaseInputName>"` added after where `<lowercaseInputName>` is the same input name but with a starting lowercase character.
* When using an existing component resource, you will want to change the package line to be a `provider` instead of `main`
* `schema.yaml` file
  * **language** section:
    * *csharp*
      * map the hyphenated naming to a camel-case name since csharp doesn't like hyphens in namespaces as in:
      ```
          namespaces:
            aws-quickstart-vpc: AwsQuickStartVpc
      ```
* `export GOPATH=<SOMEPLACE CLEAN>` 
  * This is needed for the make commands to work
  * If you set it to a local folder, make sure .gitignore is set to ignore `pkg` and `pkg/*`

## Build and Test

```bash
# Build and install the provider (plugin copied to $GOPATH/bin)
make install_provider
# Set PATH to find the provider binary created in previous step
export PATH=$PATH:$PWD/bin

make generate # Or make gen_dotnet_sdk
make build # Or make build_dotnet_sdk
make install # Or make install_dotnet_sdk
```
# Test C#
To use the generted C# package do the following:
- Assuming you ran the `make install`, in your multilanguage package folder you will see a `nuget` folder. This folder contains the C# SDK file package.
- To use it, go to your Pulumi project folder and run:
- In your Pulumi project folder:
  - mkdir nuget
  - cp <MULTILANGUAGE_PACKAGE_FOLDER>/dotnet/binDebug/Pulumi.K8sServiceDeployment.0.0.1.nupkg ./nuget
  - Run
    ```
    dotnet add package --source "<PACKAGE FOLDER>/nuget;https://api.nuget.org/v3/index.json"  Pulumi.K8sServiceDeployment
    ```
- In your Pulumi project file add: 
  ```
  using Pulumi.K8sServiceDeployment
  ```


# Test Node.js SDK
$ make install_nodejs_sdk
$ cd examples/simple
$ yarn install
$ yarn link @pulumi/xyz
$ pulumi stack init test
$ pulumi config set aws:region us-east-1
$ pulumi up
```



----- FROM ORIGINAL BOILERPLATE LIKELY TO BE DELETED ----

An example `StaticPage` [component resource](https://www.pulumi.com/docs/intro/concepts/resources/#components) is available in `provider/pkg/provider/staticPage.go`. This component creates a static web page hosted in an AWS S3 Bucket. There is nothing special about `StaticPage` -- it is a typical component resource written in Go.

The component provider makes component resources available to other languages. The implementation is in `provider/pkg/provider/provider.go`. Each component resource in the provider must have an implementation in the `Construct` function to create an instance of the requested component resource and return its `URN` and state (outputs). There is an initial implementation that demonstrates an implementation of `Construct` for the example `StaticPage` component.

A code generator is available which generates SDKs in TypeScript, Python, Go and .NET which are also checked in to the `sdk` folder. The SDKs are generated from a schema in `schema.yaml`. This file should be kept aligned with the component resources supported by the component provider implementation.

An example of using the `StaticPage` component in TypeScript is in `examples/simple`.

Note that the generated provider plugin (`pulumi-resource-xyz`) must be on your `PATH` to be used by Pulumi deployments. If creating a provider for distribution to other users, you should ensure they install this plugin to their `PATH`.

## Prerequisites

- Go 1.15
- Pulumi CLI
- Node.js (to build the Node.js SDK)
- Yarn (to build the Node.js SDK)
- Python 3.6+ (to build the Python SDK)
- .NET Core SDK (to build the .NET SDK)



## Naming

The `xyz` provider's plugin binary must be named `pulumi-resource-xyz` (in the format `pulumi-resource-<provider>`).

While the provider plugin must follow this naming convention, the SDK package naming can be customized. TODO explain.

## Example component

Let's look at the example `StaticPage` component resource in more detail.

### Schema

The example `StaticPage` component resource is defined in `schema.yaml`:

```yaml
resources:
  xyz:index:StaticPage:
    isComponent: true
    inputProperties:
      indexContent:
        type: string
        description: The HTML content for index.html.
    requiredInputs:
      - indexContent
    properties:
      bucket:
        "$ref": "/aws/v4.0.0/schema.json#/resources/aws:s3%2Fbucket:Bucket"
        description: The bucket resource.
      websiteUrl:
        type: string
        description: The website URL.
    required:
      - bucket
      - websiteUrl
```

The component resource's type token is `xyz:index:StaticPage` in the format of `<package>:<module>:<type>`. In this case, it's in the `xyz` package and `index` module. This is the same type token passed to `RegisterComponentResource` inside the implementation of `NewStaticPage` in `provider/pkg/provider/staticPage.go`, and also the same token referenced in `Construct` in `provider/pkg/provider/provider.go`.

This component has a required `indexContent` input property typed as `string`, and two required output properties: `bucket` and `websiteUrl`. Note that `bucket` is typed as the `aws:s3/bucket:Bucket` resource from the `aws` provider (in the schema the `/` is escaped as `%2F`).

Since this component returns a type from the `aws` provider, each SDK must reference the associated Pulumi `aws` SDK for the language. For the .NET, Node.js, and Python SDKs, dependencies are specified in the `language` section of the schema:

```yaml
language:
  csharp:
    packageReferences:
      Pulumi: 3.*
      Pulumi.Aws: 4.*
  go:
    generateResourceContainerTypes: true
    importBasePath: github.com/pulumi/pulumi-xyz/sdk/go/xyz
  nodejs:
    dependencies:
      "@pulumi/aws": "^4.0.0"
    devDependencies:
      typescript: "^3.7.0"
  python:
    requires:
      pulumi: ">=3.0.0,<4.0.0"
      pulumi-aws: ">=4.0.0,<5.0.0"
```

For the Go SDK, dependencies are specified in the `sdk/go.mod` file.

### Implementation

The implementation of this component is in `provider/pkg/provider/staticPage.go` and the structure of the component's inputs and outputs aligns with what is defined in `schema.yaml`:

```go
// The set of arguments for creating a StaticPage component resource.
type StaticPageArgs struct {
    IndexContent pulumi.StringInput `pulumi:"indexContent"`
}

// The StaticPage component resource.
type StaticPage struct {
    pulumi.ResourceState

    Bucket     *s3.Bucket          `pulumi:"bucket"`
    WebsiteUrl pulumi.StringOutput `pulumi:"websiteUrl"`
}

// NewStaticPage creates a new StaticPage component resource.
func NewStaticPage(ctx *pulumi.Context, name string, args *StaticPageArgs, opts ...pulumi.ResourceOption) (*StaticPage, error) {
    ...
}
```

The provider makes this component resource available in the `construct` function in `provider/pkg/provider/provider.go`. When `construct` is called and the `typ` argument is `xyz:index:StaticPage`, we create an instance of the `StaticPage` component resource and return its `URN` and state.

```go
func constructStaticPage(ctx *pulumi.Context, name string, inputs provider.ConstructInputs,
    options pulumi.ResourceOption) (*provider.ConstructResult, error) {

    // Copy the raw inputs to StaticPageArgs. `inputs.CopyTo` uses the types and `pulumi:` tags
    // on the struct's fields to convert the raw values to the appropriate Input types.
    args := &StaticPageArgs{}
    if err := inputs.CopyTo(args); err != nil {
        return nil, errors.Wrap(err, "setting args")
    }

    // Create the component resource.
    staticPage, err := NewStaticPage(ctx, name, args, options)
    if err != nil {
        return nil, errors.Wrap(err, "creating component")
    }

    // Return the component resource's URN and state. `NewConstructResult` automatically sets the
    // ConstructResult's state based on resource struct fields tagged with `pulumi:` tags with a value
    // that is convertible to `pulumi.Input`.
    return provider.NewConstructResult(staticPage)
}
```
