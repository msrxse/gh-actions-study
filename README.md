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

## Triggering events

- Trigger manually
- Push to branch
- create/update pull request
- Cron schedule
- many more...

## Environment variables

Env variables can be scoped to within the whole workflow, job or step. See .github/workflows/03-core-features--05-environment-variables.yaml

- Passing data is achieved with "outputs"
  By default, jobs run on insulated environments, so environment variables and outputs don't automatically persist in between jobs.

1. Use Step output ($GITHUB_OUTPUT) - this is workflow scoped
2. Job-scoped environment variable ($GITHUB_ENV)

See .github/workflows/03-core-features--06-passing-data.yaml

## Secrets and variables

- You can store these at the organization, repository or environment (staging vs prod) levels.
- See github >> Settings >> secrets and variables >> Actions

## About context

Context is the nameSpace that contains information you can access within your workflow, things like the current branch,commit message, event payload, job outputs, secrets, etc.

- When you see ${{ ... }}, that’s an expression using a context. Eg, ${{ github.repository }} → user/repo
  (also env,job,steps,needs,secretes,number,strategy,matrix,input)
