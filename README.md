# Spinnaker Demo

This repository shows you a demo of how to integrate with Spinnaker. It demonstrates that Spinnaker is listening for the event and deploys the application to a Kubernetes when Gitploy triggers a new deployment. 

![Demo](./docs/images/demo.gif)

## Step 0: Install Spinnaker

In this demo, we don't explain installing Spinnaker. But Armory provides the command-line tool [Minnaker] to get up and running in a few minutes.

## Step 1: Configure Artifacts

This demo build manifests with Helm chart, and it is called artifact, remote and deployable resources, in Spinnaker. And the Helm chart will be overridden for environments, respectively, by values files located under the `release` directory of this repository.

[Configure Helm artifact](https://spinnaker.io/docs/setup/other_config/artifacts/helm/) with the `hal` command:

```shell
# Move into the halyard-0 pod, first.
kubectl exec -it halyard-0 -- sh
```

```shell
hal config artifact helm enable
hal config artifact helm account add \
    --no-validate \
    --repository 'https://gitploy-io.github.io/helm-chart/' \
    helm-demo
```

[Configure Helm artifact](https://spinnaker.io/docs/setup/other_config/artifacts/github/) with the `hal` command:

```shell
hal config artifact github enable
hal config artifact github add \
    --token YOUR_TOKEN \
    github-demo

```

Apply changes:

```shell
hal deploy apply
```

## Step 2: Add custom webhook

As deploying to Kubernetes, Spinnaker has to update the deployment status by the [deployment API call](https://docs.github.com/en/rest/reference/deployments#create-a-deployment-status). Spinnaker provides a simple way to add [a custom stage](https://spinnaker.io/docs/guides/operator/custom-webhook-stages/) instead of extending through codes. And stages typically make quick API calls to an external system as part of a pipeline.

To create a custom webhook, you have to add the [configuration](./spinnaker/.hal/default/profiles/orca-local.yml) for the stage in `orca-local.yml` located in `~/.hal/default/profiles`. *Note that you should modify the `GITHUB_TOKEN` string into your token.

After locating the `orca-local.yml`, deploy it in the `halyard-0` pod(i.e. `hal deploy apply`). Then you can find a new stage in your pipeline.

![Custom Webhook](./docs/images/custom-webhook.png)

## Step 3: Add a pipeline from the pipeline template

Pipeline template is a amazing feature to share with your teams within a single application, across different applications. We'll save the pipeline template by the [spin](https://spinnaker.io/docs/guides/spin/) CLI. If you haven't already done so, you have to install the spin CLI.

To enable pipeline template:

```shell
hal config features edit --pipeline-templates true
hal deploy apply
```

And save the [pipeline template](./spinnaker/pipeline-template/deploy.json):

```shell
spin pipeline-templates save --file spinnaker/pipeline-template/deploy.json
```

Now you can find the pipeline template in the "Pipeline Template" tab, click the "Create pipeline" and configures variables like below:

![Pipeline Template Tab](./docs/images/pipeline-template-tab.png)

![Pipeline Template](./docs/images/pipeline-template.png)

## Step 4: Add a GitHub webhook


