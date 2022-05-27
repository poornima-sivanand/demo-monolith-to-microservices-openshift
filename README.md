# demo-microservices-monolith

This app demonstrates how a nodejs monolith can be converted to microservices.

# Monolith App

## Monolith Architecture


![](images/monolith1.jpg)


## Running the monolith locally

Step 1: Clone this repository
Step 2: Navigate to the monolith folder
Steo 3: Run the following commands to start the application

``` 
    npm install
    npm start
```

Step 4: To test, run the following commands:
```
$ curl localhost:3000/api/users
$ curl localhost:3000/api/users/1
$ curl localhost:3000/api/threads
$ curl localhost:3000/api/threads/1
$ curl localhost:3000/api/posts/by-user/1
```

## Run the monolith on docker

Steo 1: Install and start docker
Steo 2: Build the docker image
```
   docker build . -t demo-app:1.0
```
Step 3: Run the docker image
```
   docker run -p 3000:3000 demo-app:1.0
```
Step 4: Test using the following commands:
```
$ curl localhost:3000/api/users
$ curl localhost:3000/api/users/1
$ curl localhost:3000/api/threads
$ curl localhost:3000/api/threads/1
$ curl localhost:3000/api/posts/by-user/1
```


## Run the monolith on RedHat OpenShift

Step 1: In this demo app, the [code ready containers](https://console.redhat.com/openshift/create/local) provided by OpenShift have been used. Install code ready containers for your desktop in case you dont have access to the OpenShift Container Platform.

Step 2: Login as the kubernetes admin. This role is temporary in crc. Therefore it is necessary to provide admin privileges to the developer.

```
oc login -u kubeadmin https://api.crc.testing:6443
oc adm policy add-role-to-user admin developer
```

Step 3: Login as developer and create the following namespaces

```
oc login -u developer https://api.crc.testing:6443
oc new-project demo-tools
oc new-project demo-dev
```

Step 4: Once logged in to the openshift namespace/crc namespace, execute the following commands:
Note: 'replace the route host in line 60 of monolith/openshift/deploy.yaml with the appropriate value'
Note: 'replace the namespace value in line 151 of monolith/openshift/deploy.yaml with the appropriate value'
Note: 'replace the route host in line 60 of monolith/openshift/deploy.yaml with the appropriate value'
Note: 'replace the GIT BRANCH and GIT URL values in line 99 and 102 of monolith/openshift/build.yaml with the appropriate value'


Note: If using the RedHat OpenShift Platform which enforces zero trust network, consider running the following network policy configuration (This needs to be run only once):

```
    oc -n <namespace> process -f operations/knp.yaml | oc -n <namespace> apply -f -

```

```
   # Import the Image 
   oc -n <tools-namespace> import-image ubi9/nodejs-16:1-36 --from=registry.access.redhat.com/ubi9/nodejs-16:1-36 --confirm

   # Build the Monolith App Image
   oc -n <tools-namespace> process -f monolith/openshift/build.yaml | oc -n <tools-namespace> apply -f -

   # Tag the Image in the Dev Namespace
   oc tag <tools-namespace>/demo-monolith:1.0 <dev-namespace>/demo-monolith:1.0

   # Deploy the Image in the Dev Namespace
   oc -n <dev-namespace> process -f monolith/openshift/deploy.yaml| oc -n <dev-namespace> apply -f -

```

Step 5: Test using the following commands:

```
  curl http://<route-host>/api/users/
  curl http://<route-host>/api/users/1
  curl http://<route-host>/api/threads
  curl http://<route-host>/api/threads/1
  curl http://<route-host>/api/posts/by-user/1
```

# Microservices App

## Microservices Architecture

![](images/microservices1.jpg)


## Running the microservices locally

Step 1: Clone this repository
Step 2: Navigate to the microservices folder
Steo 3: Run the following commands to start each microservice

``` 
    cd posts
    npm install
    PORT=3001 npm start
```

``` 
    cd threads
    npm install
    PORT=3002 npm start
```

``` 
    cd users
    npm install
    PORT=3003 npm start
```

Step 4: To test, run the following commands:
```
$ curl localhost:3003/api/users
$ curl localhost:3003/api/users/1
$ curl localhost:3002/api/threads
$ curl localhost:3002/api/threads/1
$ curl localhost:3001/api/posts/by-user/1
```

## Running the microservices on docker

Step 1: Clone this repository
Step 2: Navigate to the microservices folder
Steo 3: Run the following commands to start each microservice

``` 
    cd posts
    docker build . -t post:1.0
    docker run -p 
```

``` 
    cd threads
    npm install
    PORT=3002 npm start
```

``` 
    cd users
    npm install
    PORT=3003 npm start
```

Step 4: To test, run the following commands:
```
$ curl localhost:3003/api/users
$ curl localhost:3003/api/users/1
$ curl localhost:3002/api/threads
$ curl localhost:3002/api/threads/1
$ curl localhost:3001/api/posts/by-user/1
```

## CI-CD

A sample Github actions workflow is provided as part of this repo under .github folder which can be used as reference for CI/CD, however github actions is currently not configured for this repository and pull requests will not trigger a CI or CD flow. The referenced files are in adherance to the Git flow and not the most optimized method to build and deploy on OpenShift. Methods that encorporate Single command line build and deployment will be incorporated in future versions of this repository.

Git flow is the most commonly used pattern; therefore, it makes sense to have 3 GitHub workflows:
Workflow 1 – CI : It is recommended to tie a release branch to a big user story or epic and individual feature branches for each developer working on the release. All the feature branches are merged to the release branch triggering the CI workflow which deploys changes only to the DEV namespace
Workflow 2 – CI/CD : This workflow runs only when Pull Requests are created to the main branch from the release branch and involve deploying the accumulated changes to dev, test and prod.
Workflow 3 – Clean : It is highly recommended to have a clean workflow to ensure that residual resources in the build and dev namespaces (and test) are cleaned up to optimize resource usage. This workflow is triggered when PRs are merged, closed or deleted.

### What do I need to set up GitHub Actions for OpenShift?

- A service account in OpenShift (this is needed because login tokens for regular users expire in 12 to 24 hrs)

- Secrets for actions within Github settings, one for the openshift url and the other for the service account token (generated by openshift). 

- Environments in github settings for setting reviewers before deployment to dev, test and prod environments.


#### Reference : https://github.com/jrdalino/aws-ecs-monolith-to-microservices-workshop