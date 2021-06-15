# DCP Demo

This demo uses Emojivoto (https://github.com/BuoyantIO/emojivoto), but has some additional modifications that were necessary in order to run with a more modern gRPC go library and on MacOS.

The Cluster is set up such that there is an `emojivoto-dev` namespace with the emojivoto application deployed, and an `emojivoto` namespace that has the production version that is managed in Argo Rollouts.

The sum of this work is on https://github.com/aes-t2/sol-eng-emojivoto.

## Telepresence demo

### Prerequisites

Telepresence 2.3.1

Tested on MacOS 11.4 Big Sur
Parenthesis is tested version

- Docker (20.10.6)
- go (1.16.5)
- protobuf (3.17.3)
- protoc-gen-go (1.26.0)
- protoc-gen-go-grpc (1.1.0)
- yarn (1.22.10)

### Setup

Clone this repository and cd into folder:

```sh
git clone https://github.com/aes-t2/sol-eng-emojivoto
cd sol-eng-emojivoto
```

Export Kubeconfig:

```sh
export KUBECONFIG=$(pwd)/cluster-setup/solutions-eng-linkerd.yaml
```

Export beta-app flag for Telepresence 2:

```sh
export SYSTEMA_ENV=staging
```

Build dependencies and generate protobuf code:

```sh
make build
```

### Procedure

#### Show emojivoto-dev page

```sh
open https://emojivoto-dev.linkerd.amb-labs.io
```

#### Show emojivoto-dev service catalog

Set up browser for adding the beta-app credentials
Recommended method: https://www.notion.so/datawire/Staging-Environment-Authentication-d597b317fdcd4b21829a0f0aba24e660

```sh
open https://beta-app.datawire.io
```

Recommended: You can set some favorites (click the 3 dots on the right side of a service card) so that the dashboard isn't as cluttered.  These are managed on a per-user basis so they cannot be pre-configured by another person.

Select the `web-svc` card under the Development column.  This will be the deployment we will be intercepting with Telepresence.

#### Run intercept and start code locally

Start intercept:

```sh
telepresence login && \
telepresence intercept web-svc -n emojivoto-dev --port 8080:80 -u=true
```

Check out new test code:

```sh
git checkout cakuros/web-update
```

Open new terminal window, and start running the webpack-dev-server:

```sh
cd emojivoto-web
yarn webpack-dev-server --port 8083
```

Open new terminal window and start running the web server:

```sh
cd emojivoto-web
make run-tel
```

Open the new intercepted service (e.g. https://laughing-benz-205.preview-beta.edgestack.me):

```sh
open https://laughing-benz-205.preview-beta.edgestack.me
```

This build version has been pushed to `docker.io/caseykurosawa/emojivoto-{service-name}:v13-rc0` as a new image for Rollouts in the next section.

## Argo Rollouts demo

### Prerequisites

(Optional)

- Kubectl argo rollout plugin (v1.0.0)

### Setup

Open ArgoCD dashboard (admin:initializer), click on the ac-web-svc-emojivoto app:

```sh
open https://argocd.linkerd.amb-labs.io
```

In a new terminal window, monitor Argo Rollout status:
```sh
kubectl argo rollouts get rollout -n emojivoto web -w
```

### Procedure

Show the production version of Emojivoto:

```sh
open https://emojivoto.linkerd.amb-labs.io
```

Go back to DCP startpage:

```sh
open https://beta-app.datawire.io
```

Select the web-svc card under the Production column.

#### Click on the Start Rollout button in the top bar

Set the Image Tag to `caseykurosawa/emojivoto-web v13-rc0`, set your rollout duration, and weight increment.  Please keep the number of pods set to 2 at maximum.

*Note*: make sure to only click the `Start Rollout` button once.  Multiple clicks will result in multiple PR's being generated.

The button will automatically take you to the Rollouts tab and show a `Pull Request` button that you can click.  Follow the link and merge the PR.

Go back to the Argo dashboard and click Refresh (for some reason, the rollout only initiates after I refresh)

Back in the Service Catalog, you will be able to see some stats about the Rollout, including Canary percentage and the amount of time remaining in the rollout.

#### Check Grafana

Some grafana dashboards (admin:prom-operator):

Rollouts Dashboard: https://obs.linkerd.amb-labs.io/grafana/d/fDNtJHg7k/argo-rollouts?orgId=1&refresh=5s
ArgoCD Dashboard: https://obs.linkerd.amb-labs.io/grafana/d/5RogbNRnk/argocd?orgId=1
Ambassador Dashboard: https://obs.linkerd.amb-labs.io/grafana/d/JGmAedg7z/ambassador-dashboard?orgId=1&refresh=5s
Linkerd Dashboard: https://linkerd.amb-labs.io/grafana/?orgId=1&refresh=1m

#### Check new rollout

Once the rollout is complete, go back to the emojivoto app to see the new changes:

```sh
open https://emojivoto.linkerd.amb-labs.io
```

#### Revert back

Re-run the same procedure creating a new rollout, but target `caseykurosawa/emojivoto-web v12` as the image version.
