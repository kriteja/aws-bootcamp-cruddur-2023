# Week 1 — App Containerization

## Summary of Week 1:
-  We're trying to containerize both frontend and backend applications and ensure they are running. Once done, we're going to orchestrate multiple containers to run side by side by writing a `docker-compose` file. Furthermore, mounting the directories so we can make changes while we code.

## Homework tasks: 
1. [Containerize Backend](#containerize-backend)
   - [Run Python](#run-python)
   - [Add Dockerfile - Backend](#add-dockerfile---backend)
   - [Build Container - Backend](#build-container---backend)
   - [Run Container](#run-container)
2. [Containerize Frontend](#containerize-frontend)
   - [Run NPM Install](#run-npm-install)
   - [Add Dockerfile - Frontend](#create-docker-file)
   - [Build Container - Frontend](#build-container---frontend)
   - [Run Container - Frontend](#run-container---frontend)
3. Multiple Containers
   - [Create a docker-compose file](#create-a-docker-compose-file)
   - [Adding DynamoDB Local and Postgres](#adding-dynamodb-local-and-postgres)
      - [Postgres](#postgres)
       - To install the postgres client into Gitpod
      - [DynamoDB Local](#dynamodb-local)
 4. [Volumes](#volumes)
    - Directory volume mapping
    - Named volume mapping 


## [Additional tasks:](#additional-tasks-1) 
1. Creation of notification feature (Backend & Frontend) with Open API 
2. Write a Flask Backend Endpoint for Notifications
3. Write a React Page for Notifications
4. Run DynamoDB Local Container and ensure it works
5. Run Postgres Container and ensure it works

## Containerize Backend

### Run Python

```sh
cd backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
cd ..
```

- make sure to unlock the port on the port tab
- open the link for 4567 in your browser
- append to the url to `/api/activities/home`
- you should get back json

![URL 404](https://user-images.githubusercontent.com/40818088/222906445-94c686a7-e9ae-4923-8fd3-ede9d04cedbd.PNG)



### Add Dockerfile - Backend

Create a Dockerfile in the following path: `backend-flask/Dockerfile`

```dockerfile
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```

### Build Container - Backend

```sh
docker build -t  backend-flask ./backend-flask
```

![Build container](https://user-images.githubusercontent.com/40818088/222909032-eb2ebc57-fb0b-462b-ab77-7948a39b3dea.PNG)

### Run Container

Run 
```sh
docker run --rm -p 4567:4567 -it backend-flask
FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
unset FRONTEND_URL="*"
unset BACKEND_URL="*"
```

Run in background
```sh
docker container run --rm -p 4567:4567 -d backend-flask
```

Return the container id into an Env Vat
```sh
CONTAINER_ID=$(docker run --rm -p 4567:4567 -d backend-flask)
```

> docker container run is idiomatic, docker run is legacy syntax but is commonly used.

### Get Container Images or Running Container Ids

```
docker ps
docker images
```


### Send Curl to Test Server

```sh
curl -X GET http://localhost:4567/api/activities/home -H "Accept: application/json" -H "Content-Type: application/json"
```

### Check Container Logs

```sh
docker logs CONTAINER_ID -f
docker logs backend-flask -f
docker logs $CONTAINER_ID -f
```

###  Debugging  adjacent containers with other containers

```sh
docker run --rm -it curlimages/curl "-X GET http://localhost:4567/api/activities/home -H \"Accept: application/json\" -H \"Content-Type: application/json\""
```

busybosy is often used for debugging since it install a bunch of thing

```sh
docker run --rm -it busybosy
```

### Gain Access to a Container

```sh
docker exec CONTAINER_ID -it /bin/bash
```

> You can just right click a container and see logs in VSCode with Docker extension

### Delete an Image

```sh
docker image rm backend-flask --force
```

> docker rmi backend-flask is the legacy syntax, you might see this is old docker tutorials and articles.

> There are some cases where you need to use the --force

### Overriding Ports

```sh
FLASK_ENV=production PORT=8080 docker run -p 4567:4567 -it backend-flask
```

> Look at Dockerfile to see how ${PORT} is interpolated

## Containerize Frontend

## Run NPM Install

We have to run NPM Install before building the container since it needs to copy the contents of node_modules

```
cd frontend-react-js
npm i
```

### Create Docker File

Create a file here: `frontend-react-js/Dockerfile`

```dockerfile
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```

### Build Container - Frontend

```sh
docker build -t frontend-react-js ./frontend-react-js
```

![Build container](https://user-images.githubusercontent.com/40818088/222913364-21e02652-857c-419f-a28e-badc69f17c06.PNG)


### Run Container - Frontend

```sh
docker run -p 3000:3000 -d frontend-react-js
```

## Multiple Containers

### Create a docker-compose file

Create `docker-compose.yml` at the root of your project.

```yaml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

## Adding DynamoDB Local and Postgres

We are going to use Postgres and DynamoDB local in future labs
We can bring them in as containers and reference them externally

Lets integrate the following into our existing docker compose file:

### Postgres

```yaml
services:
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```

To install the postgres client into Gitpod

```sh
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```

![pqsl](https://user-images.githubusercontent.com/40818088/222913330-25006756-4183-4aa6-8ddc-d61ac642b5bb.PNG)


### DynamoDB Local

```yaml
services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```

Example of using DynamoDB local
https://github.com/100DaysOfCloud/challenge-dynamodb-local

## Volumes

directory volume mapping

```yaml
volumes: 
- "./docker/dynamodb:/home/dynamodblocal/data"
```

named volume mapping

```yaml
volumes: 
  - db:/var/lib/postgresql/data

volumes:
  db:
    driver: local
```
---

Backend Port working as expected :heavy_check_mark:

![4567 Working](https://user-images.githubusercontent.com/40818088/222912558-9c4336f9-6bc2-4f0e-9b75-a521087c1ff8.PNG)

---

Frontend Port working as expected :heavy_check_mark:

![3000 Working](https://user-images.githubusercontent.com/40818088/222912977-910f8229-7f4e-4540-a77f-a0e9f7557f3e.PNG)

---

All ports :heavy_check_mark: 

![All ports](https://user-images.githubusercontent.com/40818088/222913219-9eab9323-bda5-4ce3-b702-27f1fb44201c.PNG)

## Additional tasks: 

1. [Creation of notification feature](https://www.youtube.com/watch?v=k-_o0cCpksk&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=27) with [Open API](www.openapis.org)
2. [Write a Flask Backend Endpoint for Notifications](https://www.youtube.com/watch?v=k-_o0cCpksk&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=27)
3. [Write a React Page for Notifications](https://www.youtube.com/watch?v=k-_o0cCpksk&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=27)
4. [Run DynamoDB Local Container and Postgres container ensure both works](https://www.youtube.com/watch?v=CbQNMaa6zTg&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=28)

---

Loggedin View of Crudder :heavy_check_mark: 

![Logedin view](https://user-images.githubusercontent.com/40818088/222915083-1f21619e-2d55-46f1-a5ac-991ed0105870.PNG)

Posted a test Crud :heavy_check_mark: 

![Test crud](https://user-images.githubusercontent.com/40818088/222915103-e4f49135-27ad-4072-96e1-7f88ed4549b5.PNG)




