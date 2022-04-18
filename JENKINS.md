# APSIM Jenkins Overview

## Background

Our Jenkins server is currently accessible at https://jenkins.apsim.info/. We migrated from an older server which ran at http://apsimdev.apsim.info:8080/jenkins/. Prior to that we had another jenkins server running somewhere under http://apsrunet.apsim.info.

## Hosting Environment

The jenkins server runs behind a reverse proxy (apache) inside docker on a GCP Linux VM. The persistent configuration is stored in a GCP persistent volume attached to the VM, and which is mounted at /data. This directory is mounted into the jenkins docker container in order to persist config changes between reboots of the container and the VM. The VM is the current "standard" dev.apsim.info box (called apsim-web in GCP). The docker configuration and infrastructure lives in the [apsim-web](https://github.com/hol430/apsim-web) repository. The jenkins server runs behind a reverse proxy (apache). See the web infrastructure document for an overview of how that works.

## Worker Nodes

### Configuration

All of the worker agents are GCP VMs provisioned on an as-needed basis. To view/modify the configuration, go to Jenkins Homepage -> Manage Jenkins -> Manage Nodes and Clouds -> Configure Clouds. Jenkins supports multiple cloud platforms, but we only use GCP.

The "Configure Clouds" page allows us to create VM templates. When Jenkins needs a node to run a job, it will look through the list of templates and attempt to find one that matches the criteria of that job (as defined in the node sections in the jenkinsfiles).

### Resource Limits

GCP imposes a limit to the number of vCPUs we can have running in each "zone" (e.g. australia-southeast1-b) at any given time. I think the default limit is 80, but this limit can be raised by contacting the GCP support team. This is relevant because we use a 96-core VM to run the apsim test suite, and so I have raised our vCPU limit in the australia-southeast1-b zone to 128. Therefore we need to be careful about spinning up to many CPUs in this zone for other work - in fact it be best to avoid this zone entirely, as it doesn't really cost us anything to do so.

If we ever want to run apsim builds in parallel (which would be nice), we would need to raise our vCPU limits in other regions, and copy the existing 96-core template (called apsimci-linux) to other regions.

All of our other build stages run on lightweight VMs (2 vCPUs?) and so instance limits are only really relevant to new apsim test runs.

### Instance Images/Disk

Each of these GCP VM templates defines a boot disk, which is one of our GCP images. These images are sort of like docker images; to create one, you create a VM in GCP, starting from one of the base/public images (such as stock Debian), you then run some commands/install some software, stop the VM, and save its disk as an image. You can then create new VMs from this image. I think GCP does support starting VMs from docker images, but I don't know if the GCP jenkins plugin supports this feature. If it does, it would be nice to change to using docker images rather than the GCP-specific solution.

The available images can be viewed in GCP by going to compute engine -> storage -> images. I have created 2 images which are used by Jenkins:

- apsim-ci-windows (default windows disk with docker and git installed)
- apsimx-ci-docker (default debian disk with docker and git installed)

All of our jobs run a docker container which performs the actual job logic. All of our specific dependencies (e.g. SDKs) are encapsulated by the docker images. Therefore, it's rare that we need to update the GCP images - but it does happen from time to time.

### Other VM configuration notes

The labels are critically important. See the "Jenkinsfile Syntax" section for details, but basically these determine which jobs (or parts of a job) can be run on which VM template.

The Minimum Cpu Platform dropdown is also critical. The value must be matched to the machine type (e.g. n1-highmem-96). Trial and error has been my best method for finding one that works. If you're able to spin up an instance of the template, you'll know it works.

I've used n1-highmem-96 for apsim NG builds - n1-standard-96 runs out of memory (we run all .apsimx files in the repository in a single process).

The windows VM templates need to have a username and password configured on the GCP side when creating the image. These credentials are used by jenkins to connect to the instance.

## Jenkinsfile Syntax

The Jenkins job logic is all stored in version control inside Jenkinsfiles. The Jenkinsfiles define the jobs using jenkins [declarative pipeline syntax](https://www.jenkins.io/doc/book/pipeline/syntax/). See the individual job sections for the path to their jenkinsfiles, as they're not all in the same repo.

It should be fairly easy to deduce what the jenkinsfile is doing just by looking at it, but here's a quick overview of the important features we use.

Each job is divided into stages, which run serially. Typically these will run on different nodes, although in principle there's no reason why they couldn't all run on the same node. Multiple stages can be run in parallel if they are defined inside a `parallel` block.

The `agent` block defines which worker nodes a stage can run on. I've only used the label directive, as it works fine for my purposes. The following indicates that a stage can only run on a VM with the labels "linux", "vm", and "large". If no workers are available with all of these labels, jenkins will attempt to spin up a new node from the cloud templates which satisfies all of these nodes.

```
agent {
    label "linux && vm && large"
}
```

The `environment` block defines environment variables which should be injected into the build environment. Typically these will be credentials or other secrets which can't be obtained from the job configuration in version control.

```
environment {
    GITHUB_PAT = credentials('github-pat')
}
```

The credentials function above will read a variable from Jenkins' global credentials store. If the credential is a secret text, the build environment will contain a variable with the allocated name (in this case GITHUB_PAT). If the credential is a secret file (e.g. a certificate), a temporary file will be created on the worker node, and the path to this file will be stored in the variable name (in this case, GITHUB_PAT). If the credential is a secret login (username/password combo), the build agent will contain two variables - one with a prefix of `_USR` (todo: double check this), and one with a prefix of `_PWD`.

The `steps` directive contains the actual logic performed by this stage of the job. I typically put the actual logic in a shell script and have the `steps` directive just call the shell script.

## Testing Pull Requests

Jenkins will automatically run the test suite for a pull request when a pull request is created, and whenever someone pushes commits to the remote branch tracked by the pull request.

## Re-testing a pull request

Jenkins will rerun the test suite when someone comments on a pull request with the "retest" phrase. This retest phrase is currently "retest this please jenkins" (without the quotes). Any new comment on a pull request containing this text should trigger a rerun in jenkins. Editing an existing comment will /not/ trigger a rerun. The exact trigger phrase can be configured in jenkins, by going to Jenkins homepage -> manage jenkins -> Configure System -> GitHub Pull Request Builder -> Test Phrase. This requires you to be signed in as a user with admin privileges. The settings here apply to all jobs, but can be overriden in a particular job. To check if a job overrides these settings, go to the job's homepage (e.g. https://jenkins.apsim.info/job/apsim/) -> Configure -> Build Triggers -> GitHub Pull Request Builder -> Advanced.

## Creating new Upgrades/Releases of APSIM

Whenever a pull request is merged, a new release of apsim will become available, if and only if the merged pull request fixes/closes an issue.

When the pull request is merged, a webhook is fired off to our REST API. Currently the API endpoint (for both new and old apsim) lives at http://apsimdev.apsim.info/APSIM.Builds.Portal/GitHubPullRequestForm.aspx. The source code for this API endpoint lives in the APSIMInitiative/APSIM.Builds repository, on the apsimdev branch. I've recently been working on migrating this API to .net core and the new Linux server, but haven't fully implemented this endpoint on the new server. For more info, see the APSIM.Builds readme.

This API endpoint will look at the pull request which has been merged and check which issue was referenced by the pull request. Specifically, the pull request must use one of the syntaxes (syntaces?) mentioned in the [github documentation](https://help.github.com/articles/closing-issues-using-keywords/) - e.g. Resolves #xxxx or Working on #xxxx. If a pull request does not reference an issue in this way, it will fail the tests, and thus cannot be merged until it does reference an issue. When checking which issue was referenced by a pull request, a request is made to GitHub's REST API, using a personal access token belonging to the ApsimBot github account. **This token will expire on Tuesday, April 11, 2023.**

If the pull request does not fix an issue (ie "working on #xxxx"), then no release build will be triggered. If the pull request does fix an issue, the API endpoint will then trigger a release build on Jenkins, passing in metadata such as the issue title, pull request number, git commit hash, etc.

See the "Apsim Releases" section for an overview of the release process/cycle for apsim next gen. See the "Old apsim releases" section for an overview of the release cycle for old apsim.

## Apsim Tests

The apsim test suite job tests a pull request to ensure that its changes don't cause any regressions. It consists of 2 stages run in parallel: the unit tests and the validation tests. Both stages are run inside the apsiminitiative/apsimng-build:latest docker container. The dockerfiles can be found [here](https://github.com/APSIMInitiative/APSIM.Docker), and are hosted on [dockerhub](https://hub.docker.com/r/apsiminitiative/apsimng-build).

The unit tests take a few minutes to run on a 4 core Linux VM. They require a github personal access token in order to verify that the pull request actually references an issue. If it doesn't, the build will immediately fail.

The validation tests run all .apsimx files in the repository. If any simulation fails, the entire build is failed. If no simulations fail, the performance tests collector will run (more on this below). If this succeeds, then the autodocs are built and uploaded to the APSIM.Builds API (POST /api/nextgen/upload/docs).



## Apsim Releases

Releasing new apsim builds is handled by a job called apsim-release.

## Old apsim tests

Testing of old apsim pull requests is handled by a job called oldapsim.

## Old apsim releases

Releasing new old apsim builds (new apsim classic builds) is handled by a job called oldapsim-release.

## Comms with GitHub

Jenkins gets notified of github events by a github webhook. This is configured in the repository settings, in the "webhooks" section. In theory, the webhooks are automatically managed by Jenkins, but in practice it's often easier to set them up manually. Automatic webhook configuration (which is enabled on our Jenkins server) requires a personal access token from a github account. In our case, Jenkins is using a token from the ApsimBot github account. This token has no expiry date.

Whenever an event occurs on a pull request, github will call the webhooks as configured here. The webhook which triggers PR test runs is https://jenkins.apsim.info/ghprbhook/. We have webhooks for testing pull requests (for new and old apsim) and for deploying new release builds (for new and old apsim).
