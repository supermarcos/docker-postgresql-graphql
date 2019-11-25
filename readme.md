# PostgreSQL + GraphQL Docker Containers

This repository has a very simple configuration for an instance of PostgreSQL and GraphQL services running in Docker Containers.

## Docker Configuration Concepts

## DB service - PostgreSQL

In `db` folder, `Dockerfile` defines the configuration for the `db` service.

- `FROM` defines the version of PostgreSQL to install
- `COPY` will copy files from the directory `init` into `docker-entrypoint-initdb.d` inside the container

In `db/init` folder, the SQL scripts are sorted to be executed in such order.

## GraphQL service

In `graphql` folder, `Dockerfile` defines the configuration for the `graphql` service.

- `FROM` defines the version of `node` to install
- `LABEL` is just a description for the node version to install
- `RUN` are to run scripts or commands. In this case to install (globally `-g` postgraphile and a plugin for the connection)
- `EXPOSE` the service at port `5000` in this case
- `ENTRYPOINT` are some args for the execution

## docker-compose.yml concepts

- `services` describe an ennumeration of the services by their names
- `container_name` is the name of the container
- `image` is the name of the image
- `build` is the building context for docker compose to build the custom image located in such context folder
  - `context` is the folder where the image should be built
- `volumes` is a mapping between the host folder and the docker one so, in this case, is where the data will be preserved so next time the changes in the data are still there
- `env_file` is the path to the file that contains all the environment variables such password, PostgreSQL url, etc...
- `networks` is the definition of a contextual virtual network used to connect two or more docker instances and different services that need to communicate to each other
- `ports` maps the docker container port and the host port to each other
- `command` is the command to be executed after the container starts, the list are the arguments and they must be provided as separated items

## Some Pointers

- ### Docker version

  Keep docker up to date, it may cause connection issues if it's too old

- ### Shared Drives

  When using Windows, allow your containers accessing your drives:

  - Right click on the docker icon by the system clock
  - Settings
  - On the settings window, go to Shared Drives
  - Grant your docker containers access to the drive you will have data shared between host and container by clicking on the checkbox by the drive in particular. i.e.: a PostgreSQL container may want to access a database, allow the container to access the drive where that data will be stored

- ### Careful with paths on Windows

  When using Windows, I noticed many issues trying to get the right path to work.

  ```[yml]
  volumes:
    - D:\Documents\experiments\dockers\postgresql\data:/db:rw
  ```

- ### SQL scripts are going to be executed in order

  And repeatedly, so if there are scripts that may crash if ran twice (with no fail-safe or try-catch wrap around), the crash will stop the rest of the scripts to be executed too.

## Handy Commands

- ### Build your containers

  Inside your project, where `docker-compose.yml` file is:

  ```[BASH]
  >_ docker-compose build
  ```

  Or, build and run:

  ```[BASH]
  >_ docker-compose up --build
  ```

  Or, build from scratch:

  ```[BASH]
  >_ docker-compose build --no-cache
  ```

- ### Start / run containers
  Inside your project, where `docker-compose.yml` file is:
  ```[BASH]
  >_ docker-compose up
  ```
  Or just one of the containers like such:
  ```[BASH]
  >_ docker-compose up db
  ```
  To run only the container with the database called `db` as defined in `docker-compose.yml`

## TESTING IF EVERYTHING WORKS

After running `docker-compose up` on your root directory of the project, you should have up (unless you have other errors popping up on your console) both services `db` and `graphql` instances working.

To check if the PostgreSQL is responding, using pgAdmin or similar, try to hit the given url (in this case will be localhost) with the user and password that are defined in `.env` file.

You may need to _add_ the server.

Then, you should be able to see the server, the database and the tables created.

To test if graphiql is responding, go to the url `http://localhost:5433/graphiql`.
You should be able to see the graphiql IDE, define a query on yur working space and run it to see on the results space if there are results.

For example, try this query:

```[JSON]
query {
  allParentTables {
    nodes {
      id
      name
      description
      chilTablesByParentTableId {
        nodes {
          id
          name
          description
        }
      }
    }
  }
}
```

And hit the _play_ button on the top tool bar. You should get a response like this:

```[JSON]
{
  "data": {
    "allParentTables": {
      "nodes": [
        {
          "id": 1,
          "name": "Parent name 1",
          "description": "Parent description 1",
          "childTablesByParentTableId": {
            "nodes": [
              {
                "id": 1,
                "name": "Child name 1",
                "description": "Child description 1"
              }
            ]
          }
        },
        {
          "id": 2,
          "name": "Parent name 2",
          "description": "Parent description 2",
          "childTablesByParentTableId": {
            "nodes": [
              {
                "id": 2,
                "name": "Child name 2",
                "description": "Child description 2"
              }
            ]
          }
        },
        {
          "id": 3,
          "name": "Parent name 3",
          "description": "Parent description 3",
          "childTablesByParentTableId": {
            "nodes": [
              {
                "id": 3,
                "name": "Child name 3",
                "description": "Child description 3"
              }
            ]
          }
        }
      ]
    }
  }
}
```
