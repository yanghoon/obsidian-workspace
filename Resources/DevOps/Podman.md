#docker #container 

## Buildah

## Containerize Vue

```bash
# Step 1: Build the application  
FROM node:16 AS builder  
WORKDIR /app  
COPY package.json package-lock.json ./  
RUN npm ci  
  
COPY . .  
RUN npm run build  
  
# Step 2: Set up the production environment  
FROM nginx:stable-alpine  
COPY --from=builder /app/build /usr/share/nginx/html  
COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf  
  
EXPOSE 8080  
CMD ["nginx", "-g", "daemon off;"]
```

## References
* 
	* [without Docker](https://netpple.github.io/docs/make-container-without-docker/)
* Optimizing Size
	* https://dev.to/agustinoberg/dockerize-vitereactjs-application-6e1
* Best Practies
	* [Node.js 웹 앱의 도커라이징](https://nodejs.org/ko/docs/guides/nodejs-docker-webapp)
	* [Docker and Node.js Best Practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md#docker-and-nodejs-best-practices)
