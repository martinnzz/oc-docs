## Running cluster tests on image

Use the provided build environment.

```bash
 ./vistaCloudEnv.sh
```

Bring in the dependencies

```bash
 yarn
```

Build (Turn TS into JavaScript) the example service and copy to dist (this is what the image is to be built from)

```bash
 yarn run build
```

Make the Docker image for the example-service. This image is what will be loaded later into the cluster as the
workload under test.

```bash
 yarn run image:build
```

(Re)start the cluster to be used for the cluster tests. (Stopping first if already running). This KinD cluster
runs locally on the developer's PC.

```bash
 yarn run cluster:restart
```

Populate the cluster with all the images and manifests needed. Creates them in the specified namespace
and wait for all the workloads to be running. This takes time so is a one off step before tests
run. It doesn't need to be repeated. Tests should be clearing out any data in these deployments between tests

Wiremock images are deployed with the workload undertest to mock the dependent services.

MongoDB or any other dependency images (needed for testing) are also deployed as needed.

The namespace 'test' can be anything. It allows more than one environment to run in the cluster if needed.
Use the same namespace when running the tests and the sync command below.

```bash
 yarn run cluster:populate -- test
```

## Live sync code to cluster

This will use a tool called devspace (in the dev env) to highjack the example service deployment in
the cluster and replace the image with a bare image that runs the code. The command
that is run (in this case tsx) is in watch mode so reloads the code or test changes. When app starts it writes
a pid file. Changes to this file re-trigger any running 'yarn run test:watch' command.

Do this step in a separate terminal to the tests - still run vistaCloudEnv.sh first.

Logs from the app can be seen in this terminal

```bash
 yarn run sync -- test
```

Leave the code running in the other terminal, and run these test commands in another terminal.

Run the cluster tests. These tests will use the image of example service build previously.

```bash
 yarn run test -- test
```

## Running cluster tests in watch mode

When code or tests change the tests are re-run automatically.

```bash
 yarn run test:watch -- test
```

## Remove the deployments from the cluster for a namespace

```bash
 yarn run cluster:depopulate -- test
```

## Trouble shooting

### A goto means of getting into a good state

First stop kind

```bash
 yarn run cluster:stop
```

Remove all the images that have are part of the cluster from docker

Find all the images

```bash
 docker images
```

Remove each of the images used in the cluster

```bash
 docker image rm <image id>
```

```bash
 yarn run cluster:restart
```

### "ket-example-service" already exists

```
 Error: launch failed: failed to create job: failed to create job: jobs.batch "ket-example-service" already exists
```

Happens if populate or test commands (they still use ket) stop prematurely.

Fix:
Delete the job from the cluster

```bash
kubectl delete job ket-example-service -n test
```
