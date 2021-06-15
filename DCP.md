# DCP Demo

This demo uses Emojivoto (https://github.com/BuoyantIO/emojivoto), but has some additional modifications that were necessary in order to run with a more modern gRPC go library and on MacOS.

The Cluster is set up such that there is an `emojivoto-dev` namespace with the emojivoto application deployed, and an `emojivoto` namespace that has the production version that is managed in Argo Rollouts.

The sum of this work is on https://github.com/aes-t2/sol-eng-emojivoto.

## Setup Telepresence

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

### Show emojivoto-dev page

```sh
open https://emojivoto-dev.linkerd.amb-labs.io
```

### Show emojivoto-dev service catalog

Set up browser for adding the beta-app credentials
Recommended method: https://www.notion.so/datawire/Staging-Environment-Authentication-d597b317fdcd4b21829a0f0aba24e660

```sh
open https://beta-app.datawire.io
```

Recommended: You can set some favorites (click the 3 dots on the right side of a service card) so that the dashboard isn't as cluttered.  These are managed on a per-user basis so they cannot be pre-configured by another person.

Select the `web-svc` card under the Development column.  This will be the deployment we will be intercepting with Telepresence.

### Run intercept and start code locally

Start intercept:

```sh
telepresence login && \
telepresence intercept web-svc -n emojivoto-dev --port 8080:80 -u=true
```

