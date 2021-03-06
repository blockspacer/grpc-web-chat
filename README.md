# gRPC Web Chat (Work in Progress)
A simple project demonstrating how both a Go and Java back end can power the same Vue.js front end using gRPC. 

* The server consists either of a Java or Go Lang project.
    * The Java project uses the Spring Boot framework with Maven for dependency management.
    * _TODO: Go Lang project..._
* The client is written in TypeScript and uses the [VueJS framework](https://github.com/vuejs/vue). It also uses [Vuetify](https://github.com/vuetifyjs/vuetify) and the [official gRPC-Web library](https://github.com/grpc/grpc-web).
* The [Envoy Proxy](https://github.com/envoyproxy/envoy) is used to translate requests and responses between the browser and the gRPC server. Check out [the state of gRPC in the browser](https://grpc.io/blog/state-of-grpc-web/).
    * TLS/SSL is used to allow HTTP2 connections between the browser and Envoy. This is to circumvent [browser connection limitations](https://docs.pushtechnology.com/cloud/latest/manual/html/designguide/solution/support/connection_limitations.html). More on this [here](https://github.com/grpc/grpc-web/issues/522).

## Overview

![gRPC-Web Overview](https://i.ibb.co/bWjrTzb/grpc-diagram.png)

## Project Branches
The root of the project contains the shared API and client code. Checkout the relevant server branch in your language of choice. 

```
master  -> dev-stable [-> dev]  -> go-dev-stable    [-> go-dev]
                                -> java-dev-stable  [-> java-dev]
```
## Installation for Development

### Requirements
##### Docker
1. Install Docker on your local machine.

##### Protoc
1. Go to [the releases page](https://github.com/protocolbuffers/protobuf/releases)  of [Protobuf](https://github.com/protocolbuffers/protobuf).
2. Select the latest release version.
3. Scroll down to `Assets` and download the applicable file (e.g., for Windows `protoc-3.8.0-rc-1-win64.zip`).
4. Extract the contents and add to your path (e.g., for Windows, simply add the `protoc.exe` to your path).

##### gRPC-Web Protoc Plugin
1. Go to [the releases page](https://github.com/grpc/grpc-web/releases)  of [gRPC-Web](https://github.com/grpc/grpc-web).
2. Select the latest release version.
3. Scroll down to `Assets` and download the applicable file (e.g., for Windows `protoc-gen-grpc-web-1.0.4-windows-x86_64.exe`).
4. Extract the `protoc-gen-grpc-web` file a directory and it to your path (e.g., for Windows add the `protoc-gen-grpc-web.exe` file to your path).

### Server

#### Java
1. Checkout the `java-dev-stable` branch.
2. Execute `mvn clean install` in the root directory to build and compile the project. 
    * This will generate all the necessary Protocol Buffer files for the server and client.
    * This will execute the `create-cert.sh` script, generating all necessary keys and certificates for the Vue development server and the Envoy Proxy.
3. To securely serve the Vue files on the Spring Boot server for production, execute the `create-cert-java.sh` script.
4. Simply run the `ChatApplication.java` as a normal Java application. This will start the server. See the `resources/application.properties` file for server configuration details.

#### Go
1. Checkout the `go-dev-stable` branch.
2. _TODO: Run gradle install/build script that builds files, generates protos, runs the create-cert.sh script, etc._ 
3. To serve the Vue files on the Go static file server for production, execute the `create-cert-go.sh` script.
4. _TODO: Run the Go server somehow..._

### Envoy Proxy
1. From the root directory, run the `create-ca-cert.sh` script to generate the Certificate Authority.
2. In the same directory, run the `create-cert.sh` script. See the `envoy/README.md` file for more details. 
    * **Note**: This step should be unnecessary for the Java project if the `mvn clean install` command was used.
3. Run `docker-compose up` from the root directory to start the Envoy proxy.
    * **Note**: Remember to remove the Docker image after generating new certificates!

### Client Development Server
1. Run `npm install` and then `npm run proto`.
    * **Note**: This step is unnecessary after running the `mvn clean install` command for the Java server.
2. Run `npm run serve` to start the development server on [https://localhost:443](https://localhost:443).

## Installation for Production
### Requirements
##### Docker
1. Install Docker on the Linux host machine. Follow the relevant instructions here:
    * [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
    * [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
    * [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
    * [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
2. Install Docker Compose by running the following two commands (see more on [Install Docker Compose](https://docs.docker.com/compose/install/)):
   ```
   sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```  

#### Java
1. From within the project root directory, run the `mvn clean install -U` command to build the Java project.
2. Copy the generated `.jar` file to a directory on the host machine.
3. Run the Java Server with `java -jar server-<version>.jar` to start the server.

#### Go
1. _TODO: Figure out how to run the Go server on production._

### Envoy Proxy
1. Modify the `envoy.yaml` file for production. Change the following:
    * Change the `domains` value to the host machine IP address (or `*` to allow all domains).
    * Change the `cors` `allow_origin` to the correct domain (e.g., https://example.com). You can also use `*` to allow all origins.
    * Change the `hosts` `socket address` to the machine IP address.
2. Copy the `envoy` directory to a directory on the host machine.
3. Copy the `docker-compose.yml` file one level up from the `envoy` directory copied in the previous step.
3. Run `docker-compose up` to build and deploy the Docker image.
    * **NOTE**: Remember to remove the docker image before running the `docker-compose` command if any changes to Envoy or the certificates were made.

### Client
1. If using a custom CA, add the relevant `ca.crt` file to the Trusted Root Certificate Authorities on your local machine.
2. The client should now be running on `https://<host-machine-ip>`.
