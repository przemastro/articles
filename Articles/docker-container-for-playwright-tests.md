# Creating a Docker Container for Playwright and Allure

End-to-end (E2E) testing requires a well-configured environment to run browser tests and generate reports. This article explains a Dockerfile that builds a container with Playwright for automated testing and Allure for test reporting.

## 1. Choosing the Base Image

Use the official Playwright image as the base image

    FROM mcr.microsoft.com/playwright:v1.49.0-noble

The Playwright image comes pre-installed with all required browsers, avoiding the need for manual setup.

## 2.Installing Dependencies

    RUN apt-get update && apt-get install -y \
        openjdk-11-jre-headless \
        xvfb \
        x11-apps \
        xfonts-base \
        fonts-liberation \
        wget \
        unzip \
        && wget https://github.com/allure-framework/allure2/releases/download/2.17.3/allure-2.17.3.zip \
        && unzip allure-2.17.3.zip -d /opt/ \
        && rm allure-2.17.3.zip \
        && ln -s /opt/allure-2.17.3/bin/allure /usr/local/bin/allure

### What Are We Installing and Why?

- Java Runtime (openjdk-11-jre-headless) – required for Allure to work.

- xvfb – a virtual framebuffer for rendering browser tests in a headless environment.

- x11-apps, xfonts-base, fonts-liberation – necessary fonts and X11 utilities for Playwright.

- wget, unzip – used to download and extract Allure.

- Allure – installed to /opt, with a symbolic link for easy access.

## 3. Setting the Working Directory

    WORKDIR /app

This sets /app as the default working directory inside the container.

## 4. Installing Node.js Dependencies

    COPY package.json package-lock.json ./
    RUN --mount=type=cache,target=/tmp/.npm \
        npm ci --cache /tmp/.npm --prefer-offline

 - npm ci ensures a clean and consistent dependency installation.

 - Caching npm dependencies speeds up build times in CI/CD pipelines.

## 5. Copying the Test Code

    COPY . .

Copies the entire project directory into the container.

## 6. Defining the Default Command

    CMD ["npx", "playwright", "test", "--reporter=allure-playwright"]

This command runs Playwright tests and generates Allure reports.

## 7. Summary

### This Dockerfile enables:

✅ Running Playwright tests inside a container.

✅ Generating Allure reports for better test analysis.

✅ Headless execution with xvfb.

✅ Consistent dependency management using npm ci.

With this setup, you can easily execute Playwright tests in CI/CD pipelines like GitLab, GitHub Actions, or Jenkins.

