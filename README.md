# Hello World Application Demo for Red Hat OpenShift Container Platform (RHOCP)
Hello World Applicaiton Demo for Red Hat OpenShift Container Platform (RHOCP) using S2I, [CI/CD](https://cloud.redhat.com/learn/topics/ci-cd) and ([Red Hat OpenShift Pipelines](https://docs.openshift.com/container-platform/latest/cicd/pipelines/understanding-openshift-pipelines.html)).

## Prerequisites

* You have access to an Red OpenShift Container Platform Container Platform Console, Command Line Interface (oc CLI) & Red Hat OpenShift Pipelines (tkn CLI)
	- Tested with Red Hat OpenShift Cluster (RHOCP) 4.12.x
	- It may work with older/newer version having OpenShift Pippelines
	- [Download & Install Red Hat OpenShift CLI](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html)
	- [Download & Install Red Hat OpenShift Pipelines](https://docs.openshift.com/container-platform/latest/cli_reference/tkn_cli/installing-tkn.html)

* You have installed Red Hat OpenShift Pipelines using the Red Hat OpenShift Pipelines Operator listed in the OpenShift OperatorHub. Once installed, it is applicable to the entire cluster.

* You have installed Red Hat OpenShift Pipelines CLI (or use the terminal offered by the `Web Terminal` operator)

* (Optional) You have forked this Git repository using your GitHub ID, and have administrator access to that repository (just for testing the Webhook part)

## Step-by-step guide

### Initial deployment

1. Create the project with the web console ( `+Add` -> `Create New Project`):

    * From command line:

	- Create new poroject `rhocp-cicd-pipelines-demo`

		```bash
		oc new-project rhocp-cicd-pipelines-demo
		```

2. Deploy the initial app:

    * Using web console ( `+Add` -> `From Git` ) with this repo: https://github.com/neoXsys/rhocp-cicd-pipelines-demo

      > ![WARNING](images/warning-icon.png) **WARNING**: `Name` is crucial here. You **must** set it to as **hello-world** as you can see in the sreenshot. If you want to change the `Name` to another value, you should change the `deployment-name` value in for the [TriggerTemplate](cicd/resources/02-triggers/hello-world-trigger.yaml).

    * From command line:

	- Create new `hello-world` application on RHEL UBI 8.0 & PHP S2I

		```bash
		oc new-app php:8.0-ubi8~https://github.com/neoXsys/rhocp-cicd-pipelines-demo --name=hello-world
		```
	
	- Expose route for `hello-world` applicaiton

	
		```bash
		oc expose service/hello-world
		```
	
	- Get external routes for `hello-world` application
	
		```bash
		oc get routes  hello-world
		```
      
3. Create Red Hat OpenShift Pipeline

    * Using web consle ( `+Add` -> `YAML` ) or the Pipeline menu:
	- Copy resource file from `https://raw.githubusercontent.com/neoXsys/rhocp-cicd-pipelines-demo/main/cicd/resources/01-pipelines/hello-world-pipeline.yamL`

    * From command line:
	
        ```bash
        oc create -f https://raw.githubusercontent.com/neoXsys/rhocp-cicd-pipelines-demo/main/cicd/resources/01-pipelines/hello-world-pipeline.yaml
        ```

4. Running the Pipeline

    * Using web consle ( `Pipeline` -> `Start` )
	- REPO: `https://github.com/neoXsys/rhocp-cicd-pipelines-demo`
	- Workspace: volumeClaimTemplateFile

    * From command line:

        ```bash
        tkn pipeline start build-and-deploy-hello-world-app -w name=workspace,volumeClaimTemplateFile=https://raw.githubusercontent.com/neoXsys/rhocp-cicd-pipelines-demo/main/cicd/resources/00-worspaces/hello-world-workspace.yaml -p GIT_REPO=https://github.com/neoXsys/rhocp-cicd-pipelines-demo --use-param-defaults

        ```
    > ![NOTE](images/info-icon.png) **NOTE**: A `PipelineRun` resource starts a pipeline and ties it to the Git and image resources that should be used for the specific invocation. It automatically creates and starts the `TaskRun` resources for each task in the pipeline.

### Triggers and Webhook

Triggers enable pipelines to respond to external GitHub events, such as push events and pull requests. After you assemble and start a pipeline for the application, add the `TriggerBinding`, `TriggerTemplate`, `Trigger`, and `EventListener` resources to capture the GitHub events.

1. Adding triggers to the pipeline

    * Using web console ( `+Add` -> `YAML` )
	- Copy resource file from `https://raw.githubusercontent.com/neoXsys/rhocp-cicd-pipelines-demo/main/cicd/resources/02-triggers/hello-world-trigger.yaml`

    * From command line:

        ```bash
        oc create -f https://raw.githubusercontent.com/neoXsys/rhocp-cicd-pipelines-demo/main/cicd/resources/02-triggers/hello-world-trigger.yaml
        ```

2. Expose the `EventListener` service as an OpenShift Container Platform route to make it publicly accessible:

		```bash
		oc expose service/el-hello-world-app
		```

	- Check route for `EventListener` 

		```bash
		oc get routes el-hello-world-app
		```
 

> ![NOTE](images/info-icon.png) **NOTE**: Adding webhooks requires administrative privileges to the repository. If you do not have administrative access to your repository, contact your system administrator for adding webhooks.

1. Get the webhook URL:

	```bash
	echo "URL: $(oc  get route el-hello-world-app --template='http://{{.spec.host}}')"
 	```

2. Configure webhook manually on your repository:

    * Open the repositoryin your browser.
    * Click **Settings** → **Webhooks** → **Add Webhook**
    * On the Webhooks/Add Webhook page:
        * Enter the webhook URL from step 1 in Payload URL field
        * Select **application/json** for the Content type
        * Ensure that the **Just the push event** is selected
        * Select **Active**
        * Click **Add Webhook**


Reference: https://github.com/josgonza-rh/openshift-s2i-php
# rhocp-cicd-pipelines-demo
