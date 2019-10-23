# Step 1 - Prepare
The goal of this step is to ensure that your application fulfills the basic requirements for moving onto Codefresh and Kubernetes.

It is highly recommended that you finish all the tasks stated in this document to help ensure a good experience when completing the next steps.

Most of the tasks are related to 12-factorizing your applications. You can read more about the 12 factors [here](https://12factor.net/). Following these patterns ensures that the application will work well with Containers/Kubernetes, can be easily tested on Codefresh, and be used in dynamic, ephemeral environments and integration tests.

## Requirements
* Your application works in development
* You understand the dependencies of your application
* You understand which settings change the run-time behavior of your application.

## Tasks
* [Use environment variables to configure your application](#use-environment-variables-to-configure-your-application)
* [Consolidate your environments](#consolidate-your-environments)
* [Run your application as a service](#run-your-application-as-a-service)
* [Supply a working `Dockerfile` to run the application](#supply-a-working-dockerfile-to-run-the-application)
* [Setup a good `.dockerignore` file](#setup-a-good-dockerignore-file)
* [Keep your application stateless](#keep-your-application-stateless)
* [Utilize logs](#utilize-logs)
* [Define and document your dependencies](#define-and-document-your-dependencies)
* [Retrieve secrets from vault](#retrieve-secrets-from-vault)
* [Optional: Provide a docker based development workflow](#optional-provide-a-docker-based-development-workflow)
* [Use environment variables to configure your application](#use-environment-variables-to-configure-your-application)

### Use environment variables to configure your application
In a nutshell, you have to make sure that settings can be passed to the application via environment variables at *run-time*. This also means that connection information to backing services (database servers, URLs for APIs you consume, …) need to be passed in via environment variables. This enables us to dynamically discover services in ephemeral environments.

> The run-time part might be a bit tricky for front-end applications. We have our own solution to enable this: https://github.com/hoverinc/front-end-fastify-server

A couple things to keep in mind:
* Don’t spread reading the environment variables throughout the codebase. Reading the environment variables should only happen when the application boots, and in a well-defined places (e.g. config files, initializers, …).
* Feel free to use your own module/class to access the settings.
* Use common libraries/techniques that allow you to interpolate environment variables into a configuration file
* Set meaningful defaults for the settings so that the application can be easily be executed for development and testing purposes.
* You might be using an `.env` file in development to make your life easier. The `.env` will automatically be read up by Docker Compose and libraries like `dotenv` to inject environment variables into the application. However, do not check this file into source control. Feel free to provide an example (`.env.example`) with meaningful defaults that developers can copy.
* Stop using the legacy service discovery environment variables like `POSTGRES_PORT_5432_TCP_ADDR` to discover backing services. Either use `POSTGRES_URL` or two environment variables for the host and port (`POSTGRES_HOST` and `POSTGRES_PORT`). The same is true for Redis (`REDIS_URL`, `REDIS_HOST`, `REDIS_PORT`, `REDIS_DB`) and other services. These deprecated environment variable names are long, hard to pass, and deliver a wrong message - the port is independent of the host 
* Credentials should be read from environment variables.
* Document the supported and required environment variables.

> Read more about this in [Factor 3](https://12factor.net/config) and [Factor 4](https://12factor.net/backing-services)

### Consolidate your environments

Your application should only have one environment from an application perspective. Well, ok - there will be three for applications based on Rails and other frameworks: `development`, `test`, `production`. We do not want any additional environments like `qa` or `staging`. Those environments should use the `production` environment and change the parts of the configuration that are different using environment variables.

> If you are using more than the 3 environments mentioned above, it is recommended to keep doing so until you reach step 5 and deploy to Kubernetes. Just prepare your `production` environment to use environment variables for every setting that needs to be different for the various environments. While doing this, configure the default values for those settings to be the ones for production and document the settings for the other environments. This will make the transition easier and save everybody time.

### Run your application as a service
Your application should be self-contained. That means it runs as its own process.

> For frontend applications, this means that you ship a web server that serves your application. We have our own solution based on Fastify to enable this: https://github.com/hoverinc/front-end-fastify-server

As an org, we need to ensure that services we deploy are up and running. The team developing a service is responsible for providing a mechanism to determine whether an app is running or not.
* Provide a `/status`-endpoint that can be used to determine whether the application is up and running if you are running a web application
* Provide a script to determine if the application is up and running in other cases

> Read more about this in [Factor 6 - Execute the app as one or more stateless processes](https://12factor.net/processes)) and [Factor 7 - Export services via port binding](https://12factor.net/port-binding)


### Supply a working `Dockerfile` to run the application
We run all applications in Docker containers. Creating and maintaining the `Dockerfile` that is used to create the container image is the respective teams' responsibility. Docker allows us to bundle our application and its dependencies into a portable package that we can test and run anywhere. Docker also helps us separate the build from the release and run stages in our applications' lifecycle.

There are many great resources that teach you how to write a `Dockerfile`. We already run a variety of applications using Docker. Copying and changing one of the existing `Dockerfile`s might be a good approach.

A couple things to keep in mind:
* Don’t store secrets (or ssh keys) in the image! 
This means that you should not put secrets in the `Dockerfile` itself or pass them in via the `ARG` instruction. Run-time variables should be set at run-time only! If you require a secret at build time, for example to access private GitHub repos, use [buildkit](https://docs.docker.com/develop/develop-images/build_enhancements/). You can also use a multi-stage build and pass in a secret using `ARG` *if* the build stage is not part of the final image! (TODO: Add a link)
* If you have slightly different requirements for development and production, utilize multi-stage builds. You can have a stage for development and production. Make sure to keep differences to a minimum to ensure maximum dev/prod parity.

*TODO*: Link to example FE/BE `Dockerfile`s
*TODO*: Add example how to access private GH repos at build time

> Read more about this in [Factor 2 - Dependencies](https://12factor.net/dependencies), [Factor 5 - Build, release, run](https://12factor.net/build-release-run), and [Factor 10 - Dev/prod parity](https://12factor.net/parity)


### Setup a good `.dockerignore` file
The `.dockerignore` file controls which files are excluded when we build our container images. This file is important and helps us to
* keep image sizes small - we only add what is really required
* ensures we don’t accidentally put secrets or other sensitive information in our images

You should exclude everything that is not required to build the image and run the application. We generally want to exclude at least:
* `codefresh*.yml`
* `Dockerfile*`
* `docker-compose*.yml`
* `.git*`
* `.env*`

### Keep your application stateless
Don't store any state in the file-system and/or memory. Instances of your application will be started and stopped without any warning in order to scale and deploy the service. That means that you should think about the containers running the applications as a disposable construct. Any data that you care about needs to be stored in a stateful backing service like a database management system (PostgreSQL), key-value stores (Redis), or another suitable service.

Also, you should generally avoid writing to the file-system if possible. Writes to the container file-system are slow and might fill up the file-system of the underlying node. If you need to write temporary data to disk, clean up after your self.

> Read more about this in [Factor 6 - Execute the app as one or more stateless processes](https://12factor.net/processes)

### Utilize logs
Make use of logging and log, log, log. The ability to dig into logs is a big help when it comes to troubleshooting. At least log every request the application receives and the result of the request.

It is also very important that you send logs only to `STDOUT`/`STDERR` and DO NOT write logs to files to the file-system. This allows the platform to automatically pick up the logs, attach additional tags and send them to our central logging destination.

* Use structured logging. If you output your logs wrapped in something like JSON, parsing and searching the logs becomes a lot easier.
* For Rails applications, set the `RAILS_LOG_TO_STDOUT` environment variable in your `Dockerfile`
* If you want to write logs to the file-system in development, use an environment variable to turn this behavior on/off as required.

> Read more about this in [Factor 11 - Treat logs as event streams](https://12factor.net/logs)

### Define and document your dependencies
Understanding an application's dependencies is important to get an application up and running in different scenarios:
* Development
* Unit tests
* Integration tests
* Running the application in a non-development environment
* And others

The dependencies of your application can be divided into two categories: Internal and external dependencies.

Internal dependencies are the system/language libraries and utilities that your application needs. Those types of dependencies should be declared explicitly in manifest files. That means you should use tools like Bundler (Ruby), Yarn (JavaScript), and Pipenv (Python) to declare, install, and manage your dependencies. System libraries and utilities can be declared as part of the `Dockerfile`. Make sure to specify and lock the version to install.

External dependencies classify everything that is not running as part of your applications:
* Datastores (Redis)
* Databases (PostgreSQL)
* Message queuing system (Amazon SQS)
* Downstream services (The API for your front-end)
* 3rd Party services (Stripe)

External dependencies should at least be described in a `DEPENDENCIES.md` file as part of your application repository. Describe why these dependencies are required and how they are used. Ideally you should also provide a Docker Compose configuration (`docker-compose.yml`) and scripts that allow spinning up the application AND its dependencies.

Identify and document tight coupling to other application / external services as well. For example, if your application uses a naming schema to store files consumed by other applications, those applications are tightly coupled. We can not change one without changing the other. We want to identify these dependencies and eliminate them to make the overall architecture more flexible and easier to manage. Use the `DEPENDENCIES.md` for documenting them.

### Retrieve secrets from vault
*TODO*

### Optional: Provide a docker based development workflow
To make it easier for you and others to run the application, and create compositions for integration tests, it is recommended to setup a Docker based development workflow.

*TODO*


## Examples
This sections contains examples for the various tasks that can be used as a baseline.

*TODO*
