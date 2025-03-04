---
slug: /723462/use-secrets
displayed_sidebar: "current"
category: "guides"
tags: ["go", "python", "nodejs"]
authors: ["Vikram Vaswani"]
date: "2023-03-28"
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# Use Secrets in Dagger

## Introduction

Dagger allows you to utilize confidential information, such as passwords, API keys, SSH keys and so on, when running your Dagger pipelines, without exposing those secrets in plaintext logs, writing them into the filesystem of containers you're building, or inserting them into cache.

This tutorial teaches you the basics of using secrets in Dagger.

## Requirements

This tutorial assumes that:

- You have a Go, Python or Node.js development environment. If not, install [Go](https://go.dev/doc/install), [Python](https://www.python.org/downloads/) or [Node.js](https://nodejs.org/en/download/).
- You have a Dagger SDK installed for one of the above languages. If not, follow the installation instructions for the Dagger [Go](../sdk/go/371491-install.md), [Python](../sdk/python/866944-install.md) or [Node.js](../sdk/nodejs/835948-install.md) SDK.
- You have Docker installed and running on the host system. If not, [install Docker](https://docs.docker.com/engine/install/).

## Create and use a secret

The Dagger API provides the following queries and fields for working with secrets:

- The `setSecret` query creates a new secret from a plaintext value.
- A `Container`'s `withMountedSecret()` field returns the container with the secret mounted at the named  filesystem path.
- A `Container`'s `withSecretVariable()` field returns the container with the secret stored in the named container environment variable.

Once a secret is loaded into Dagger, it can be used in a Dagger pipeline as either a variable or a mounted file. Some Dagger SDK methods additionally accept secrets as native objects.

Let's start with a simple example of setting a secret in a Dagger pipeline and using it in a container.

The following code listing creates a Dagger secret for a GitHub personal access token and then uses the token to authorize a request to the GitHub API. To use this listing, replace the `TOKEN` placeholder with your personal GitHub access token.

<Tabs groupId="language">
<TabItem value="Go">

```go file=./snippets/use-secrets/raw/main.go
```

In this code listing:

- The client's `SetSecret()` method accepts two parameters - a unique name for the secret, and the secret value - and returns a new `Secret` object
- The `WithSecretVariable()` method also accepts two parameters - an environment variable name and a `Secret` object. It returns a container with the secret value assigned to the specified environment variable.
- The environment variable can then be used in subsequent container operations - for example, in a command-line `curl` request, as shown above.

</TabItem>
<TabItem value="Node.js">

```javascript file=./snippets/use-secrets/raw/index.mjs
```

In this code listing:

- The client's `setSecret()` method accepts two parameters - a unique name for the secret, and the secret value - and returns a new `Secret` object
- The `withSecretVariable()` method also accepts two parameters - an environment variable name and a `Secret` object. It returns a container with the secret value assigned to the specified environment variable.
- The environment variable can then be used in subsequent container operations - for example, in a command-line `curl` request, as shown above.

</TabItem>
<TabItem value="Python">

```python file=./snippets/use-secrets/raw/main.py
```

In this code listing:

- The client's `set_secret()` method accepts two parameters - a unique name for the secret, and the secret value - and returns a new `Secret` object
- The `with_secret_variable()` method also accepts two parameters - an environment variable name and a `Secret` object. It returns a container with the secret value assigned to the specified environment variable.
- The environment variable can then be used in subsequent container operations - for example, in a command-line `curl` request, as shown above.

</TabItem>
</Tabs>

## Use secrets from the host environment

Most of the time, it's neither practical nor secure to define plaintext secrets directly in your pipeline. That's why Dagger lets you read secrets from the host, either from host environment variables or from the host filesystem, or from external providers.

Here's a revision of the previous example, where the secret is read from a host environment variable. To use this listing, create a host environment variable named `GH_SECRET` and assign it the value of your GitHub personal access token.

<Tabs groupId="language">
<TabItem value="Go">

```go file=./snippets/use-secrets/host-env/main.go
```

This code listing assumes the existence of a host environment variable named `GH_SECRET` containing the secret value. It performs the following operations:

- It reads the value of the host environment variable using the `os.Getenv()` function.
- It creates a Dagger secret with that value using the `SetSecret()` method.
- It uses the `WithSecretVariable()` method to return a container with the secret value assigned to an environment variable in the container.
- It uses the secret in subsequent container operations, as explained previously.

</TabItem>
<TabItem value="Node.js">

```javascript file=./snippets/use-secrets/host-env/index.mjs
```

This code listing assumes the existence of a host environment variable named `GH_SECRET` containing the secret value. It performs the following operations:

- It reads the value of the host environment variable using the `process.env` object.
- It creates a Dagger secret with that value using the `setSecret()` method.
- It uses the `withSecretVariable()` method to return a container with the secret value assigned to an environment variable in the container.
- The environment variable can then be used in subsequent container operations, as explained previously.

</TabItem>
<TabItem value="Python">

```python file=./snippets/use-secrets/host-env/main.py
```

This code listing assumes the existence of a host environment variable named `GH_SECRET` containing the secret value. It performs the following operations:

- It reads the value of the host environment variable using the `os.environ` object.
- It creates a Dagger secret with that value using the `set_secret()` method.
- It uses the `with_secret_variable()` method to return a container with the secret value assigned to an environment variable in the container.
- The environment variable can then be used in subsequent container operations, as explained previously.

</TabItem>
</Tabs>

## Use secrets from the host filesystem

Dagger also lets you mount secrets as files within a container's filesystem. This is useful for tools that look for secrets in a specific filesystem location - for example, GPG keyrings, SSH keys or file-based authentication tokens.

As an example, consider the [GitHub CLI](https://cli.github.com/), which reads its authentication token from a file stored in the user's home directory at `~/.config/gh/hosts.yml`. The following code listing demonstrates how to mount this file at a specific location in a container as a secret, so that the GitHub CLI is able to find it.

<Tabs groupId="language">
<TabItem value="Go">

```go file=./snippets/use-secrets/host-fs/main.go
```

This code listing assumes the existence of a GitHub CLI configuration file containing an authentication token at `/home/USER/.config/gh/hosts.yml`. It performs the following operations:

- It reads the host file's contents into a string and loads the string into a secret with the Dagger client's `SetSecret()` method. This method returns a new `Secret` object.
- It uses the `WithMountedSecret()` method to return a container with the secret mounted as a file at the given location.
- The GitHub CLI reads this file as needed to perform requested operations. For example, executing the `gh auth status` command in the Dagger pipeline after mounting the secret returns a message indicating that the user is logged-in, testifying to the success of the secret mount.

</TabItem>
<TabItem value="Node.js">

```javascript file=./snippets/use-secrets/host-fs/index.mjs
```

This code listing assumes the existence of a GitHub CLI configuration file containing an authentication token at `/home/USER/.config/gh/hosts.yml`. It performs the following operations:

- It reads the host file's contents into a string and loads the string into a secret with the Dagger client's `setSecret()` method. This method returns a new `Secret` object.
- It uses the `withMountedSecret()` method to return a container with the secret mounted as a file at the given location.
- The GitHub CLI reads this file as needed to perform requested operations. For example, executing the `gh auth status` command in the Dagger pipeline after mounting the secret returns a message indicating that the user is logged-in, testifying to the success of the secret mount.

</TabItem>
<TabItem value="Python">

```python file=./snippets/use-secrets/host-fs/main.py
```

This code listing assumes the existence of a GitHub CLI configuration file containing an authentication token at `/home/USER/.config/gh/hosts.yml`. It performs the following operations:

- It reads the host file's contents into a string and loads the string into a secret with the Dagger client's `set_secret()` method. This method returns a new `Secret` object.
- It uses the `with_mounted_secret()` method to return a container with the secret mounted as a file at the given location.
- The GitHub CLI reads this file as needed to perform requested operations. For example, executing the `gh auth status` command in the Dagger pipeline after mounting the secret returns a message indicating that the user is logged-in, testifying to the success of the secret mount.

</TabItem>
</Tabs>

## Use secrets from an external secret manager

It's also possible to read secrets into Dagger from external secret managers. The following code listing provides an example of using a secret from Google Cloud Secret Manager in a Dagger pipeline.

<Tabs groupId="language">
<TabItem value="Go">

```go file=./snippets/use-secrets/external/main.go
```

This code listing requires the user to replace the `PROJECT-ID` and `SECRET-ID` placeholders with  corresponding Google Cloud project and secret identifiers. It performs the following operations:

- It imports the Dagger and Google Cloud Secret Manager client libraries.
- It uses the Google Cloud Secret Manager client to access and read the specified secret's payload.
- It creates a Dagger secret with that payload using the `SetSecret()` method.
- It uses the `WithSecretVariable()` method to return a container with the secret value assigned to an environment variable in the container.
- It uses the secret to make an authenticated request to the GitHub API, as explained previously.

</TabItem>
<TabItem value="Node.js">

```javascript file=./snippets/use-secrets/external/index.mjs
```

This code listing requires the user to replace the `PROJECT-ID` and `SECRET-ID` placeholders with  corresponding Google Cloud project and secret identifiers. It performs the following operations:

- It imports the Dagger and Google Cloud Secret Manager client libraries.
- It uses the Google Cloud Secret Manager client to access and read the specified secret's payload.
- It creates a Dagger secret with that payload using the `setSecret()` method.
- It uses the `withSecretVariable()` method to return a container with the secret value assigned to an environment variable in the container.
- It uses the secret to make an authenticated request to the GitHub API, as explained previously.

</TabItem>
<TabItem value="Python">

```python file=./snippets/use-secrets/external/main.py
```

This code listing requires the user to replace the `PROJECT-ID` and `SECRET-ID` placeholders with  corresponding Google Cloud project and secret identifiers. It performs the following operations:

- It imports the Dagger and Google Cloud Secret Manager client libraries.
- It uses the Google Cloud Secret Manager client to access and read the specified secret's payload.
- It creates a Dagger secret with that payload using the `set_secret()` method.
- It uses the `with_secret_variable()` method to return a container with the secret value assigned to an environment variable in the container.
- It uses the secret to make an authenticated request to the GitHub API, as explained previously.

</TabItem>
</Tabs>

## Use secrets with Dagger SDK methods

Secrets can also be used natively as inputs to some Dagger SDK methods. Here's an example, which demonstrates logging in to Docker Hub from a Dagger pipeline and publishing a new image. To use this listing, replace the `DOCKER-HUB-USERNAME` and `DOCKER-HUB-PASSWORD` placeholders with your Docker Hub username and password respectively.

<Tabs groupId="language">
<TabItem value="Go">

```go file=./snippets/use-secrets/sdk/main.go
```

In this code listing:

- The client's `SetSecret()` method returns a new `Secret` representing the Docker Hub password.
- The `WithRegistryAuth()` method accepts three parameters - the Docker Hub registry address, the username and the password (as a `Secret`) - and returns a container pre-authenticated for the  registry.
- The `Publish()` method publishes the container to Docker Hub.

</TabItem>
<TabItem value="Node.js">

```javascript file=./snippets/use-secrets/sdk/index.mjs
```

In this code listing:

- The client's `setSecret()` method returns a new `Secret` representing the Docker Hub password.
- The `withRegistryAuth()` method accepts three parameters - the Docker Hub registry address, the username and the password (as a `Secret`) - and returns a container pre-authenticated for the  registry.
- The `publish()` method publishes the container to Docker Hub.

</TabItem>
<TabItem value="Python">

```python file=./snippets/use-secrets/sdk/main.py
```

In this code listing:

- The client's `set_secret()` method returns a new `Secret` representing the Docker Hub password.
- The `with_registry_auth()` method accepts three parameters - the Docker Hub registry address, the username and the password (as a `Secret`) - and returns a container pre-authenticated for the  registry.
- The `publish()` method publishes the container to Docker Hub.

</TabItem>
</Tabs>

## Understand how Dagger secures secrets

Dagger automatically scrubs secrets from its various logs and output streams. This ensures that sensitive data does not leak - for example, in the event of a crash. This applies to secrets stored in both environment variables and file mounts.

The following example demonstrates this feature:

<Tabs groupId="language">
<TabItem value="Go">

```go file=./snippets/use-secrets/security/main.go
```

</TabItem>
<TabItem value="Node.js">

```javascript file=./snippets/use-secrets/security/index.mjs
```

</TabItem>
<TabItem value="Python">

```python file=./snippets/use-secrets/security/main.py
```

</TabItem>
</Tabs>

This listing creates dummy secrets on the host (as an environment variable and a file), loads them into Dagger and then attempts to print them to the console. However, Dagger automatically scrubs the sensitive data before printing it, as shown in the output below:

```shell
secret env data: *** || secret file data:
***
```

## Conclusion

This tutorial walked you through the basics of using secrets in Dagger. It explained the various API methods available to work with secrets and provided examples of using secrets as environment variables, file mounts and native objects.

Use the [API Key Concepts](../api/975146-concepts.mdx) page and the [Go](https://pkg.go.dev/dagger.io/dagger), [Node.js](../sdk/nodejs/reference/modules.md) and [Python](https://dagger-io.readthedocs.org/) SDK References to learn more about Dagger.
