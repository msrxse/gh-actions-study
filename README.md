# gh-actions-study

## About

EVENTS (push, pull) trigger WORKFLOWS, workflows have one or more STEPS. And the RUNNER is the actual server environment that is going to execute those jobs.

Between steps you are in the compute environment. Across jobs you are in a different compute environment, you need to think about that in terms og passing data or sharing environment variables or state between these different elements.

```
on: // this is an event
  workflow_dispatch: // this event is for manually dispatch stuff - UI will show button where you can trigger it yourself

jobs:  // this workflow has one job and one step
  say-hello-inline-bash:
    runs-on: ubuntu-24.04 // This is the runner
    steps:
      - run: echo "Hello World"
```
