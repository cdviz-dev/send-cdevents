# Send CDEvents GitHub Action

A GitHub Action that sends CDEvents using the `cdviz-collector send` subcommand.

## Usage

### Basic Example

```yaml
name: Send CDEvent
on:
  push:
    branches: [main]

jobs:
  send-event:
    runs-on: ubuntu-latest
    steps:
      - name: Send CDEvent
        uses: cdviz-dev/send-cdevents@v1
        with:
          data: |
            {
              "context": {
                "version": "0.4.1",
                "source": "github-actions",
                "type": "dev.cdevents.taskrun.started.0.2.0"
              },
              "subject": {
                "id": "${{ github.run_id }}",
                "type": "taskRun",
                "content": {
                  "taskName": "ci-build"
                }
              }
            }
          url: "https://your-webhook-endpoint.com/cdevents"
```

### Advanced Example with Headers

```yaml
name: Send CDEvent with Authentication
on:
  push:
    branches: [main]

jobs:
  send-event:
    runs-on: ubuntu-latest
    steps:
      - name: Send CDEvent
        uses: cdviz-dev/send-cdevents@v1
        with:
          data: |
            {
              "context": {
                "version": "0.4.1",
                "source": "github-actions",
                "type": "dev.cdevents.service.deployed.0.2.0"
              },
              "subject": {
                "id": "${{ github.repository }}",
                "type": "service",
                "content": {
                  "environment": {
                    "id": "production"
                  }
                }
              }
            }
          url: "https://your-webhook-endpoint.com/cdevents"
          headers: |
            Authorization: Bearer ${{ secrets.API_TOKEN }}
            X-Source: GitHub
```

### Using Configuration Content

```yaml
name: Send CDEvent with Config
on:
  push:
    branches: [main]

jobs:
  send-event:
    runs-on: ubuntu-latest
    steps:
      - name: Send CDEvent with HMAC signature
        uses: cdviz-dev/send-cdevents@v1
        with:
          data: |
            {
              "context": {
                "version": "0.4.1",
                "source": "github-actions",
                "type": "dev.cdevents.testcaserun.started.0.2.0"
              },
              "subject": {
                "id": "${{ github.run_id }}-test",
                "type": "testCaseRun",
                "content": {
                  "testCase": {
                    "name": "Integration Tests",
                    "type": "integration"
                  }
                }
              }
            }
          url: "https://your-webhook-endpoint.com/cdevents"
          config: |
            [sinks.http.headers.x-signature-256]
            type = "signature"
            token = "${{ secrets.WEBHOOK_SECRET }}"
            algorithm = "sha256"
            prefix = "sha256="
```

### Using Environment Variables for Secrets

```yaml
name: Send CDEvent with Environment Variables
on:
  push:
    branches: [main]

jobs:
  send-event:
    runs-on: ubuntu-latest
    steps:
      - name: Send CDEvent with env vars
        uses: cdviz-dev/send-cdevents@v1
        env:
          CDVIZ_COLLECTOR__SINKS__HTTP__HEADERS__X_SIGNATURE_256__TOKEN: ${{ secrets.WEBHOOK_SECRET }}
        with:
          data: '{"type": "dev.cdevents.build.started.0.1.1", "source": "github-action"}'
          url: "https://your-webhook-endpoint.com/cdevents"
          config: |
            [sinks.http.headers.x-signature-256]
            type = "signature"
            algorithm = "sha256"
            prefix = "sha256="
```

### Reading from File

```yaml
name: Send CDEvent from File
on:
  push:
    branches: [main]

jobs:
  send-event:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Send CDEvent
        uses: cdviz-dev/send-cdevents@v1
        with:
          data: "@.github/cdevents/build-started.json"
          url: "https://your-webhook-endpoint.com/cdevents"
```

## Inputs

| Input             | Description                                                                          | Required | Default  |
| ----------------- | ------------------------------------------------------------------------------------ | -------- | -------- |
| `data`            | JSON data to send. Can be direct JSON string, file path (@file.json), or stdin (@-)  | Yes      | -        |
| `url`             | HTTP URL to send the data to. When specified, automatically enables the HTTP sink    | No       | -        |
| `config`          | TOML configuration content for advanced sink settings                                | No       | -        |
| `directory`       | Working directory for relative paths                                                 | No       | -        |
| `headers`         | Additional HTTP headers for the request (one per line, format: "Header-Name: value") | No       | -        |
| `additional-args` | Additional arguments to pass to the cdviz-collector send command                     | No       | -        |
| `version`         | Version/tag of the cdviz-collector container to use                                  | No       | `latest` |

