---
title: Authorization using pxctl
linkTitle: Authorization
keywords: pxctl, command-line tool, cli, reference, authorization, auth-enabled, token, self-signed, JWT, shared-secret, security
description: Learn to interact with your authorization-enabled Portworx cluster using pxctl
weight: 1800
---

## Overview

This page outlines how to interact with an authorization-enabled Portworx cluster. The main way to do it is by using the `pxctl context` command. Also, you can integrate with a generated token through `pxctl` command-line tool. Use your own JWT compliant application to generate security token. For more details, refer to the [Generate tokens](/reference/cli/generate-token) page.

## Contexts

Portworx stores the following locally to your home directory, allowing you to switch between configurations with a few commands:

 - contexts
 - associated clusters
 - privileges
 - tokens


{{<info>}}
**Note:**
Since Portworx stores the context locally on each node, you must create your context on the node you're working on.
{{</info>}}

Run the `pxctl context` command with the `--help` flag to list the available subcommands and flags:

```text
/opt/pwx/bin/pxctl context --help
```

```output
Portworx pxctl context commands for setting authentication and connection info

Usage:
  pxctl context [flags]
  pxctl context [command]

Aliases:
  context, ctx

Available Commands:
  create      create a context
  delete      delete a context
  list        list all contexts
  set         set the current context
  unset       unset the current context

Flags:
  -h, --help   help for context

Global Flags:
      --ca string            path to root certificate for ssl usage
      --cert string          path to client certificate for ssl usage
      --color                output with color coding
      --config string        config file (default is $HOME/.pxctl.yaml)
      --context string       context name that overrides the current auth context
  -j, --json                 output in json
      --key string           path to client key for ssl usage
      --output-type string   use "wide" to show more details
      --raw                  raw CLI output for instrumentation
      --ssl                  ssl enabled for portworx

Use "pxctl context [command] --help" for more information about a command.
```

### Create a context

You can create a new context by running the `pxctl context create` command and passing it the following arguments:

- the name of the context
- `--token` with the token Portworx must use for this context
- `--endpoint` with the endpoint for this context

Here's an example of how you can create a new context:

```text
pxctl context create <context> --token <token> --endpoint <endpoint>
```

### Delete a context

To delete a context, run the `pxctl context delete` command with the name of the context as in the following example:

```text
pxctl context delete <context>
```

### List your contexts

Portworx stores your contexts in the `~/.pxctl/contextconfig` directory. Use the `pxctl context list` command to view them:

```text
pxctl context list
```

```output
contextconfig:
  current: user
  configurations:
  - context: user
    token: XXXX
    identity:
      subject: user@example.com/username
      name: Example User
      email: user@example.com
    endpoint: http://localhost:9001
```

### Select the current context

Once you've created your contexts, use the `pxctl context` command to switch between them. Under the hood, Portworx reads your current context and then uses the associated token for all commands.

{{<info>}}
**Note:**
Alternatively, you can use the global `--context` flag to run a single command with a given context.
{{</info>}}

Use the following command to set the current context:

```text
pxctl context set <context>
```

Unset the current context with:

```text
pxctl context unset
```

## How to debug token issues

This section explains how to debug common token issues.

### Permission denied issues

**Problem symptom**: You see an unexpected `Permission denied` or other auth-related error.

**Find the root cause**: Take a look into your token permissions. Decode and verify your token with a JWT token decoding tool such as [jwt.io](https://jwt.io/)

{{<info>}}
**Note:**
The [jwt.io](https://jwt.io/) debugger does client-side validation and debugging. It does not store your token anywhere.
{{</info>}}

### Protocol error

**Problem symptom**: you see an error message similar to `rpc error: code = Internal desc = stream terminated by RST_STREAM with error code: PROTOCOL_ERROR`.

**Find the root cause**: Make sure your token doesn't contain a newline character. The `gRPC/http2` protocol doesn't allow newline characters.
that your token does not accidentally contain a newline character. This is due to gRPC/http2 not allowing newline characters.

## Related topics

* For information about enabling and managing Portworx authorization through Kubernetes, refer to the [Authorization](/operations/operate-kubernetes/authorization/) page.

