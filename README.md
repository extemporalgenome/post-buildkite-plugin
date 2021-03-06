# Post buildkite plugin

A [Buildkite plugin](https://buildkite.com/docs/agent/v3/plugins) for
running pipeline **serial** steps conditionally when a step has succeed
or failed

The `post` section defines one or more additional steps that are run upon
the completion of a command.

`post` can support any of of the following post-condition blocks: `failure`, `success`.
These condition blocks allow the execution of serial steps inside each condition
depending on the completion status of the step.

## Example

The following pipeline will execute `annotate.sh`, wait for completion, and then `cleanup.sh` only when the command fails

```yml
steps:
  - command: test.sh
    plugins:
      - telefonica/post#0.1.1:
          post:
            - when: failure
              # steps is a string, note the `|`
              steps: |
                - command: email.sh
                - wait
                - command: clenaup.sh
            - when: success
              # steps is a string, note the `|`
              steps: |
                - command: slack.sh
```

## How it works

The plugin evaluates the `command` exit code, (via `post-command` hook) and starts
adding one step after another dinamically using `buildkite-agent pipeline upload`

When adding a pipeline dynamically, it's executed by buildkite
directly after the step that added it.
But when a `wait` is found after a failure in this situation,
the execution will be aborted.
The plugin transform the pipeline and uses this trick to add
steps one after another, making possible to have a serial
pipeline (with `wait`) after failures.

## Current limitations

- Only `wait`, `command` and `plugins` are available as steps in the post section
- Parallel steps in the steps sections are not allowed when using `wait`

This **does not** work

```yml
steps:
  - command: test.sh
    plugins:
      - telefonica/post#0.1.1:
          post:
            - when: failure
              steps: |
                - command: annotate.sh
                - command: email.sh
                - wait
                - command: clenaup.sh
```

But are allowed when not using `wait`

This work

```yml
steps:
  - command: test.sh
    plugins:
      - telefonica/post#0.1.1:
          post:
            - when: failure
              steps: |
                - command: annotate.sh
                - command: email.sh
                - command: clenaup.sh
```

## License

Copyright 2018 [Telefónica](http://www.telefonica.com)

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
