FROM node:10.15.0-alpine

RUN apk update && apk add openssh vim bash

WORKDIR /tmp

COPY package.json /tmp/

RUN npm install

WORKDIR /usr/src/app

COPY . /usr/src/app/

RUN cp -a /tmp/node_modules /usr/src/app/

EXPOSE 80

CMD ["bash","docker_start_script.sh"]