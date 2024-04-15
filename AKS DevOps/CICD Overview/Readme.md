# Key concepts for new Azure Pipelines

Azure Pipelines automatically builds and tests code projects. It supports all major languages and project types and combines continuous integration, continuous delivery, and continuous testing to build, test, and deliver your code to any destination.

## Continuous Integration
Continuous Integration (CI) is the practice used by development teams of automating, merging, and testing code. CI helps to catch bugs early in the development cycle, which makes them less expensive to fix. Automated tests execute as part of the CI process to ensure quality. 
- Increase code coverage
- Build faster by splitting test and build runs
- Automatically ensure you don't ship broken code
- Run tests continually

> [!Note]
> CI systems produce artifacts and feed them to release processes to drive frequent deployments.


## Continuous Delivery
Continuous Delivery (CD) is a process by which code is built, tested, and deployed to one or more test and production environments. Deploying and testing in multiple environments increases quality. - CD systems produce deployable artifacts, including infrastructure and apps. 
- Automated release processes consume these artifacts to release new versions and fixes to existing systems.
- Systems that monitor and send alerts run continually to drive visibility into the entire CD process.
- Automatically deploy code to production
- Ensure deployment targets have latest code
- Use tested code from CI process

## Continuous Testing
Whether your app is on-premises or in the cloud, you can automate build-deploy-test workflows and choose the technologies and frameworks. Then, you can test your changes continuously in a fast, scalable, and efficient manner

> [!Note]
> Continuous testing with Azure DevOps Server ensures your app still works after every check-in and build, enabling you to find problems earlier by running tests automatically with each build.

## Pricing for Azure DevOps
If you use public projects, Azure Pipelines is free, but you will need to request the free grant of parallel jobs. 

If you use private projects, you can run up to 1,800 minutes (30 hours) of pipeline jobs for free every month.

> [!Note]
> You can use your custom agent to have unlimited pipeline jobs execution time.



- Pipelines are made of stages, jobs and steps.
- A pipeline can deploy to one or more environments.
- A Pipeline response to a automatic or manual trigger
 - A trigger can be a 
   - Manual trigger 
   - Push to a Repo
   - Scheduled time
   - After another build finished

## Pipeline Hierarchy 
Pipelines are made of stages, jobs and steps.

- Pipeline > Stage - A stage is a way to logically divide your pipeline
  - A stage is a logical boundary in the pipeline.
  - It can be used to mark the separation of concerns (for example, Build, QA, and production).
  - For example, you could have a stage for Build, Test and Deploy
  - Inside each stage is one or more jobs
  - a stage can depend on another stage

- Pipeline > Stage > Job - Each job runs on one agent. A job can also be agentless.
  - A job can attached to a custom agent 
  - A job can also be an agent less for example a 30 seconds delay.
  - A job represents an execution boundary of a set of steps.
  - A job is a collection of steps run by an agent on a server or client custom host machine.

- Pipeline > Stage > Job > Step/Tasks - A step can be a task or script and is the smallest building block of a pipeline.
  - For example: A test can be
    - A Task is a pre-packaged script that performs an action
    - All of the steps run together on the same agent. 
    - Build artifact 
    - Publish artifact 
    - Invoke an API  

