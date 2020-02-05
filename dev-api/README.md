## api service for entity relationship graphdb

## Table of contents
* [General info](#general-info)
* [Setup](#technologies)
* [Docker](#setup)

## General info
This project implement api service for entity relationship graphdb

## Technologies
This project use [Nodejs](https://nodejs.org/en) for building service.

## Setup
In order to run this project, run the following commands:
```
yarn install
yarn start
```

## Docker
This project can be used with docker.

### Build image
In order to build image for this project, run the following command:
```
docker build -t dev-api .
```

### Run project
After building image, excute the following command to run project:
```
docker run -p 3001:3001 -d --name dev-api dev-api
```

If you want to override environment variables at the runtime, you need do the followinng operations:<br>
* Create a ```env.list``` file with the paramters will be overridden - the parameters same as in the ```env``` file.<br>
* Add ```--env-file``` parameter to command, like the following command:
```
docker run -p 3001:3001 -d --name --env-file env.list dev-api dev-api
```