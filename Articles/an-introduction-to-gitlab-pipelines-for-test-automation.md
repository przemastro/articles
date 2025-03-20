# Automating Playwright Tests and Reporting in GitLab CI/CD

Continuous testing is crucial in modern software development. This GitLab CI/CD pipeline automates Playwright test execution and generates reports using Allure. The pipeline consists of two stages:

- **Test Stage** – Runs Playwright tests in a specified environment.
- **Deploy Stage** – Generates and stores test reports.

## 1. Defining Pipeline Stages


    stages:
      - test
      - deploy

These stages define the logical flow: first, tests run, then reports are generated.

## 2. Running Playwright Tests


    front_test:
      stage: test
      tags:
        - tag
      when: always
      image: mcr.microsoft.com/playwright:v1.49.0-noble

Uses the official Playwright image with pre-installed browsers.
Runs in all cases (when: always), ensuring tests are executed on every pipeline run.
Environment Setup Based on Branch

     script:
        - npm ci
        - |
          if [ "$CI_COMMIT_BRANCH" == "main" ]; then
            export ENV="PROD"
            export PROD_EMAIL="$PROD_EMAIL"
            export PROD_PASSWORD="$PROD_PASSWORD"
          elif [ "$CI_COMMIT_BRANCH" == "develop" ]; then
            export ENV="QA"
            export QA_EMAIL="$QA_EMAIL"
            export QA_PASSWORD="$QA_PASSWORD"
          else
            export ENV="QA"
            export QA_EMAIL="$QA_EMAIL"
            export QA_PASSWORD="$QA_PASSWORD"
          fi

main branch → Production environment (PROD)
develop branch → QA environment (QA)
Other branches default to QA
This allows secure credential management without hardcoding sensitive information.
Running Playwright Tests

    - echo "Running tests in $ENV environment"
    - echo "Using $EMAIL_AUTHENTICATION for email authentication"
    - npx playwright test

Executes Playwright tests within the configured environment.
Storing Test Results

     artifacts:
        when: always
        paths:
          - allure-results
        expire_in: 24 hours

Saves test results for 24 hours.
Results can be used in the deploy stage for report generation.

## 3. Generating Allure Reports

    front_report:
      stage: deploy
      tags:
        - tag
      when: always
      image: timbru31/java-node:8-14

Uses an image that supports both Java (required for Allure) and Node.js.
Report Generation

     script:
        - npm install -g allure-commandline
        - allure generate allure-results -o allure-report

Installs Allure globally.
Converts test results into a readable Allure report.
Storing the Reports

     artifacts:
        paths:
          - allure-report
        expire_in: 24 hours

Stores Allure reports for further analysis.

## 4. Summary

✅ Automated Playwright tests in GitLab CI/CD

✅ Environment-based configuration (Production vs. QA)

✅ Allure reports for detailed test results

✅ Artifact storage for easy access to test data

This pipeline provides scalability and automation, ensuring that tests run smoothly across multiple environments.

