# Overview

Let's think about how to package a simple web application consist of Spring Boot and SPA. SPA framework can build source codes as some static resources and that can be packaged within the Jar file. In this article, we consider best practices of this case.

The baseline of environments that we can not change are below.

|Category|Frontend|Backend|Note|
|-|-|-|-|
|Runtime|Node(Development, Build), Browser|Java||
|Framework|Vue, React|Spring Boot||
|Build Tool|NPM, Yarn(Package Manager)<br>Vite(Build, Dev Server)|Gradle, Maven||
|Artifact|Web Resources(html, css, js)|Jar||
|Containerize|Dockerfile|Jib Gradle Plugin||

## Repository Patterns

1. Mono Repo
2. Multiple Repo
## Packaging Patterns

1. Build and Containerize Each Special Tool
	1. Mono Repo or Multiple Repo
2. Build with Each Tool and Package Single Jar File
	1. Mono Repo
## Uscase

Before talking about packaging patterns, we can also think about use cases. Development experiences should be considered as much as a build pipeline process.
### Local Development

* Hot Reload
* Test Configurations (Proxy for CORS)
* Collaboration (Frontend and Backend)
	* Same or Different Developers
	* Git History

### Pipeline

* Checkout
* Build
* Containerize
* Deploy (Infrastructure Code)
