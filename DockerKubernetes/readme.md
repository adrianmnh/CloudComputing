# **`Docker Containerization and Kubernetes Cluster`**

## **`Volumes`**
| |Named Volumes|Bind Mounts|
|:----|:----|:----|
|Host Location|Docker chooses|You control|
|Mount Example (using -v)|my-volume:/usr/local/data|/path/to/data:/usr/local/data|
|Populates new volume with container contents|Yes|No|
|Supports Volume Drivers|Yes|No|

    docker run `
    -dp 1234:3000 `
    -w /app -v "$(pwd):/app" `
    todo-app:version `
    sh -c "yarn install && yarn run dev"

`-dp 1234:3000` → detached(background mode) and port routing: traffic from container's port 1234 to the application's, listening on port 3000

`-w /app` → working dir: directory command will execute from

`-v "$(pwd):/app"` → volume: bind-mount host's current dir ⇒ image /app dir

`node:12-alpine` → base image for application

`sh -c "yarn install && yarn run dev"` → alpine doesn't have bash, so we use the *base shell* **sh** to install all dependencies(npm install) and run the *app-start* script(npm run.exe dev). **dev** starts **nodemon**

<sub>**nodemon** is a node.js based tool that automatically relaunches apps upon changes

## **`Container Networking`**

Allows isolated containers running of the same network to talk to each other. Different network containers *cannot* communicate

### **Change base SQLite to MySQL**

### **Start MySQL**
1. Assign at start 

or

2. Connect an existing container

<sub>Create network → **attach MySQL at start**

`docker network create todo-app`

    docker run -d `
     --network todo-app  `
     --network-alias mysql `
     -v todo-mysql-data:/var/lib/mysql `
     -e MYSQL_ROOT_PASSWORD=secret `
     -e MYSQL_DATABASE=todos `
     mysql:5.7

**Create MySQL Container:**

    docker run --detach `
    --network todo-app `
    --network-alias mysql `
    --volume todo-mysql-data:/var/lib/mysql `
    --env MYSQL_ROOT_PASSWORD=secret `
    --env MYSQL_DATABASE=todos `
    mysql:5.7

### **Connect to database and enter password _secret_**

    docker exec -it container_id mysql -u root -p

    SHOW DATABASES;

### **Connect to MySQL**

Usefull tools for troubleshooting and debugging network issues

    docker run --interactive --tty --network todo-app nicolaka/netshoot

    dig mysql

**`A record`** for **mysql**(network alias) resolves to an IP address. Application can communicate with MySQL database container by simply connecting to host _named_ **mysql**

### **Run app with MySQL**

This app supports setting of a few env variables for MySQL connection settings

**MYSQL_HOST** → host running MySQL server

**MYSQL_USER**

**MYSQL_PASSWORD**

**MYSQL_DB** → database to use once connected

or 

**MYSQL_PASSWORD_FILE** → for a secure way to reference secrets from the container itself, avoiding the use of environment variables, which can be leaked

<sub>MySQL version8.0 and higher must include:</sub>

    ALTER USER 'root'
    IDENTIFIED WITH mysql_native_password 
    BY 'secret';
    flush privileges
<sub>**docker logs container_id** → information about container </sub>

    docker run -dp 1234:3000 `
    -w /app -v "$(pwd):/app" `
    --network todo-app `
    -e MYSQL_HOST=mysql `
    -e MYSQL_USER=root `
    -e MYSQL_PASSWORD=secret `
    -e MYSQL_DB=todos `
    todo-list:version1.2 `
    sh -c "yarn install && yarn run dev"

 **Check database is working properly:**

    docker exec --interactive --tty `
    <sql-container-id> mysql -p todos

    SELECT * FROM todos.todos_items;

