#docker #container 

## Usage

### Install

```bash
#TODO
```

### Build Image

```bash
podman build
podman 
```

## *Buildah*

## Containerize Vue

#### Dockerfile

```dockerfile
# build stage  
FROM node:lts-alpine as build-stage  
WORKDIR /app  
COPY package*.json ./  
RUN yarn install  
COPY . .  
RUN yarn run build  
  
# production stage  
FROM nginx:stable-alpine as production-stage  
COPY --from=build-stage /app/dist /usr/share/nginx/html  
EXPOSE 80  
CMD ["nginx", "-g", "daemon off;"]
```

#### nginx.conf

```bash
```

## References
* [without Docker](https://netpple.github.io/docs/make-container-without-docker/)
* Optimizing Size
	* [Dockerize Vite+ReactJS application](https://dev.to/agustinoberg/dockerize-vitereactjs-application-6e1)
	* [Real-World Example](https://v2.vuejs.org/v2/cookbook/dockerize-vuejs-app#Real-World-Example "Real-World Example")
* Best Practies
	* [Node.js 웹 앱의 도커라이징](https://nodejs.org/ko/docs/guides/nodejs-docker-webapp)
	* [Docker and Node.js Best Practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md#docker-and-nodejs-best-practices)
