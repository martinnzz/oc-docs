# Demonstration image pipeline

This application repository contains a demonstration image pipeline. The images are intended to be
deployed in a kubernetes cluster. The Jenkinsfile contains the steps in the pipeline.

This example is for a nodejs application. But most of the steps are relevant
to any programming language.

It's interned that this example pipeline can live on as the reference pipeline for the best
practises for building and publishing known and truster images.
The Jenkinsfile contains each step in the pipeline. This is a description of each step in order.

## Static checks

Checks that can run on the static source code such as linting, strict type checking.

## Node tests

Runs the unit tests for the nodejs application.

## Build application

Compiles the application

## Build image

Use docker commands to make an OCI compliant image.

## Populate kind

Run up a k8s local l8s cluster to be able to run the next tests

## Image tests in kind"

The cluster tests stage runs using a real built image.

## Check Image

Check that the image has the proper annotations

## Publish Image

Push the image to an image repository such as dockerhub as a trusted and verified and documented image.
