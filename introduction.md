---
description: Development tools and best practices
---

# Introduction

`fastrepo` is a tool to generate python project repositories from templates. It comes with several templates by default:

* `default`: Boilerplate for a standard python package.
* `rest`: Boilerplate for Rest API written using FastAPI as a python package.
* `nats`: Boilerplate for an NATS application written using FastSTAN as a python package.
* `react-web:` Boilerplate for a ReactJS web application as a javascript \(npm\) package.
* `vue-web`: Boilerplate for a VueJS application as a javascript \(npm\) package.
* `datascience`: Boilerplate for a datascience project as a python package.

This documentation describes not only the usage of the `fasterpo` command line tool, but also tries to explain and illustrate development best practices for each one of the available templates.

## Features

* Automate package management configuration
  * Manage dependencies
  * Build package
  * Publish package
* Automate development tools configuration
  * Linter
  * Formater
  * Test Runner
  * Test Coverage
  * Type Checking
* Automated version release
  * Bump Versions
  * Generate Changelogs
* Git Hook integration
  * Always lint and format before commit
  * Always lint commit message
  * Prevent large files from being added to repository
* Docker integration
  * Build Docker Image
  * Deploy Docker Services
* VSCode Integration
  * Lint/Format on Save
  * Run tests
  * Debug



