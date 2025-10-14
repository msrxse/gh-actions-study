# gh-actions-study

These are my own notes to the **GitHub Actions: Beginner to Pro** course from **DevOps Directive**.

Links:

- [GitHub Actions: Beginner to Pro](https://courses.devopsdirective.com/github-actions-beginner-to-pro/lessons/00-introduction/01-main)
- [Youtube video](https://www.youtube.com/watch?v=Xwpi0ITkL3U)

# 3. Core features

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

- When you see

```
${{ ... }}, that’s an expression using a context. Eg, ${{ github.repository }} → user/repo
```

- Contexts are needs, steps, secrets, vars, github, env, matrix, strategy, job, runners, inputs

### Note:

The **uses** keyword:

Eg,

```
// uses tells GitHub to use a prebuilt action.
uses: actions/setup-node@v4 // means “use the checkout action from the official actions/checkout repository.”
  with: // passes input parameters to that action
    node-version: 20
    cache: npm
    cache-dependency-path: ./package-lock.json
```

# 4. Advanced features

## Runner types and execution environments

- Every job must declare where it runs.The runs-on key determiners the operating system, and you can layer on container.

## Artifacts

Actions runners are ephemeral, everything created during a job disappears when it finishes. Artifacts provide an escape hatch by storing files in Github's object storage so they can be downloaded later or consumed by downstream jobs.

Use the generic **actions/upload-artifact@v4**

## Caches

caches store intermediate data to speedup the subsequent runs. Github's hosted cache stores up th 10 GG of data per repo.

Use the generic **actions/cache** action lets you save directories.

Many actions include custom caching. For example, actions/setup-node can automatically cache node_modules based on a lockfile so future installs simply hydrate from the cache

```
steps:
      - uses: actions/checkout@v5.0.0
      - uses: actions/setup-node@v4.4.0
        with:
          node-version: 20
          cache: npm
          # Point cache action at the specific lock-file path
          cache-dependency-path: package-lock.json
```

## Permissions

- Every Github workflow automatically gets an authentication token called GITHUB_TOKEN. You can access it with: ` ${{ secrets.GITHUB_TOKEN }}`

```
jobs:
  read-only-pr:
    runs-on: ubuntu-24.04
    permissions:
      pull-requests: read # By default on contents/packages read, here you override that to pull-requests = read
    steps:
      - uses: actions/checkout@v5.0.0 # Fetches your repository’s code into the runner.
      - name: List the first 5 open PRs (allowed)
        run: gh pr list --limit 5 # uses the GitHub CLI to list open PRs.
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }} # Pass it into env just makes the token available to your command above (gh pr list)


```

## Authenticating to third-party systems

Most workflows need to talk to external platforms. You can authenticate in two primary ways:

- Static credentials (API keys) stored as encrypted secrets. Easy to integrate but riskier if they leak.
- OIDC federation where GitHub mints a short-lived JSON Web Token at runtime that a cloud provider exchanges for temporary credentials. This removes the need to manage secrets and sharply limits blast radius.

- The preferred alternative is to register GitHub's OIDC provider in IAM, create a role with a trust policy that allows your repository to assume it, and then request that role inside the workflow.

## Matrix strategies, conditionals, and concurrency controls

Advanced scheduling features help you orchestrate complex automation without duplicating YAML.

- Matrix jobs: fan out a single job definition across multiple inputs—useful for testing on different operating systems, language versions, or dependency sets.
- Conditionals: (if: expressions) control whether a step or job should run for a particular matrix combination or based on data from previous steps.
- Concurrency groups: instruct GitHub Actions to cancel superseded runs so you aren't burning time on outdated builds.
