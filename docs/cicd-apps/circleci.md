---
sidebar_position: 2
---

# Circle CI

## Circle CI and Testing Results
:::info
Circle CI is one of the most popular SAAS CI applications and is used by thousands of companies all over the world. Circle CI is used to define the application build and test processes. QA teams also use Circle CI to run/schedule functional tests across a variety of tools and frameworks. However, the results of these tests can only be viewed in Circle CI, and it is not really possible to aggregate results across multiple jobs, retain long history, and delegate test failures to your team.
:::

## TestRail + Circle CI + Railflow CLI
:::info
By using the Railflow CLI, you can easily integrate Circle CI testing jobs with TestRail and automatically export all testing reports to TestRail. Aggregating result from all your Circle CI jobs into TestRail allows teams to look at test trends, auto-assign failures via Railflow automation, create link between Circle CI and TestRail, and much more.
:::info


## Docker to the Rescue
:::tip
[Railflow CLI Docker Image](https://hub.docker.com/r/railflow/railflow) can be easily incorporated within your Circle CI pipelines to quickly and easily integrate Circle CI with TestRail.
:::

In this example below, we are building and publishing the test results from a TestNG project. The CircleCI pipeline uses the [Railflow Docker Image](https://hub.docker.com/r/railflow/railflow) which is available on DockerHub.


```jsx title="Circle CI Pipeline Example"
version: 2
jobs:
  build:
    docker:
      - image: circleci/openjdk:8-jdk

    working_directory: ~/repo

    environment:
      # Customize the JVM maximum heap limit
      JVM_OPTS: -Xmx3200m
      TERM: dumb

    steps:
      - checkout
      # run tests!
      - run: mvn clean test || true
      - run: ls .
      - store_artifacts:
          path: ./target/surefire-reports/testng-results.xml

  railflow_test:
    docker:
      - image: railflow/railflow:latest

    working_directory: /usr/railflow
    steps:
      - run:
          name: download artifacts
          command: |
            curl -H "Circle-Token: $CIRCLE_TOKEN" https://circleci.com/api/v1.1/project/github/railflow/testng_example/10/artifacts \
            | grep -o 'https://[^"]*' \
            | wget --verbose --header "Circle-Token: $CIRCLE_TOKEN" --input-file -
      - run:
          name: Run railflow testing
          command: npx railflow -k $RAILFLOW_KEY -url $TESTRAIL_URL -u $RAILFLOW_USERNAME -p $RAILFLOW_PASSWORD -pr "CircleCI-Demo" -path Demo/TestNG -f testng -a john@foo.com, jane@foo.com -r /usr/railflow/testng-results.xml -sm path -tp TestPlanName

workflows:
  version: 2
  build_and_test:
    jobs:
      - build
      - railflow_test:
          requires:
            - build

```


## Circle CI Orb
:::tip
[Railflow Circle CI Orb](https://circleci.com/developer/orbs/orb/railflow/railflow-orb) is another method that can quickly and securly be integrated in your Circle CI pipelines.
:::

:::info
#### What are Orbs?
Orbs are reusable snippets of code that help automate repeated processes, speed up project setup, and make it easy to integrate with third-party tools.
<div style={{display: 'flex', alignItems: 'center', lineHeight: "26px"}}>
<svg role="img" fill="none"style={{marginRight: '10px'}} height="48" viewBox="0 0 48 48" width="48" xmlns="http://www.w3.org/2000/svg" aria-label="response time icon"><g stroke="currentColor" strokeLinecap="round" strokeWidth="2"><path d="m21 41h-8"></path><path d="m15.32 33.1h-12.32"></path><path d="m21 37h-6"></path><path d="m11 37h-2.5"></path><path d="m24.97 16.9351v8.05h4.025"></path><path d="m11 23c0-7.732 6.268-14 14-14s14 6.268 14 14-6.268 14-14 14"></path><path d="m11.675 27.31c-.4499-1.3923-.6777-2.8468-.675-4.31"></path><path d="m7 23c0-9.9411 8.0589-18 18-18"></path><path d="m25 5c9.9411 0 18 8.0589 18 18"></path><path d="m43 23c0 9.9411-8.0589 18-18 18"></path><path d="m7 23c0-9.9411 8.0589-18 18-18"></path><path d="m8.25501 29.625c-.83191-2.1096-1.25771-4.3573-1.255-6.625"></path></g></svg>
One of the main advantages using orbs is to save time on project configuration and abstract away complexity
</div>
:::

In this example below, we are building and publishing the test results from a TestNG project. The CircleCI pipeline uses the [Railflow Circle CI Orb](https://circleci.com/developer/orbs/orb/railflow/railflow-orb) we released as open source.


```jsx title="Circle CI Pipeline Example using Railflow Orb"
---
version: 2.1
orbs:
  railflow: railflow/railflow-orb@0.0.5

jobs:
  build:
    docker:
      - image: circleci/openjdk:8-jdk

    working_directory: ~/repo

    environment:
      # Customize the JVM maximum heap limit
      JVM_OPTS: -Xmx3200m
      TERM: dumb

    steps:
      - checkout
      # run tests!
      - run: mvn clean test || true
      - run: ls .
      - store_artifacts:
          path: ./target/surefire-reports/testng-results.xml

      - persist_to_workspace:
          root: ~/repo
          paths:
            - ./target

  railflow_orb_test:
    working_directory: ~/repo
    executor: railflow/default
    shell: /bin/bash --login -o pipefail
    environment:
      # secrets are stored in Circle CI project environment variables
      # RAILFLOW_KEY
      # RAILFLOW_USERNAME
      # RAILFLOW_PASSWORD

      RAILFLOW_URL: https://testrail6.railflow.io/
      RAILFLOW_PROJECT_NAME: Github-Demo
      RAILFLOW_PROJECT_PATH: Demo/TestNG
      RAILFLOW_REPORT_FORMAT: testng
      RAILFLOW_REPORT_PATH: ~/repo/target/surefire-reports/testng-results.xml
      RAILFLOW_SEARCH_MODE: path
      RAILFLOW_TEST_PLAN: TestPlanName
      RAILFLOW_ASSIGN_FAILURE_TO: assign-email@example.com
    steps:
      - checkout
      - attach_workspace:
          at: ./
      - railflow/test:
          RAILFLOW_KEY: $RAILFLOW_KEY_RAZVAN
          RAILFLOW_URL: $RAILFLOW_URL
          RAILFLOW_USERNAME: $RAILFLOW_USERNAME
          RAILFLOW_PASSWORD: $RAILFLOW_PASSWORD
          RAILFLOW_PROJECT_NAME: $RAILFLOW_PROJECT_NAME
          RAILFLOW_PROJECT_PATH: $RAILFLOW_PROJECT_PATH
          RAILFLOW_REPORT_FORMAT: $RAILFLOW_REPORT_FORMAT
          RAILFLOW_REPORT_PATH: $RAILFLOW_REPORT_PATH
          RAILFLOW_SEARCH_MODE: $RAILFLOW_SEARCH_MODE
          RAILFLOW_TEST_PLAN: $RAILFLOW_TEST_PLAN
          RAILFLOW_ASSIGN_FAILURE_TO: $RAILFLOW_ASSIGN_FAILURE_TO

      - store_artifacts:
          path: $RAILFLOW_REPORT_PATH
          destination: railflow-artifacts
workflows:
  version: 2.1
  build_and_test:
    jobs:
      - build
      - railflow_orb_test:
          requires:
            - build
```

### Environment variables
When using Railflow ORB, is it crucial to configure the environment variables to fully benefit from the tool. Bellow is a description of each of them.

| Name                   | Description               |
| -----------------------| ------------------------- |
| RAILFLOW_KEY               | License key for online activation                                                                                                                                                                                                                                                                                                                                                                                                                   |
| RAILFLOW_USERNAME          | TestRail username                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| RAILFLOW_PASSWORD          | TestRail password or API Key                                                                                                                                                                                                                                                                                                                                                                                                                        |
| RAILFLOW_URL               | TestRail instance URL                                                                                                                                                                                                                                                                                                                                                                                                                               |
| RAILFLOW_PROJECT_NAME      | TestRail project name                                                                                                                                                                                                                                                                                                                                                                                                                               |
| RAILFLOW_PROJECT_PATH      | TestRail test cases path                                                                                                                                                                                                                                                                                                                                                                                                                            |
| RAILFLOW_REPORT_PATH       | The file path(s) to the test report file(s) generated during the build.<br /> User can pass multiple values separated with spaces.<br /> Ant-style patterns such as **/surefire-reports/*.xml can be used.<br /> E.g. use target/surefire-reports/*.xml to capture all XML files in target/surefire-reports directory                                                                                                                                           |
| RAILFLOW_REPORT_FORMAT     | Report format: JUnit, TestNg, TestNg-Steps, Cucumber, NUnit, Allure, Robot, TRX (case insensitive)                                                                                                                                                                                                                                                                                                                                                  |
| RAILFLOW_SEARCH_MODE       | Specifies the test case lookup algorithm.<br /> - 'name': search for test case matching the name within the entire test suite. If test case found, update the test case. If test case not found, create a new test case within specified `-path`<br /> - 'path': search for test case matching the name within the specified `-path`. If test case<br /> found, update the test case. If test case not found, create a new test case within<br /> specified `-path` |
| RAILFLOW_TEST_PLAN         | TestRail test plan Name                                                                                                                                                                                                                                                                                                                                                                                                                             |
| RAILFLOW_ASSIGN_FAILURE_TO | Smart Test Failure Assignment. File path containing list of TestRail users (email addresses). Note: One user per line                                                                                                                                                                                                                                                                                                                               |



## TestRail Export Details
Railflow creates a very rich and flexible integration between Circle CI and TestRail. Based on Railflow configuration, TestRail entities can be created or updated automatically as part of your CICD process. The screenshots below shows the output of processing a typical JUnit test framework report.

### Circle CI Job Output
![circleci job output ](/img/cicd/circleci/circle-build-output.png)

### Automatic Test Creation
![automatic test creation ](/img/cicd/jenkins/plugin-exec-3.png)

### Automatic Test Plan and Runs
![automatic testplan ](/img/cicd/jenkins/plugin-exec-4.png)
![automatic testrun ](/img/cicd/jenkins/plugin-exec-5.png)

### Test Results Details
![test results ](/img/cicd/jenkins/plugin-exec-6.png)

### Automatic Milestones
![automatic milestones ](/img/cicd/jenkins/plugin-exec-7.png)


## Smart Failure Assignment
:::info
Smart Failure assignment is a very powerful feature of Railflow and allows teams to efficiently and strategically assign test failures to specified team members. Doing this automatically as part of the CI process means that teams don't waste valuable time triaging test failures.
:::

:::note
To use Smart Failure Assignment feature, the users need to have `Global Role` under `Project Access`.
:::

![smart failure](/img/cicd/jenkins/smart-failure-5.png)

### Example
Consider a Circle CI Selenium Webdriver project build is failing with 5 test failures, and 2 users configured in the Railflow CLI

![circleci build logs](/img/cicd/circleci/circle-smart-assign.png)

### Circle CI Build Logs

![circleci build logs](/img/cicd/jenkins/smart-failure-2.png)

### TestRail Results View
In this example above, Railflow's Smart Assignment feature filtered all the test failures and distributed them equally across the specified users. This nifty feature greatly eliminates the manual test triage process and saves teams valuable time.

![smart failure](/img/cicd/jenkins/smart-failure-3.png)
![smart failure](/img/cicd/jenkins/smart-failure-4.png)


