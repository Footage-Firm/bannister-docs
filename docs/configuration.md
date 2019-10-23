## Directory structure

In the root of your code base, create a folder called `.ciblocks`.

Inside this folder, we will need to add two files:

* `config.yaml` which contains basic instructions on how to build and run your codebase
* `docker-compose.yml` which defines the `docker-compose` configuration for spinning up an instance of your project

## `config.yaml`

### `version`

Changing the version allows you to use newer CiBlocks features, the newest version number is currently `0.1`

### `settings`

This section specifies general project settings

#### `docker-compose`

```
settings:
    docker-compose:
        configs:
          - .bannister/docker-compose.yml
```

### `tasks`

This section defines what happens when a new test run has been invoked.

Currently, 4 types of tasks are supported:

* `checkout`
* `launch_containers`
* `build`
* `run tests`

#### `checkout`

This clones and checks out the code for a given commit. This is usually the first step.

No additional configuration is available for this step.

#### `launch_containers`

This essentially runs `docker-compose up` with the given docker-compose yaml configuration file.

No additional configuration is available for this step.

#### `build`

This performs any necessary build steps prior to running test suites.

You can use this step to build assets, run migrations, generate seed data, etc.

The following build steps are available:

* `build_image`
* `exec`
* `run_container`

##### `build_image`

Builds a new docker image and pushes it to the default registry.

```
- build_image:
    label: Building my image # Description of the step
    name: "myorganization/myimage" # Name of image to build
    tag: "{{ commitHash }}" # What to use as the image tag
    path: "{{ checkoutPath }}" # The build context/root path for the image
    dockerfile: "Dockerfile.build" # This is optional, defaults to "Dockerfile"
```

##### `exec`

Executes a command within the docker-compose environment.

```
- exec:
    label: Delete everything # Description of the step
    container: # The name of the docker-compose container to run this command within
    commands: rm -rf / # Don't try this at home
```

##### `run_container`

Creates a new container and runs a command within it. Useful if you need to run a command, using an image that is *not* used in your docker-compose setup.

```
- run_container:
    image: myorganization/assets-builder # Image to pull and run
    commands: npm run build assets
    volumes: "{{ checkoutPath }}:{{ checkoutPath }}" # You can mount any path(s) here
    environment:
        ENV_VAR_1:    #leave this blank to pull from your project envs
        OTHER_ENV_VAR: "some_value"
    network: some_other_network  # this is always appened with your current workspace id
```

#### `run_tests`

Whew! The environment is finally configured and running, now all we have left is to run those sweet tests.

You can specify a `test_suite` for each type of test tht you want to run.

Here's an example:

```
- test_suite:
    label: Run Selenium Tests
    type: selenium
    image: "myorganization/selenium:{{ commitHash }}"
    tests: java -cp 'lib/*:build/classes' meta.DumpTests # This can also be a static list of test units to run, if a string is given, this command is executed using a new container created with the above image
    command: ant runtest -f /selenium/build.xml -Dtest={{ testUnitClass }} -Dmethods={{ testUnitMethod }}
```

> **Note:** Currently, `selenium` is the only supported test type
