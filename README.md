#  GitHub Actions workflow with Flux and Amazon EKS

A workflow that uses GitHub Actions to build a static website (app/site/) into a Docker container, push that image to Amazon Elastic Container Registry, and uses to automatically update an existing Amazon Elastic Kubernetes Service cluster with that image.


## Prerequisites

1. Create an EKS cluster, e.g. using [`eksctl create cluster`]
2. Set up Flux on the cluster
```bash
export GHOWNER=<github user or organization account where your fork lives>
export GHREPO=example-actions-flux-eks

kubectl create ns flux

fluxctl install \
    --git-user=${GHUSER} \
    --git-email=${GHUSER}@users.noreply.github.com \
    --git-url=git@github.com:${GHOWNER}/${GHREPO} \
    --git-path=manifests \
    --namespace=flux | kubectl apply -f -
```
3. Give Flux read/write access to the GitHub repository 
4. Create a repository called `task-eks` in in the same AWS region as the EKS cluster
5. Update the image in [`deployment.yml`](manifests/deployment.yml) to use your `REGISTRY`, `IMAGE`, and `TAG`. `TAG` will be replaced by Flux as new images are available in the registry.

## Secrets

The following secrets are required to be set on the repository:

1. `AWS_ACCOUNT_ID`: The AWS account ID that owns the EKS cluster
1. `AWS_ACCESS_KEY_ID`: An AWS access key ID for an account having the [EKS IAM role](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html)
1. `AWS_SECRET_ACCESS_KEY`: An AWS secret sccess key for an account having the [EKS IAM role](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html)

## Workflow

The workflow(.github/workflows/build.yml) will trigger on every push to this repo.

For _pull requests_, the workflow will:
1. Build and tag [the Docker image](app/Dockerfile)
    - The image will be tagged with the feature branch's HEAD commit SHA
    
For _pushes_ to the default branch (`master`), in addition to the above, the workflow will:

1. Push the image to Amazon Elastic Container Registry

## Beyond the workflow

Flux watches ECR for changes to the image listed in our [deployment configuration](manifests/deployment.yml). When it detects a change, it updates the EKS cluster with the new image, no manual `kubectl apply` needed!

