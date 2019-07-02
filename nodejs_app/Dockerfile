#FROM node:7
FROM node:alpine
EXPOSE 8080

RUN apk add --no-cache tini

WORKDIR /app

COPY package.json /app
RUN npm install
COPY . /app

ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "index.js"]