A typical pipeline hierarchy 
![image](https://github.com/wasi-shah/Docker-AKS/assets/51749920/34356325-ac1e-4e2f-93cd-c93d6710c39a)

```

- Pipeline
  - Stages
    - Stage : Build
        - Jobs
            - Job 1
                - Steps
                    - Task 1
                    - Task 2

```
> [!important]
> Simple pipelines don't require all of these levels. For example, in a single-job build, you can omit the containers for stages and jobs because there are only steps. And because many options shown in this article aren't required and have good defaults, your YAML definitions are unlikely to include all of them.

If you have a single stage, you can omit the stages keyword and directly specify the jobs keyword:
```
# ... other pipeline-level keywords
jobs: [ job | template ]
```

If you have a single stage and a single job, you can omit the stages and jobs keywords and directly specify the steps keyword:
```
# ... other pipeline-level keywords
steps: [ script | bash | pwsh | powershell | checkout | task | template | ... ]
```
A typical pipeline file with stages
```
stages: [ stage | template ] # Required. Stages are groups of jobs that can run without human intervention.
pool: string | pool # Pool where jobs in this pipeline will run unless otherwise specified.
name: string # Pipeline run number.
appendCommitMessageToRunName: boolean # Append the commit message to the build number. The default is true.
trigger: none | trigger | [ string ] # Continuous integration triggers.
parameters: [ parameter ] # Pipeline template parameters.
pr: none | pr | [ string ] # Pull request triggers.
schedules: [ cron ] # Scheduled triggers.
resources: # Containers and repositories used in the build.
  builds: [ build ] # List of build resources referenced by the pipeline.
  containers: [ container ] # List of container images.
  pipelines: [ pipeline ] # List of pipeline resources.
  repositories: [ repository ] # List of repository resources.
  webhooks: [ webhook ] # List of webhooks.
  packages: [ package ] # List of package resources.
variables: variables | [ variable ] # Variables for this pipeline.
lockBehavior: string # Behavior lock requests from this stage should exhibit in relation to other exclusive lock requests.
``` 
Example pipeline
```
trigger:
- main

pool: 
  vmImage: ubuntu-latest

stages:
- stage: CI
  jobs:
  - job: CIWork
    steps:
    - script: "Do CI work"

- stage: Test
  jobs:
  - job: TestWork
    steps:
    - script: "Do test work"
    
```

## Run
A run represents one execution of a pipeline. It collects the logs associated with running the steps and the results of running tests. During a run, Azure Pipelines will first process the pipeline and then send the run to one or more agents.

## Artifact
An artifact is a collection of files or packages published by a run.

## Agent
When your build or deployment runs, the system begins one or more jobs. An agent is computing infrastructure with installed agent software that runs one job at a time. For example, your job could run on a Microsoft-hosted Ubuntu agent or your own hosted custom agent.

## Environment
An environment is a collection of resources where you deploy your application. One environment can contain one or more virtual machines, containers, web apps, or any service. Pipelines deploy to one or more environments after a build is completed and tests are run.

# Variables 
There are three types of variables flows in the pipeline
1. User-defined Variables
2. System variables
3. Environment variables


## User-defined Variables
Variables give you a convenient way to get key bits of data into various parts of the pipeline. The most common use of variables is to define a value that you can then use in your pipeline. All variables are strings and are mutable. 

> [!Note]
> The value of a variable can change from run to run or job to job of your pipeline.

### Variable scopes
In the YAML file, you can set a variable at various scopes:

- You can also specify variables outside of a YAML pipeline in the UI. When you set a variable in the UI, that variable can be encrypted and set as secret.
- At the root level, to make it available to all jobs in the pipeline.
- At the stage level, to make it available only to a specific stage.
- At the job level, to make it available only to a specific job.

> [!Note]
> Variables at the job level override variables at the root and stage level. Variables at the stage level override variables at the root level.
> When you define the same variable in multiple places with the same name, the most locally scoped variable wins. So, 
> - a variable defined at the job level can override a variable set at the stage level. 
> - A variable defined at the stage level overrides a variable set at the pipeline root level. 
> - A variable set in the pipeline root level overrides a variable set in the Pipeline settings UI. 

```
variables:
  global_variable: value    # this is available to all jobs

jobs:
- job: job1
  pool:
    vmImage: 'ubuntu-latest'
  variables:
    job_variable1: value1    # this is only available in job1
  steps:
  - bash: echo $(global_variable)
  - bash: echo $(job_variable1)
  - bash: echo $JOB_VARIABLE1 # variables are available in the script environment too

- job: job2
  pool:
    vmImage: 'ubuntu-latest'
  variables:
    job_variable2: value2    # this is only available in job2
  steps:
  - bash: echo $(global_variable)
  - bash: echo $(job_variable2)
  - bash: echo $GLOBAL_VARIABLE
  ```

  Output
  ```
# job1
value 
value1
value1

# job2
value
value2
value
  ```

### Variable groups and templates

#### Variable groups
Variable groups are a set of variables that you can use across multiple pipelines. They allow you to manage and organize variables that are common to various stages in one place.

#### Variable templates
With templates, variables can be defined in one YAML and included in another YAML file.
Use templates to define variables in one file that are used in multiple pipelines.


Sample using variables, group and templates
```
variables:
# a regular variable
- name: myvariable
  value: myvalue
# a variable group
- group: myvariablegroup
# a reference to a variable template
- template: build.yml
```
build.yml
```
variables:
- name: vmImage
  value: vs2017-win2016
- name: arch
  value: x64
- name: config
  value: debug
```


## System variables
In addition to user-defined variables, Azure Pipelines has system variables with predefined values. For example, the predefined variable Build.BuildId gives the ID of each build and can be used to identify different pipeline runs. You can use the Build.BuildId variable in scripts or tasks when you need to a unique value.

https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml


If you're using classic release pipelines, see release variables.

System variables get set with their current value when you run the pipeline. Some variables are set automatically. As a pipeline author or end user, you change the value of a system variable before the pipeline runs.

System variables are read-only.

## Environment variables

Environment variables are specific to the operating system you're using. They're injected into a pipeline in platform-specific ways. The format corresponds to how environment variables get formatted for your specific scripting platform.

On UNIX systems (macOS and Linux), environment variables have the format $NAME. On Windows, the format is %NAME% for batch and $env:NAME in PowerShell.

> [!Note]
> System and user-defined variables also get injected as environment variables for your platform.
> When variables convert into environment variables, variable names become uppercase, and periods turn into underscores.
>For example, the variable name any.variable becomes the variable name $ANY_VARIABLE.

## List variables
You can list all of the variables in your pipeline with the az pipelines variable list command.
```
az pipelines variable list [--org]
                           [--pipeline-id]
                           [--pipeline-name]
                           [--project]
```
