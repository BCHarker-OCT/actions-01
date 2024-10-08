# Chapter 1.1 GitHub Actions Learning Continued 

## Matrix Strategies (Example #11)

Use a variable to allow running a job on multiple oses 

```yaml
jobs:
  example_matrix:
    strategy:
      matrix:
        version: [10, 12, 14]
        os: [ubuntu-latest, windows-latest]
```

The keys under Matrix are user created. Refer to them with `${{ matrix.version }}`.

- `fail-fast`: Default behavior is any step in the matrix fails, then all other runs in the matrix fail.Change this by setting `fail-fast` to false in the strategy. 
- `max-parallel`: Sets the maximum jobs to process at a time

### Include/Exclude for Part of the Matrix (Example #11)

```yaml
    deploy:
        strategy: 
            matrix: 
              os: [ubuntu-latest, ubuntu-20.04, windows-latest]
              images: [hello-world, alpine]
              # Exclude alpine image from windows 
              exclude: 
                - images: alpine 
                  os: windows 
              # includes amd64/alpine image on ubuntu-latest
              include:
                - images: amd64/alpine
                  os: ubuntu-latest                  
```

## [Contexts](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs) 

> Contexts are a way to access information about workflow runs, variables, runner environments, jobs, and steps. Each context is an object that contains properties, which can be strings or other objects. 

## [Expressions](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/evaluate-expressions-in-workflows-and-actions) (Example #13)

> You can use expressions to programmatically set environment variables in workflow files and access contexts. An expression can be any combination of literal values, references to a context, or functions. You can combine literals, context references, and functions using operators. For more information about contexts, see "Accessing contextual information about workflow runs."

## [Workflow Event Filters and Activity Types](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows)

> Workflow triggers are events that cause a workflow to run. For more information about how to use workflow triggers, see "Triggering a workflow."

> Some events have multiple activity types. For these events, you can specify which activity types will trigger a workflow run. For more information about what each activity type means, see "Webhook events and payloads."

## [Skipping workflow runs](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/skipping-workflow-runs)

> You can skip workflow runs triggered by the push and pull_request events by including a command in your commit message.

Commands that can be added to a commit message to skip GH Action:
```
[skip ci]
[ci skip]
[no ci]
[skip actions]
[actions skip]
```

## [Debug Logging](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/troubleshooting-workflows/enabling-debug-logging)

> If the workflow logs do not provide enough detail to diagnose why a workflow, job, or step is not working as expected, you can enable additional debug logging.

How to enable it? Set the secret variable for the repo. Can add as variables or as secrets. Secrets has precedence over variables. 
```
# SET 
ACTIONS_RUNNER_DEBUG to true //runner debug 
ACTIONS_STEP_DEBUG to true //step debug
```

You can also enable the logs when running the action manually with the workflow_dispatch button. 

## [Workflow Logs via RestAPI](https://docs.github.com/en/rest/actions/workflows?apiVersion=2022-11-28) (No example workflow)

Use this to download the logs programmatically. 
We can: 
- List workflows 
- Get a workflow 
- Disable/Enable a workflow
- Create a workflow dispatch 
- Get workflow usage 
- Get workflow logs

```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/OWNER/REPO/actions/workflows
```

## [Workflow Dispatch Inputs](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#onworkflow_dispatch) (Example #15) 

10 Inputs Maximum for a Workflow! 

You can setup environments at the GitHub repo level in the GUI. You can attach variables to the environment. This is a pretty cool way to add additional info to your workflow to lessen variables you need to have the user select. 

[Cheating the System more than 10 inputs](https://medium.com/@hemantbhosale916/github-actions-alternative-to-define-more-than-10-inputs-for-workflow-dispatch-event-d4d327d2680b)

🤔 Unsolved Mystery: How do I access variable and secrets from a repository environment? Nothing I tried so far works. 

## Triggers Events with Webhooks 

Manual Dispatch: 
- `workflow_dispatch` --> Click a button 
- `repository_dispatch` --> Send HTTPs request with payload 

**Repository Dispatch Example**
```
on: 
  repository_dispatch:
    types: [system_result]
```

Automated Dispatch: 
- `workflow_run` --> Reusable workflows, cleaner modular workflows 
- `workflow_call` --> chain reaction workflows, if one workflow runs, then trigger the next one 

You can use if statements to not run if the workflow failed: 

```
jobs:
  on-success:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success'}}
    steps:
      -run: echo "Run deploy steps..."
  on-fail:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'failure'}}
    steps:
      -run: echo "Previous workflow failed"      
```