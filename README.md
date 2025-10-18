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

# 5. Marketplace actions

- The github actions contains reusable building blocks that can help your automation workflows. (http://www.github.com/marketplace?type=actions)
- Most actions are small applications tha you invoke from a uses: step inside your workflow.
- Pin specific commits for security. Tags can be re-targeted after publication, but commit hashes are immutable.

## Popular actions:

Official Github actions:

- actions/checkout: clone your repository into the runner.
- actions/cache: persist dependencies or build outputs between runs.
- actions/upload-artifact and actions/download-artifact: move build artifacts across jobs.
- actions/github-script: run short JS snippets authenticated against the github API.

Runtime and dependency installers

- actions/setup-node

Authentication helpers

- aws-actions/configure-aws-credentials

Additional utilities

- github/super-linter: run language-agnostic linting in one step.
- docker/build-push-action: build and publish multi-architecture container images.

# 6. Avoiding duplication

Github gives us several ways to reuse custom automation between jobs, workflows and repositories.

## composite actions

Composite actions allow you to collect a series of **workflow job steps** into a single **action** which you can then run as a single job step in multiple workflows.
[See docs: reuse workflow steps](https://docs.github.com/en/actions/tutorials/create-actions/create-a-composite-action)

## reusable workflows

Avoid duplication when creating a workflow by reusing existing workflows.
[See docs: reuse workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows)

## JavaScript/TypeScript action

Rich logic with tight GitHub integration. Runs as a step with Node.js runtime.
You can import @actions/core to read inputs, emit logs, set outputs, or fail the run.
"npm run bundle"

## container actions

Container actions let you pick any runtime or dependency stack. GitHub can build the image from a local Dockerfile, or you can point to a registry-hosted image to avoid rebuilding on every run.

# 7. Common workflows

## Validate

### Lint

1. **Checkout the repository code**
2. **Install linter**
3. **Run linter**

Improvements: See [super-lint](https://github.com/marketplace/actions/super-linter)

1. **Checkout the repository code**
2. **Install linter**
3. Determine which files you have changed or created
4. **Run linter on changed files**
5. Report status (summary, pull request comment or status checks)
6. Cache the linting tool to avoid paying the setup cost on every run

### Test

1. **Checkout the repository code**
2. **Install the language runtime**
3. **Download dependencies**
4. **Build application**
5. **Runs the test suite**

Improvements: To setup toolchain, see [setup-node](https://github.com/actions/setup-node) (only 2,3,4 and 8)

1. **Checkout the repository code**
2. **Install the language runtime**
3. Setup caching
4. Restore from cache
5. **Download** (uncached) **dependencies**
6. **Build application**
7. **Runs the test suite**
8. Store test results in cache
9. Upload test results as artifact
10. Report status

### Static analysis

- Runs queries to find common code vulnerabilities

1. **Checkout the repository code**
2. **Install analysis tool**
3. **Run analysis tool**
4. Upload test results to Github and can be displayed in security tab

## Build

Build workflows extend the testing pipeline by producing deployable outputs.

## Deploy

Most deployment workflows start where the build leaves off—using the packaged artifact and promoting it into an environment.

## Repo automations

- [See release-please](https://github.com/marketplace/actions/release-please-action)

- Release-please automates CHANGELOG generation, the creation of GitHub releases, and version bumps for your projects. Release Please does so by parsing your git history, looking for [Conventional Commit](https://www.conventionalcommits.org/en/v1.0.0/) messages (see \* below), and creating release PRs.

- What's a Release PR?
  - Rather than continuously releasing what's landed to your default branch, release-please maintains Release PRs. These Release PRs are kept up-to-date as additional work is merged. When you're ready to tag a release, simply merge the release PR.

\* How should I write my commits? Release Please assumes you are using Conventional Commit messages.The most important prefixes:fix, feat, feat!:, or fix!:, refactor!: (breaking changes indicated by the !)

## Notes

- Under the actions tab in github theres a series of suggested workflows you can start with.

# 8. Developer experience

## For local actions

- A local action is simply one action stored inside your own repo under .github/actions/your-action-name/. This means it’s not published — it’s just used by workflows in the same repository.

- [@github/local-action](https://github.com/github/local-action) Runs custom GitHub Actions locally and test them in Visual Studio Code. Now you don't need to push dummy changes and wait for runners to test/debug your actions. You can also run tests on them.

## On Workflows

-Fist technique:Pull logic out of workflow files into a separate location withing my repo. This way rather than needing to push a change and execute it, i can iterate much more quiclly on my local system. For example inline bash scripts within your YAML, instead you can exatch to, for example, a task file. Now from the workflow you can call this task

1. **Checkout**
2. **Install task because is not available by default**
3. **Execute that task**

- 2nd technique: Test the workflow as a whole. You can run Workflows locally with [act](https://github.com/nektos/act). Rather than having to commit/push every time you want to test out the changes you are making to your .github/workflows/ files, you can use act to run the actions locally.

## Debugging and Observability

- Turn up the logging: GitHub exposes debug channels that are hidden by default. Toggling **ACTIONS_STEP_DEBUG** and **ACTIONS_RUNNER_DEBUG** to true at the repository level surfaces additional logs from both your steps and the runner itself.
- To add these variables:
  `Settings >> Secret and variables >> and add new variable ACTIONS_STEP_DEBUG=true` Could also be a secret - now you will get additional debug logs on the steps output console.
- The **ACTIONS_RUNNER_DEBUG** will create additional runner diagnostic logs you can download. Tons of info from the runner itself.

- [otel-cicd-action](https://github.com/corentinmusard/otel-cicd-action) + Honeycomb. This action exports Github CI/CD workflows to any endpoint compatible with OpenTelemetry, for example Honeycomb. In Honeycomb you can see how long each workflow/step lasts and where exactly time was expend.

# 9. Best Practices

## Measure performance first

- Measure the performance before trying to optimize anything. It doesn't make any sense to try to optimize something if there's no way for you to tell if that optimization actually improved he state of things.
- A good way to measure this is to export that data to some 3rd party platform that you ca use to analyze the data and see what is taking the most time which will inform your decision about where to invest your efforts.

## Wait less

- **Reduce runner queueing**: Ensure you have sufficient capacity (hosted or managed) so jobs start quickly.
- **Fail fast**: Order the most failure-prone tests first so developers get feedback before a long job fails near the end.

## Try doing less

- **Conditional filters**: Don't execute things unless they are actually necessary. You can use conditional filters to inly run workflows, jobs or steps when the relevant code changes withing the repo.
- **Caching**: And you can use caching to avoid repeating work that was done in a previous execution.

## Improve resource utilization

- **Parallelize**: This could mean splitting work across jobs so they can run in parallel, splitting a job so it can run across the multiple cores that your runner may have.
- ** Avoid emulation** where possible: QEMU is an emulation technology that allows you to build apps for a diff architecture that you are running on. Building for the same architecture will always be more efficient.

## Prevent slowdowns

- Once you start improving performance, keep tracking the metrics. Feature work will inevitably add more tests and build steps; monitor timings so you can respond before slowdowns become a bottleneck.

## Caching

Caching is one of the most effective tools for speeding up runs. Depending on your stack, consider layering several of these techniques:

- **Git checkout cache**: Store a recent copy of the repository so actions/checkout only needs to fetch a small delta each run.
- **Language toolchains**: If you install a custom version of Node, Go, Java, etc., cache the toolchain to skip repeated downloads.
- **Dependencies**: Cache package manager directories (npm, pip, cargo, etc.) so you are not pulling dependencies over the public internet every time.
- **Build and test artifacts**: Persist compiled assets or expensive test fixtures when their size makes sense.
- **Container layers**: Keep base images hot, reuse unchanged layers, and use cache mounts to expose dependency caches inside Docker builds.

## Maintainability

### Mono vs multi-repo considerations

- Both repository strategies can support high-quality automation, but the trade-offs differ. Use these to decide what to optimize in your environment.
- Mono-repo: Best for workflow reuse, discoverability and artifacts/hand-offs
- Multi-repo: Best for selective execution, caching, permissions and secretes,auditing and ownership.

### Use a single task runner or build tool

- Choose a single entry point for each service (for example Task, make, or Bazel)

### Define a standard CI API.

- Expose a predictable set of commands such as install, test, build, and dev. When every repository follows the same contract, your GitHub Actions can call those targets without knowing internal details, and developers onboard more quickly.

### Avoid duplication by using Composite Actions and/or reusable Workflows

- Reuse composite actions and reusable workflows instead of copying YAML between projects. This keeps behavior consistent and makes it obvious where to update a shared behavior.

### Optimize local feedback loops

- Pair your CI configuration with local tooling (as shown in module 8) so developers can run the exact same commands before pushing. This shortens the fix/verify loop and reduces noisy workflow runs.

## Security

- Grant the minimum permissions necessary for each workflow or job.
- Favor short-lived tokens over long-lived credentials whenever possible.
- Maintain an allow list of approved marketplace actions.
- Pin marketplace actions by commit SHA to guarantee repeatable builds.
- Do not let forked pull requests run on self-hosted runners; otherwise, attackers could exfiltrate secrets or tamper with the host.
- Use GitHub environments to require manual approvals before sensitive deployments (such as production) execute.
