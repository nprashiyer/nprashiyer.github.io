### Triggers

Can be **branches, paths, tags** & can have include/exclude directives

```
trigger:
  branches:
	include:
	- main
	- releases/*
	- feature/*
	exclude:
	- releases/old*
	- feature/*-working
  paths:
	include:
	- docs/*.md

  tags:
    include:
    - v2.*
    exclude:
    - v2.0
```

**Pipeline** - one pipeline completion can trigger another pipeline
Below pipeline has a pipeline resource trigger that configures the **app-ci** pipeline to run automatically every time a run of the **security-lib-ci** pipeline completes.
``````
resources:
  pipelines:
  - pipeline: securitylib
    source: security-lib-ci
    project: FabrikamProject
    trigger: true

steps:
- bash: echo "app-ci runs after security-lib-ci completes"
``````

**Scheduled** - run a cron job
```
schedules:
- cron: '0 0 * * *'
  displayName: Daily midnight build
  branches:
    include:
    - main
```

**Manual Trigger**  - for manually triggering use none
```
trigger: none
```
<br>

### Branch Protection Policies
`master` should be protected. Generally followed
1. Minimum 2 approvals - include one from CODEOWNERS
2. All comments must be addressed
3. Linear history - Squash merge / rebase & merge
4. Build validation - Include a number of PR checks
5. Assocaited with a ticket/WorkItems

Other Best Practise
1. PR Template - always good to be dscriptive in your PRs. 
```
.azuredevops/pull_request_template.md
```

As an infra team, we could have a number of check before merging code to main
1. Checkov/Snyk -  IaC Scanning
2. Linters - JSON/MD/YAML/TF FMT/TFLINT/HADOLINT/HELMLINT
3. PSRule/ ARM TTK / TF VALIDATE
4. SECRET SCANNING - TRUFFLEHOG
5. DOCS - PSDocs/TFDOCS


### Agents
Use `pool` to define self-hosted agent. `demands` is the values in capabilities.
```
pool:
  name: Default
  demands: SpecialSoftware # exists check for SpecialSoftware
```

With GH Actions, we currently do not have a usecase for self-hosted runners. We basically buold docker image with all the necessary tooling for the platform team (AWS CLI, TF, TFLINT, TFDOCS, KUBECTL, KUSTOMIZE, YQ) and publish to GH packgaes. Within the pipeline, we use the `image` attribute, on the GH runners to get the software applications we need.

### Templates
Templates let you define reusable content, logic, and parameters in YAML pipelines. Templates can help you speed up development. For example, you can have a series of the same tasks in a template and then include the template multiple times in different stages of your YAML pipeline.

There are two types of templates: includes and extends.

```
# File: templates/include-npm-steps.yml

steps:
- script: npm install
- script: yarn install
- script: npm run compile


# File: azure-pipelines.yml
jobs:
- job: Linux
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - template: templates/include-npm-steps.yml  # Template reference
- job: Windows
  pool:
    vmImage: 'windows-latest'
  steps:
  - template: templates/include-npm-steps.yml  # Template reference
```

Job Reuse
```
jobs:
- template: templates/jobs.yml  # Template reference
```

Stage Reuse:
```
# File: azure-pipelines.yml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: Install
  jobs: 
  - job: npminstall
    steps:
    - task: Npm@1
      inputs:
        command: 'install'
- template: templates/stages1.yml # Template reference
- template: templates/stages2.yml # Template reference
```

We can re-use templates from other repo using resources section:
```
resources:
  repositories:
  - repository: templates
    name: Contoso/BuildTemplates
    endpoint: myServiceConnection # Azure DevOps service connection
jobs:
- template: common.yml@templates
```

### Others
Pipeline variables are values that can be set and modified during a pipeline run. Unlike pipeline parameters, which are defined at the pipeline level and cannot be changed during a pipeline run, pipeline variables can be set and modified within a pipeline using a Set Variable activity

Directories in ADO:
<br>
/ - Agent.WorkFolder --> This is the working director for agent
<br>
/1 - Agent.BuildDirectory --> This is the working director for agent - Dowloaded Artifacts Are placed here
<br>
/1/s - Build.SourcesDirectory | System.DefaultWorkingDirectory -> Source directory. This is where your source code is store
<br>
/1/a - Build.ArtifactStagingDirectory - The publish build artifacts task creates an artifact of whatever is in this folder.
<br>
/1/b - The output folder for compiled binaries. - Build.BinariesDirectory

### TOOLS
**SCA - Software Composition Analysis** - Open Source vulnerabilities - We use Snyk in our pipelines. It also serves as an additional IaC scanner. Basically, we have a partnership going with Snyk, so they dont charge us. Mend/Whitesource alternative.

**SAST - Static Application Security Testing** - Sonarqube - was used prreviously. Very easy to integrate with ADO/GH Actions. Checkmarx another alternative.

If it were to boil down to a short phrase, SonarQube is used for ensuring code quality, and CheckMarx is used for ensuring the security of a system running that code.

SonarQube looks at several areas, including the code coverage percentage of unit tests of the code, duplication percentages, and also code quality issues found through static analysis of the code.

CheckMarx, on the other hand, just analyzes the flow of the code and the inputs and outputs. It looks for situations where inputs that could have been provided by an end user are used directly to control behavior, and other "attack vectors".

Personally I have also worked with Prisma Cloud - Provides IaC Scanning capabilities - SCA - Opensource from Dockerfile, or packages.json or requirements.txt. 
And runtime security in Kubernetes and WebApps(docker based) by installing Twistlock agent, which monitors activities inside the pod/container. We can also set rules to control what can be run inside the container like spawning shell, or predefined rules for cryptp-mining etc.