# Dorkly Feature Flags
Open Source Feature Flag system.
Dorkly is a git-based open source feature flag backend for [LaunchDarkly](https://launchdarkly.com/features/feature-flags/)'s open source SDKs.
The goal is to provide a simple intuitive way to manage feature flags using existing GitHub workflows. If you're feeling fatigue from too many SaaS tools then this might be if interest to you.

## Status
This project is in the early stages of development. Your feedback is appreciated. Early adopters, tire-kickers, and contributors are welcome. 

## Parity with LaunchDarkly
LaunchDarkly is a powerful feature management system with a lot of features. Dorkly is a subset of that functionality. Here's what is supported (much more to come):
1. One [project](https://docs.launchdarkly.com/home/getting-started/vocabulary#project) per git repo. If you need more projects create more repos.
2. Boolean flags: either on or off, or a percent rollout based on user id
3. Server-side flags and client-side flags (can exclude client-side on a per-flag basis)
4. Secrets management: SDK keys are stored in AWS Secrets Manager and exported as Terraform outputs.

Components include (all managed by the [dorkly-flags Terraform module](https://registry.terraform.io/modules/dorklyorg/dorkly-flags/aws/latest):
1. Feature flag definitions stored as yaml files in a GitHub repository
2. A GitHub Action that reads in human-friendly yaml files, and converts them to an archive format consumed by:
3. A Docker image that runs a backend service that serves the flags to your application. This backend service is a [very thin wrapper](docker/Dockerfile) around the [ld-relay](https://docs.launchdarkly.com/sdk/relay-proxy) appliance running in [offline mode](https://docs.launchdarkly.com/sdk/relay-proxy/offline)

# Getting Started: One time setup
## First steps
1. Determine your [project](https://docs.launchdarkly.com/home/getting-started/vocabulary#project) scope and come up with a short name.
2. Determine your starting [environments](https://docs.launchdarkly.com/home/getting-started/vocabulary#environment). These can be changed later so it's ok to use the defaults.
3. Provision your infrastructure using the [dorkly-flags Terraform module](https://registry.terraform.io/modules/dorklyorg/dorkly-flags/aws/latest). [Example](https://github.com/dorklyorg/terraform-aws-dorkly-flags/blob/main/examples/main/main.tf)

## Setting up your application with a properly configured LaunchDarkly SDK: Server-side (golang example)
TODO: terraform: autogenerate example snippets including the sdk key and backend service url instead of this manual process.
1. Set up your application with the LaunchDarkly SDK. [Helpful doc](https://docs.launchdarkly.com/sdk/server-side)
2. For a quick example check out the [hello-go example](https://github.com/launchdarkly/hello-go/blob/main/main.go#L35) program.
3. Grab the sdk key from either AWS secrets manager or if it is a non-production environment, from the GitHub repo. You'll use it in the next step.
4. Using the hello-go example as a starting point, set the `LAUNCHDARKLY_SDK_KEY` environment variable to the SDK key you used when provisioning the infrastructure.
5. *Not yet implemented but needed for MVP*: Grab the backend service url from the GitHub repo's readme/other file tbd. You'll use it in the next step.
6. Instead of initializing a default client, initialize a client with the url of the backend service:
```golang
    import (
"github.com/launchdarkly/go-server-sdk/v7"
"github.com/launchdarkly/go-server-sdk/v7/ldcomponents"

)
	dorklyConfig := ld.Config{
		ServiceEndpoints: ldcomponents.RelayProxyEndpoints("<YOUR_DORKLY_URL>"),
		Events:           ldcomponents.NoEvents(),
	}

	ldClient, err := ld.MakeCustomClient(dorklySdkKey, dorklyConfig, 5*time.Second)
```

# Common Tasks
## Adding a feature flag
In your newly created GitHub repo you'll notice some example yaml files under the `project/` directory. These are intended to be a starting point, so you can create additional yaml files for your own flags.
1. Create a flag overview file for the project. Each flag's basic properties are defined at the project level. This includes name, description, and the type of flag (boolean, booleanRollout, etc). This file should be created in the `project/flags` directory. The naming convention is `<flagName>.yml`. Check out the example flags as a starting point.
2. Create environment-specific flag config files. Under each environment directory in `project/environments`, create a file with the same name as the flag file. This file will contain the environment-specific configuration for the flag. The naming convention is `<flagName>.yml`. Check out the example flags as a starting point.
3. *Not yet implemented but needed for MVP*: Pull Request checks will validate your changes for well-formedness.
4. Commit your changes to the main branch. The GitHub Action will automatically pick up the changes and update the backend service.

## Changing a feature flag for an environment
1. Update the flag file in the environment directory.
2. Commit your changes to the main branch. The GitHub Action will automatically pick up the changes and update the backend service.

## Adding an environment
1. Navigate to your Terraform config
2. Add a new environment to the `environments` variable and execute the changes.
3. Once your Terraform run has been applied, you can add flag configs for the environment manually, or by copying the contents of an existing environment.

## Helpful Links
* [Feature Flags](https://launchdarkly.com/features/feature-flags/)
* [Relay Proxy Configuration](https://docs.launchdarkly.com/sdk/features/relay-proxy-configuration/proxy-mode)
* [Relay Proxy SDK config](https://docs.launchdarkly.com/sdk/relay-proxy/sdk-config)