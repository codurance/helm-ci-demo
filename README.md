# Helm + KinD + GitHub Actions = <3

This is a demo of how KinD (Kubernetes-in-Docker) can be used as part of a GitHub Actions
Workflow for testing Helm charts.

## What's what?

`/hello-kubernetes`
This is a simple Helm 3 Chart for an external application (i.e. not hosted in this repository).
The application is a static web app taken from [https://github.com/paulbouwer/hello-kubernetes](https://github.com/paulbouwer/hello-kubernetes).

The `templates/` folder contains the actual chart templates for a `Deployment` and a `Service`, while the `values.yaml`
in this example contains the only set of values to render the chart with.

This chart can be installed on a cluster by simply running `helm install hello-kubernetes`.


`./github/workflows/kind-test.yaml`
This is the GitHub Actions workflow that performs a sample CI run for this chart.


`ct.yaml`
This is the config overrides for Helm's Chart Testing tools. The most important setting is the base branch for the
repository (see below)

## CI Workflow

After checking out code (`Checkout`), the workflow first un-shallows the repository clone (`Fetch history`) because Helm's
Chart Testing tools require the git history.

Next it uses the Chart Testing tools to lint the chart (`Run chart-testing (lint)`). This provides two things: firstly it
ensures that the Chart is syntactically correct and well-formed. Secondly, it indicates whether there have been any
changes to the chart itself, comparing to the configured base branch (see `ct.yaml`).

To save time, all subsequent steps are only performed if there are any changes to the Chart.

The next step is to spin up a [KinD](https://kind.sigs.k8s.io/docs/user/quick-start/) cluster (`Create kind cluster`).
This will also install and configure `kubectl`.

Once the cluster has come live, Chart Testing deploys the chart to a temporary namespace (`Run chart-testing (install)`). This is to ensure the Chart itself works correctly. The namespace is then deleted. While not strictly necessary, this step provides a convenient shortcut in case the Chart is faulty because it handles `helm` errors internally.

Finally, we deploy the chart again to a known namespace using `helm` directly (`Install chart`) to create an environment
we can perform more thorough testing on.

To access the application, we use `kubectl port-forward` (`Port Forward to Service`).

In this example the only test being performed is a simple `curl` to verify the web application is responding (`Test App`).
In the same manner, an integration or e2e test suite could be run against the same endpoint.

## How to Try it Out

First, clone this repository.

Create a new branch, make a change to the chart (e.g. the message kept in `values.yaml`), bump the version in the
`Chart.yaml` itself and push the branch to Github.

Open a Pull Request. The workflow will now run and compare your chart changes to the `main` branch.
