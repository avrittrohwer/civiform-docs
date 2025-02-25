# Getting started

This guide helps you get started running and interacting with a local development server and tests. If you are a Full Time SWE onboarding, first check out the links/tutorials/account-settup detailed in [New Full Time SWE Onboarding Material](new-full-time-swe-onboarding-materials.md).

## Setting up your environment

Start here! This step is a prerequisite for everything that follows, even if you only want to interact with a local server without pushing changes. If you're working in Windows, check out [Getting started with Windows](getting-started-with-windows.md).

1. [Join GitHub](https://github.com/join).
2. [Install git](https://github.com/git-guides/install-git) on your machine.
3. [Configure an SSH key with your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh). We recommend not including the "UseKeychain yes" line or setting a password.
4. Install Docker
   * On Mac
     * Download [Docker Desktop](https://www.docker.com/get-started). 
     * Run Docker Desktop and go to Preferences > Resources to increase max RAM for containers to at least 4GB (ideally 6GB), otherwise sbt compiles can get killed before completing and produce strange runtime behavior.
   * On Linux
     * Install [Docker Engine](https://docs.docker.com/engine/install/#server) for your Linux distribution.
     * Configure Docker to run in [Rootless Mode](https://docs.docker.com/engine/security/rootless). This will create generated files as the correct user, otherwise file permission issues occur.
     * Install Docker compose v2 by following [these instructions](https://docs.docker.com/compose/cli-command/#install-on-linux).
5. Clone the CiviForm repo. This will create a copy of the codebase on your machine:
   1. Open a terminal and navigate to the directory you'd like the copy of the CiviForm codebase to live.
   2. In that directory, run the following (and/or refer to [this guide](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/cloning-a-repository)):

       ```
       git clone git@github.com:civiform/civiform.git
       ```

### A note on IDEs

#### Running SBT

You may use whichever IDE you prefer, though _DO NOT_ use the IDE's built-in sbt (Scala Build Tool) shell if it has one. Instead, run `bin/sbt` to bring up an sbt shell inside the Docker container. The sbt shell allows you to compile, run tests, and run any other sbt commands you may need

#### Configuring IntelliJ

The easiest way to get IntelliJ to index the project correctly is to install the Scala plugin and open the project by specifying the [build.sbt](https://github.com/civiform/civiform/blob/main/server/build.sbt) file in IntelliJ's open existing project dialog. If this isn't done correctly IntelliJ probably won't load library code correctly and will complain that it can't find symbols imported from libraries.

This setup will only include the files under (.../server), therefore, if you would like to edit other files under .../civiform, you need to add additional modules manually:

* Go to File/Project Structure, 
* Select modules 
* Ensure `civiform-server` and `sources` are selected in second and third column,
* Click on + on the right handside to add all folders under civiform except "server" to the content roots 

If you still have trouble getting some symbols to show (such as `routes` packages), try the following:

1. Go to Preferences, then "Languages and Frameworks", then "Scala".
2. Switch Error Highlighting from "Built-in" to "Compiler".

#### Configuring VSCode

**Setup**

1. Install the following extensions:
    * [Extension Pack for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)
    * [Scala Syntax (official)](https://marketplace.visualstudio.com/items?itemName=scala-lang.scala)
    * [Scala (Metals)](https://marketplace.visualstudio.com/items?itemName=scalameta.metals)
2. Run `bin/vscode-setup` to generate a pom.xml file that allows VSCode to resolve dependencies
3. Open the workspace file `civiform.code-workspace` in VSCode
4. Metals automatically detects the project. Click `Import Build` at the prompt. <img width="676" alt="Screen Shot 2022-05-06 at 9 28 17 AM" src="https://user-images.githubusercontent.com/1870301/167177672-45594f16-cfea-4f8c-845d-8c266443acc3.png">
5. Choose `sbt` at the prompt for multiple build definitions. <img width="665" alt="Screen Shot 2022-05-06 at 9 51 24 AM" src="https://user-images.githubusercontent.com/1870301/167177704-cc3c290f-5cda-4776-88f5-a88573af3662.png">


**Troubleshooting**

* If source code isn't being indexed / symbols aren't found, you may need to clean the Java workspace. Trigger the command pallet and select `Java: Clean Java Language Server Workspace`
* If a new dependency is added, the `pom.xml` file may be out of date. You'll need to either add the dependancies manually, or re-run the bin/vscode-setup script to regenerate the file.
  * Here's a [guide to pom.xml setup](https://yongjie.codes/guides/java-play-sbt-on-vscode/) with the play framework

**Important files**

VSCode uses the following files

* server/pom.xml (maven dependancies for symbols)
* server/.settings/\*. (generated by `sbt eclipse`)
* civiform.code-workspace (opens the top level directory, and server)

## Running a local server

**Note:** the project makes heavy use of shell scripts for development tasks. Run `bin/help` to see a list of them.

1.  To run the application, navigate to the `civiform` directory you created when you cloned the repo and run the following:

    ```
    bin/run-dev
    ```

    To run the application using Azure instead of AWS, run:

    ```
    bin/run-dev –-azure
    ```

This will start up a dev instance of the application that uses Azurite, the Azure emulator, instead of local stack, the AWS emulator. This script sets an environment variable, `STORAGE_SERVICE_NAME` telling the application to run using the Azure emulator instead of AWS.

**Tip**: Once you have the local server working, you may want to set `USE_LOCAL_CIVIFORM=1` by default in your environment.  This will speed up reruns by not always redownloading the latest docker images (1-2GB each) which is not usually necessary.

**Warning**: There's a [known issue](https://github.com/civiform/civiform/issues/2230) where you may encounter a compile loop, the most reliable way to address that is to do `bin/sbt clean` before the above.

2. Once you see "Server started" in your terminal (it will take some time for the server to start up), you can access the app in a browser at http://localhost:9000. Be patient on the initial page load since it will take some time for all the sources to compile.

If you want to use the Log In flow see [those instructions](authentication-providers.md#testing) which include a one-time setup too.

The `bin/run-dev` script uses `docker-compose` (see [`docker-compose.yaml`](https://github.com/civiform/civiform/blob/main/docker-compose.yml)). It enables Java and Javascript hot-reloading: when you modify most files, the server will recompile and restart. This is pretty time-consuming on first page load, but after that, it's not so bad.

### Setting up routing for local testing

For login and file upload to work with your local server, you need to edit your `/etc/hosts` file to include the following:

```
127.0.0.1 dev-oidc
127.0.0.1 azurite
127.0.0.1 localhost.localstack.cloud
# Required for test AWS S3 bucket
127.0.0.1 civiform-local-s3.localhost.localstack.cloud
```
This provides a local IP route for the 'dev-oidc', 'azurite', and 'localstack' hostnames.

### Seeding the development database

Creating questions and programs in CiviForm is a bit time consuming to do manually with the UI, so in dev mode there is a controller action that generates several for you. You can access this feature at `localhost:<port number>/dev/seed`.

### Help! It's not working!

We know setting up a development environment can have some snags in the road. If something isn't working, check out our [Troubleshooting ](troubleshooting.md)guide or reach out on [Slack](https://join.slack.com/t/civiform/shared\_invite/zt-niap7ys1-RAICICUpDJfjpizjyjBr7Q).

## Running tests

This section will help you run CiviForm unit and browser tests in a basic way. For more information on _writing_ and _debugging_ these tests, check out the [Testing ](testing.md)guide.

### Running java unit tests

To run the java unit tests (includes all tests under [test/](https://github.com/civiform/civiform/tree/main/server/test)), run the following:

```
bin/run-test
```

If you'd like to run a specific test or set of tests, and/or save sbt startup time each time you run the test(s), use the following steps. This is recommended for developer velocity.

1.  Bring up an sbt shell inside the Docker container by running:

    ```
    bin/sbt-test
    ```
2.  Run any sbt commands! For example:

    ```
    testOnly services.question.types.QuestionDefinitionTest

    test
    ```
    
### Running typescript unit tests

To run the unit tests in [app/assets/javascripts](https://github.com/civiform/civiform/tree/main/server/app/assets/javascripts), run the following:

```
bin/run-ts-tests
```

If you'd like to run a specific test or set of tests, run the following:

```
bin/run-ts-tests file1.test.ts file2.test.ts
```

### Running browser tests

To run the browser tests (includes all the [Playwright](https://playwright.dev) tests in [`civiform/browser-test/src/`](https://github.com/civiform/civiform/tree/main/browser-test/src), there are three steps:

1.  Bring up the local test environment using the AWS emulator. Leave this running:

    ```
    bin/run-browser-test-env
    ```

2.  Once you see "Server started" in the terminal from the above step, in a separate terminal run the tests in a docker container:

    ```
    bin/run-browser-tests
    ```

    Or, to run a test in a specific file, you can pass the file path relative to the `browser-test/src` directory:

    ```
    bin/run-browser-tests landing_page.test.ts
    ```

## Tips

Browser tests are heavy handed and can take a while to run.  You can focus the run to only execute a single `it` test or `describe` suite per file by prefixing it with `f`.  EG: `fit` and `fdescribe`.

## Creating fake data

To create Questions and Programs that use them, you need to log in as a "Program and Civiform Admin" through http://localhost:9000/loginForm .

You can return to that screen to switch to a Guest user and back again to an Admin as needed.

## Debug logging

You can change the logging levels by editing [conf/logback.xml](https://github.com/civiform/civiform/blob/main/server/conf/logback.xml). This can help get a deeper understanding of what the server is doing for development.

## Running Coverage

To generate coverage report, run the following:
   
  ```
    bin/sbt-run-test-coverage
  ```

Navigate to server/code-coverage/report/html/index.html and see the detailed report of the code coverage data and also dig deep to see how much your implemented classes are covered.

## What's next?

To learn more about how to make code contributions, head to [Technical contribution guide.](technical-contribution-guide.md)

To learn about the CiviForm tech stack and standards, head to [Technology overview ](technology-overview.md)and [Development standards.](development-standards.md)
