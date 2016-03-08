FROM node:5.0.0


RUN mkdir /src
ENV NODE_ENV='production'
RUN apt-get -y install python

# Expose the app server port
EXPOSE 1337

RUN apt-get update \
	&& apt-get install -y wget make g++ \
	&& apt-get install -y build-essential

## TODO: In production copy files and run from there !!

ADD . /src
# Set working directory
WORKDIR	/src
RUN npm install
RUN npm install -g forever pg pg-hstore sequelize sequelize-cli pm2 node-gyp

ENTRYPOINT ["bash", "-c", "pm2 start server.js"]
