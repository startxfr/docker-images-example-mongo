<img align="right" src="https://raw.githubusercontent.com/startxfr/docker-images/master/travis/logo-small.svg?sanitize=true">

# docker-images-example-mongo


Example of a simple Nosql database using the startx s2i builder [startx/sv-mongo](https://hub.docker.com/r/startx/sv-mongo). 
For more information on how to use this image, **[read startx mongo image guideline](https://github.com/startxfr/docker-images/blob/master/Services/mongo/README.md)**.

## Running this example in OKD (aka Openshift)

### Create a sample application

```bash
# Create a openshift project
oc new-project startx-example-mongo
# start a new application (build and run)
oc process -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/mongo/openshift-template-build.yml -p APP_NAME=myapp | oc create -f -
# Watch when resources are available
sleep 30 && oc get all
```

### Create a personalized application

- **Initialize** a project
  ```bash
  export MYAPP=myapp
  oc new-project ${MYAPP}
  ```
- **Add template** to the project service catalog
  ```bash
  oc create -f https://raw.githubusercontent.com/startxfr/docker-images/master/Services/mongo/openshift-template-build.yml -n startx-example-mongo
  ```
- **Generate** your current application definition
  ```bash
  export MYVERSION=0.1
  oc process -n startx-example-mongo -f startx-mongo-build-template \
      -p APP_NAME=v${MYVERSION} \
      -p APP_STAGE=example \
      -p BUILDER_TAG=latest \
      -p SOURCE_GIT=https://github.com/startxfr/docker-images-example-mongo.git \
      -p SOURCE_BRANCH=master \
      -p MONGO_ROOT_PASSWORD=root_pwd \
      -p MONGO_USER=mongouser \
      -p MONGO_PASSWORD=mongouser_pwd \
      -p MONGO_STORES=example \
      -p MEMORY_LIMIT=256Mi \
  > ./${MYAPP}.definitions.yml
  ```
- **Review** your resources definition stored in `./${MYAPP}.definitions.yml`
- **build and run** your application
  ```bash
  oc create -f ./${MYAPP}.definitions.yml -n startx-example-mongo
  sleep 15 && oc get all
  ```
- **Test** your application
  ```bash
  oc describe route -n startx-example-mongo
  curl http://<url-route>
  ```

## Running this example with source-to-image (aka s2i)

### Create a sample application

```bash
# Build the application
s2i build https://github.com/startxfr/docker-images-example-mongo startx/sv-mongo startx-mongo-sample
# Run the application
docker run --rm -d -p 8777:27017 startx-mongo-sample
# Test the sample application
telnet -h localhost 8777
```

### Create a personalized application

- **Initialize** a project directory
  ```bash
  git clone https://github.com/startxfr/docker-images-example-mongo.git mongo-myapp
  cd mongo-myapp
  rm -rf .git
  ```
- **Develop** and create a personalized page
  ```bash
  cat << "EOF"
  {"field":"name"}
  EOF > data.json
  ```
- **Build** your current application with startx mongo builder
  ```bash
  s2i build . startx/sv-mongo:latest startx-mongo-myapp
  ```
- **Run** your application and test it
  ```bash
  docker run --rm -d -p 8777:27017 startx-mongo-myapp
  ```
- **Test** your application
  ```bash
  telnet -h localhost 8777
  ```
