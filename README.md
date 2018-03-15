# Dynamic Verticle Deployment with YAML config

The following is a Entrypoint Verticle (and FatJar) written in Javascript/Nashorn for Vertx.io.

It provides the ability to config *sub-verticles* (verticles that are not the entrypoint verticle) through a YAML configuration file.

```yaml
# Verticle Configurations
verticles:
    my_verticle_steve:
        path: "verticles/vert1/vert1.js"
        worker: false
        instances: 1
        isolationGroup:
        isolatedClasses:
        config:
            value1: "dog"
            value2: "cat"
    processing_verticle_ABC:
        path: "some/docker/volume/location/verticleABC.js"
        worker: true
        instances: 10
```

The only required field is the `path`.

## ENV Vars:

On entrypoint verticle startup, two ENV variables are assesed:

`YAML_PATH`: provides the path to where the YAML Config file is located.  If this ENV is not provided, it will fallback to using `default-config.yam` in the `./app` folder

`YAML_PATH_SCAN_PERIOD`: provides the scan period/frequency (milliseconds) for the YAML config file.  If this ENV is not provided, it will fallback to a value of 5000.

## Deployment Change Detection

When a change to the config file is detected, each verticle in the `verticles` object will be compared against the previous configuration.  For each verticle that has changed or if it is a new verticle, it will be deployed.

- [ ] TODO: Add detection for removed verticles in config YAML and undeploy those.


# Docker Setup

In practice, when setting up this project with a docker deployment, set the YAML_PATH env of the container for a Docker Volume location

# Console output

Example of console output for the first load and then a change to the number of instances for the my_verticle_steve verticle

```console
Starting vert.x application...
090eba58-623c-4288-9364-a196d8f866b3
YAML_PATH ENV variable is null or a empty string. Falling back to default-config.yaml
YAML_PATH_SCAN_PERIOD ENV variable is null or is not an integer. Falling back to value: 5000
Yaml Store Config: {"type":"file","format":"yaml","config":{"path":"/Users/MyUser/Documents/GitHub/verticle-deploy-by-config-vertx/build/resources/main/default-config.yaml"},"optional":false}
{"scanPeriod":5000,"stores":[{"type":"file","format":"yaml","config":{"path":"/Users/MyUser/Documents/GitHub/verticle-deploy-by-config-vertx/build/resources/main/default-config.yaml"},"optional":false}]}
Starting primary verticle
Succeeded in deploying verticle
Detected changed in config yaml
prev conf:
{}
new conf:
{"verticles":{"my_verticle_steve":{"name":"my verticle steve","path":"verticles/vert1/vert1.js","worker":false,"instances":1,"isolationGroup":null,"isolatedClasses":null,"config":{"value1":"dog","value2":"cat"}}}}
Prev conf was empty
Deploying: my_verticle_steve from path verticles/vert1/vert1.js
All verticles in config have been reviewed
vert 1 sending a message
Vert 1 confg: {"value2":"cat","value1":"dog"}
Deployment id is: 82d3a66e-a148-40d7-ad90-41ad99d0386c
Deployments current active: [82d3a66e-a148-40d7-ad90-41ad99d0386c, 37913bdd-9aa6-4524-8e1b-6b8a84f474af]
Detected change in config yaml
prev conf:
{"verticles":{"my_verticle_steve":{"name":"my verticle steve","path":"verticles/vert1/vert1.js","worker":false,"instances":1,"isolationGroup":null,"isolatedClasses":null,"config":{"value1":"dog","value2":"cat"}}}}
new conf:
{"verticles":{"my_verticle_steve":{"name":"my verticle steve","path":"verticles/vert1/vert1.js","worker":false,"instances":2,"isolationGroup":null,"isolatedClasses":null,"config":{"value1":"dog","value2":"cat"}}}}
Reviewing: my_verticle_steve
Change detected with: my_verticle_steve, new conf: {"name":"my verticle steve","path":"verticles/vert1/vert1.js","worker":false,"instances":2,"isolationGroup":null,"isolatedClasses":null,"config":{"value1":"dog","value2":"cat"}}
Deploying: my_verticle_steve from path verticles/vert1/vert1.js
All verticles in config have been reviewed
Verticle my_verticle_steve(82d3a66e-a148-40d7-ad90-41ad99d0386c) has been undeployed
Deployments current active: [37913bdd-9aa6-4524-8e1b-6b8a84f474af]
vert 1 sending a message
Vert 1 confg: {"value2":"cat","value1":"dog"}
vert 1 sending a message
Vert 1 confg: {"value2":"cat","value1":"dog"}
Deployment id is: bf166ca9-bb1c-4808-8c8a-aa616ae8c853
Deployments current active: [bf166ca9-bb1c-4808-8c8a-aa616ae8c853, 37913bdd-9aa6-4524-8e1b-6b8a84f474af]
```


# Build and get started

1. Install npm dependencies: `./gradlew npmInstall`

2. Run the project locally: `./gradlew clean run`

3. Build a FatJar for deployment on other systems: `./gradlew clean shadowJar`