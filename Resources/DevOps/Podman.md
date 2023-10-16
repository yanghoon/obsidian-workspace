#docker #container 

## Buildah

## Containerize Vue

```bash
FROM mhart/alpine-node:12 AS builder
WORKDIR /app
COPY . .
RUN yarn install
RUN yarn run build
FROM nginx:1.16.0-alpine
COPY --from=builder /app/dist /usr/share/nginx/html
RUN rm /etc/nginx/conf.d/default.conf
COPY deploy/nginx/nginx.conf /etc/nginx/conf.d
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## References
* [without Docker](https://netpple.github.io/docs/make-container-without-docker/)
* Optimizing Size
	* [Dockerize Vite+ReactJS application](https://dev.to/agustinoberg/dockerize-vitereactjs-application-6e1)
* Best Practies
	* [Node.js 웹 앱의 도커라이징](https://nodejs.org/ko/docs/guides/nodejs-docker-webapp)
	* [Docker and Node.js Best Practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md#docker-and-nodejs-best-practices)