## Environment Variables

The action automatically passes all environment variables starting with `CDVIZ_COLLECTOR__` to the cdviz-collector container. This allows you to override configuration values securely using GitHub secrets without exposing them in config files.

### Environment Variable Naming Convention

Environment variables should follow the pattern: `CDVIZ_COLLECTOR__<SECTION>__<SUBSECTION>__<KEY>`

Examples:

- `CDVIZ_COLLECTOR__SINKS__HTTP__HEADERS__X_API_KEY__VALUE` → `sinks.http.headers.x-api-key.value`
- `CDVIZ_COLLECTOR__SINKS__HTTP__HEADERS__X_SIGNATURE__TOKEN` → `sinks.http.headers.x-signature.token`
- `CDVIZ_COLLECTOR__SINKS__HTTP__DESTINATION` → `sinks.http.destination`

## Data Input Formats

### Direct JSON String

```yaml
with:
  data: |
    {
      "context": {
        "version": "0.4.1",
        "source": "github-actions",
        "type": "dev.cdevents.taskrun.started.0.2.0"
      },
      "subject": {
        "id": "${{ github.run_id }}",
        "type": "taskRun",
        "content": {
          "taskName": "ci-build"
        }
      }
    }
```

> [!TIP]
> The `context.id` and `context.timestamp` fields can be automatically generated by cdviz-collector if missing from your CDEvent. You can provide minimal CDEvents and let the collector handle the required metadata:
>
> ```yaml
> with:
>   data: |
>     {
>       "context": {
>         "version": "0.4.1",
>         "source": "github-actions",
>         "type": "dev.cdevents.taskrun.started.0.2.0"
>       },
>       "subject": {
>         "id": "${{ github.run_id }}",
>         "type": "taskRun",
>         "content": {
>           "taskName": "ci-build"
>         }
>       }
>     }
> ```

### From File

```yaml
with:
  data: "@path/to/event.json"
```

### From Stdin (for piped data)

```yaml
with:
  data: "@-"
```

## Configuration Content Example

You can provide TOML configuration content directly in the action:

```yaml
config: |
  [sinks.http.headers.x-signature-256]
  type = "signature"
  token = "${{ secrets.WEBHOOK_SECRET }}"
  algorithm = "sha256"
  prefix = "sha256="

  [sinks.http.headers.x-custom-header]
  type = "static"
  value = "custom-value"
```

> **⚠️ Security Note**: Always use GitHub secrets (`${{ secrets.SECRET_NAME }}`) for sensitive information like API keys, tokens, and webhook secrets. Never hardcode sensitive values directly in your workflow files. The action creates a temporary config file with restrictive permissions in the workspace and automatically cleans it up after execution, even if the action fails.

## Examples

### Build Event

```yaml
- name: Send Build Started Event
  uses: cdviz-dev/send-cdevents@v1
  with:
    data: |
      {
        "context": {
          "version": "0.3.0",
          "id": "build-${{ github.run_id }}",
          "source": "github-actions",
          "type": "dev.cdevents.build.started.0.1.1",
          "timestamp": "${{ steps.timestamp.outputs.timestamp }}"
        },
        "subject": {
          "id": "${{ github.repository }}",
          "source": "${{ github.server_url }}/${{ github.repository }}"
        }
      }
    url: ${{ secrets.CDEVENTS_WEBHOOK_URL }}
```

### Test Event

```yaml
- name: Send Test Started Event
  uses: cdviz-dev/send-cdevents@v1
  with:
    data: |
      {
        "context": {
          "version": "0.3.0",
          "id": "test-${{ github.run_id }}",
          "source": "github-actions",
          "type": "dev.cdevents.test.started.0.1.0",
          "timestamp": "${{ steps.timestamp.outputs.timestamp }}"
        },
        "subject": {
          "id": "${{ github.repository }}",
          "source": "${{ github.server_url }}/${{ github.repository }}",
          "testCase": {
            "id": "${{ matrix.test-suite }}",
            "name": "${{ matrix.test-suite }}"
          }
        }
      }
    url: ${{ secrets.CDEVENTS_WEBHOOK_URL }}
    headers: |
      Authorization: Bearer ${{ secrets.API_TOKEN }}
      X-GitHub-Run-ID: ${{ github.run_id }}
```

## License

This project is licensed under the same license as the cdviz-collector project.
