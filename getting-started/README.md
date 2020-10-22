# Getting Started

## Setting up the Development Environment

The quickest way to get started is by running PostgreSQL in a Docker Container.

1. Install [Docker](https://docs.docker.com/get-docker/) and [Docker-Compose](https://docs.docker.com/compose/install/)

1. Run `docker-compose up-d`

You can always stop your Development Environment by entering `docker-compose down`.

## Connecting to PostgreSQL with pgAdmin 4

1. Visit [http://localhost:8080/](http://localhost:8080/) in your Webbrowser to open pgAdmin4.
    ![Screenshot pgAdmin Login](pgAdmin_login.png)

1. Login with `name@host.com` as **E-Mail** and `badsecret` as **Password**.

1. Right-Click on **Servers** and create a new Server.
    ![Screenshot pgAdmin Add Server](pgAdmin_add_server.png)

1. In the General tab, enter a **Server Name** e.g. `dockerized-postgres`.
    ![Screenshot pgAdmin Server General tab](pgAdmin_server_general_tab.png)

1. In the Connection tab, enter `postgres-server` as **Host**, `5432` as **Port**, `postgres` as **Maintenance database**, `admin` as **Username** and `bad secret` as **Password**. Check **Save Password?** and click on **Save**.
    ![Screenshot pgAdmin Server Connection tab](pgAdmin_server_connection_tab.png)

1. PgAdmin should now be successfully connected.
    ![Screenshot pgAdmin Servers Overview](pgAdmin_servers_overview.png)

## Connecting to PostgreSQL with psql (PostgreSQL interactive terminal)

1. Open shell in running Database Container

    ```shell
    docker-compose run db bash
    ```

1. Connect to the Database

    ```shell
    psql --host=db --username=${POSTGRES_USER} --dbname=${POSTGRES_DB}
    >> Password for user admin:
    ```

    Instead of using the environment variables defined in `docker-compose.yml`, we could manually enter `admin` as **username** and `mydatabase` as **dbname**.

    Next, enter your **password** for admin user.

1. psql should now be successfully connected.
Enter `\l` to list all databases.
