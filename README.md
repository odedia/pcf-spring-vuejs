Todo on PCF
-----------

This is a Todo demo application that uses the following components:
- VueJS application for the frontend.
- Spring Boot with MySQL and RabbitMQ for the backend.
- .NET Framework "Analytics" app receiving events from RabbitMQ, runs in a Windows Container.
- Runs on Pivotal Cloud Foundry.
- Includes separate concourse pipelines for the frontend and the backend.

The application shows the following features of PCF:
1. The VueJS application is packaged as a docker image to illustrate the ability of PCF to run cloud-native docker images.
2. The Spring Boot application uses the Java buildpack. There is no need to create a Dockerfile for the backend.
3. The Spring Boot application binds itself to a MySQL Service provided from the PCF marketplace. 
4. Both frontend and backend share the same base domain URL. PCF handles the routing tier for us. As a result - there is no need to deal with port bindings or to add CORS support to our backend. VueJS maps to the `/` path and Spring Boot maps to the `/api` path.

# Running

You can push the code either via the commandline or through Concourse.

Initial setup
-------------

Create a PCF mysql service called `mysql`. The name of the mysql service varies by PCF foundation, so check your plans via the `cf marketplace` command. Here's an example:

`cf create-service p.mysql db-small mysql`

Edit `manifest.yml` in the `VuewJSClient` folder.
- Replace the route with the route you plan to use.
- Replace `odedia` with your docker username.

Edit `manifest.yml` in the `SpringBootVueApplication` folder and replace the route with the route you plan to use in your PCF foundation. Make sure you keep the `/api` in the end.

Via command line
----------------

Inside the VueJSClient folder, run a docker build and push to dockerhub:

```
docker build -t <your-dockerhub-username>/todos-ui .
docker push <your-dockerhub-username>/todos-ui
```

Push the frontend app from the `VuewJSClient` folder:

```
cf push
```


Push the backend app from the `SpringBootVueApplication` folder:
`cf push`


Via concourse pipeline
----------------------

Login to your concourse server:

`fly -t concourse login -c <url>`

Create a `params.yml` in your home directory (**never put this file inside your git folder! Credentials may leak!**). 
Add the following parameters:
```
---
cf-api: <redacted>
cf-user: <redacted>
cf-password: <redacted>
cf-org: <redacted>
cf-space: <redacted>
docker-hub-email: <redacted>
docker-hub-username: <redacted>
docker-hub-password: <redacted>
```

For the frontend, run:
`fly -t concourse set-pipeline -p todos-ui -c VueJSClient/ci/pipeline.yml -l ~/params.yml`

For the backend, run:
`fly -t concourse set-pipeline -p todos-backend -c SpringBootVueApplication/ci/pipeline.yml -l ~/params.yml`

Credits:
Based on work published by Andrew Hughes at the Okta developer blog: https://developer.okta.com/blog/2018/11/20/build-crud-spring-and-vue
