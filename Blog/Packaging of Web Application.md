# Overview

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

Local Development
* Hot Reload
* Test Configurations (Proxy for CORS)
* Different Developers (Frontend and Backend)

Build Pipeline
* Checkout
* Build
* Containerize
