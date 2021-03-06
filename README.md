GEF
===

<img src="https://user-images.githubusercontent.com/17616266/30423134-3f594120-9942-11e7-9ce0-bf189aceb067.png" alt="official EUDAT logo" align="right" width="80" height="50">

The EUDAT Generic Execution Framework

[![Build Status](https://travis-ci.org/EUDAT-GEF/GEF.svg?branch=master)](https://travis-ci.org/EUDAT-GEF/GEF)


The GEF is a software developed in the frame of [EUDAT 2020](https://eudat.eu) as part of the [Horizon 2020](https://ec.europa.eu/programmes/horizon2020/) programme of the European Union. It enables execution of containerized software tools on data stored in the EUDAT [Collective Data Infrastructure](https://eudat.eu/eudat-collaborative-data-infrastructure-cdi) (CDI). The GEF employs the [Docker](https://docker.com) software containerization technology and uses specifically annotated Docker container images called GEF services for scientific data processing. By deploying these GEF services to Dockerized computational resources as close to the data as possible, data transfers can be minimized. GEF services can also be reused to reproduce results or toolchains. Data inputs to GEF services can be specified via [PID](http://www.pidconsortium.eu) or URL before they are automatically transferred to the processing location. Configuration of the GEF allows it to interface with Docker Server and Docker Swarm installations on various platforms. Users can interact with the GEF via a graphical interface (GUI) or an HTTP application programming interface (API). The GEF is using an SQLite database for internal data storage. Using a different SQL database is possible.

The software has grown to a beta release version that is capable of processing data stored in [B2SHARE](https://b2share.eudat.eu) and [B2DROP](https://www.eudat.eu/services/b2drop). Support for [B2SAFE](https://github.com/EUDAT-B2SAFE/B2SAFE-core) will follow in the remaining time of the project. The GEF is still in development and will only reach its full functionality at the end of the EUDAT 2020 project.

---

This documentation includes:

1. The Core Idea: GEF Services
   1. GEF Demo Services and Services for Internal Use
   2. GEF Service Metadata
2. The GEF Testing Instance
3. GEF Architecture
4. Installing the GEF
   1. GEF Server Installation and Deployment
   2. GEF Server Installation from Sources and Deployment
   3. Example Webserver Configuration (Apache Tomcat)
   4. GEF Configuration with `config.json`
   5. Packaging GEF Source Code for Deployment
5. GEF User Roles
   1. Superadministrators
   2. Community Administrators
   3. Community Members
   4. General Users
6. GEF User Interfaces
   1. HTTP API
   2. User Managment API
7. GEF User Workflows
   1. GEF Service Building Workflow
   2. GEF Service Enactment Workflow



## The Core Idea: GEF Services

GEF services are at the core of any scientific workflow supported by the GEF as they encapsulate the tools that are required to perform the data processing. GEF services are based on Docker images that provide a Linux-based transient workspace which can be perceived by users as an encapsulated lightweight Linux operating system running on a temporary file system that only exists while the GEF service is being executed. Communities are asked to familiarize themselves with the Docker technology in so far as to be able to build their own Docker images as Dockerfiles. Dockerfiles consist of the instruction set necessary for Docker image building. Scientific users who wish to employ the GEF are required to upload their Dockerfile to the GEF where the service will be built. The containerization of software tools makes them reusable and mobile since the same Docker image can be instantiated by the GEF at every computation location. Only community administrators are allowed to build new GEF services while community users are restricted to employing them. GEF services can be generated by community administrators by uploading their Dockerfiles through the web user-interface and triggering service building. More details on this can be found in the section GEF Service Building Workflow. The GEF then makes these services available at their deployment location. This will be augmented by a controlled GEF service repository based on DockerHub technology that will function as an exclusive storage for GEF services in the future. Such a repository will increase security by creating a closed system that makes all service images that can be enacted by the GEF completely transparent and open to scrutiny.



### GEF Demo Services and Services for Internal Use

In the root of the repository there is a folder called `GEF/services` which contains several GEF demo services. They are not essential for the system and therefore are not available by default (after the installation process). However, they can help to better understand how to build your own GEF services and how they work. If you want to use those services, you can build them through the web interface (by uploading the corresponding Dockerfiles and image-related files found in the dedicated subdirectories, see GEF Service Building Workflow). The `GEF/services` directory also contains a folder `_internal` which holds several Docker images for internal use by the GEF system. When the GEF connects to a Docker server it builds several custom images that are necessary for the system to function properly. Please do not remove them.

### GEF Service Metadata

Every Docker image needs to be labeled in a specific way to function as a GEF service. Labelling images fills certain metadata fields attached to a Docker image and is currently done by the GEF development team as the number of GEF services is still manageable, but will also become a responsibility of our scientific communities in the future as the number of GEF services grows. For the upcoming beta release of the GEF, so far we have opted for a collection of labels that we will illustrate with the Stanford Parser example service found on the testing instance. We aim for a standardisation of metadata fields until the end of the project. These are the label instructions that annotate the Stanford Parser GEF service taken from the corresponding Dockerfile in `GEF/services/stanford-parser`:

~~~~
LABEL "eudat.gef.service.name"="Stanford Parser for English"
LABEL "eudat.gef.service.description"="Parses a given text and produces constituency and dependency trees for each sentence"
LABEL "eudat.gef.service.version"="1.0"
LABEL "eudat.gef.service.input.1.name"="Input Directory"
LABEL "eudat.gef.service.input.1.path"="/root/input"
LABEL "eudat.gef.service.output.1.name"="Output Directory"
LABEL "eudat.gef.service.output.1.path"="/root/output"
~~~~

The GEF Testing Instance
--------------
Apart from the code made available here on Github, we have set up a GEF testing instance on the VMWare cluster of Gesellschaft für wissenschaftliche Datenverarbeitung in Göttingen (GWDG) to showcase the GEF's functionality. You can visit it at https://eudat-gef.mpimet.mpg.de where you will find a preinstalled GEF server. Use your browser to test a few example services that we have provided for this purpose. More example services will be added as they become available. You will not be able to build new services on the testing instance, but you can try out the existing ones. Please also note that the GEF testing instance requires B2ACCESS user authentication if you wish to run a GEF service. It currently relies on the B2ACCESS development instance instead of the official B2ACCESS instance which requires users to create separate accounts on the development instance.

GEF Architecture
----------------

This section includes a description of the main GEF components and how they interact with each other, and a brief overview of the most important technologies involved. The basic architecture of the GEF is comprised of a frontend and a backend. The GEF frontend consists of a web user interface written in JavaScript conforming to the ECMAScript 6 specification and uses the [React](https://facebook.github.io/react/) library for building the user interface in combination with [Redux](http://redux.js.org/) for state management and [Bootstrap](https://react-bootstrap.github.io/) user interface design templates. The GEF backend is written in Go and functions as page server to the web user interface and as client to a Docker Swarm represented by a manager node, interfacing with it through the Docker HTTP/HTTPS Remote API. Thus, the backend relies on the [Docker](https://www.docker.com/) application containerization technology for data processing. Within the swarm, one or more manager nodes are responsible for distributing tasks to the worker nodes. A swarm may consist of a single manager node that functions as a worker node and can be scaled up, but no upper limit to the swarm size is documented. The GEF was designed in a modular way allowing the GEF backend to be exchanged if different (containerization) technologies are to be employed. The prerequisite for any processing of data is a GEF service provided by the respective scientific community. This service needs to contain the installed software tools required for the desired computation. The GEF can enact such services close to the data storage location if the GEF deployment node where computation takes place is Docker-enabled. The following diagram illustrates the different GEF components and how they interact with each other:

![alt text](https://user-images.githubusercontent.com/17616266/30423128-3a142efa-9942-11e7-83aa-cb14bbb3259d.png)

Please note that the GEF frontend, GEF backend, and the Docker Swarm manager node to which the GEF backend is a client interact via HTTP/HTTPS. This allows colocation of all components as well as the distributed deployment depicted here. Also note that although Docker Swarm was chosen in this diagram, the GEF backend can still operate with a single Docker Server without having to rely on the swarm mode. The GEF service repository shown in a subdued tone is a future component that is still being conceptualized.

Installing the GEF
-----------------

If you want to install the GEF yourself and build your own services, either use the 64-bit Linux binary we provide or follow the installation instructions below. This will allow you to build your own GEF service with a Dockerfile and execute it. Please refer to the original Docker [documentation](https://docs.docker.com/engine/reference/builder/) for more information on Dockerfiles. Also be aware of the best practices published [here](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/). Building GEF services from Dockerfiles is explained below.


### GEF Server Installation and Deployment

Follow these instructions to set up your own GEF server instance with a GEF 64-bit Linux binary:

- Download the latest GEF 64-bit Linux binary archive file from [here](https://github.com/EUDAT-GEF/GEF/releases)
- Unpack the tarball to a dedicated folder and set it as the current directory
- Inspect and change the default server configuration in the  `./config.json` (see section on `config.json`)
- Create a new self-signed certificate for the GEF server (with `make certificate`) or edit `config.json` to use your own (see section on `config.json`)
- Define the `GEF_SECRET_KEY` environment variable (a random string is preferred, remember it and keep it in a safe location), e.g. with `export GEF_SECRET_KEY="E60su8IL6VY6Ca2"`
- Define the `GEF_B2ACCESS_CONSUMER_KEY` and `GEF_B2ACCESS_SECRET_KEY` environment variables for connection to the B2ACCESS service. Since the development instance of B2ACCESS is used, random keys can be used here.
- Run `./gef_linux` to start the GEF.
- The GEF Server is now running and the user interface can be reached on default port 8443. Go to `https://localhost:8443` to see whether the GEF user interface is online.
- Adjust your webserver configuration to map this port to the HTTPS port 443 to make the GEF visible from the outside.



### GEF Server Installation from Sources and Deployment

Follow these instructions to build the GEF from sources and deploy a GEF server:

- Make sure you have the following tools installed on your machine:
    - the [`go 1.8`](https://golang.org) language tools (for MacOS it is recommended to have at least `go 1.8.1`)
    - the [`glide`](https://github.com/Masterminds/glide) vendor package management tool for Go
    - the [`docker`](https://www.docker.com) containerization software
    - the [`npm`](https://www.npmjs.com) package manager for JavaScript

- Set a GOPATH, e.g. with `export GOPATH=/Users/myself/Projects/Go`.
- Use `go get` to clone the GEF repository with `go get -u github.com/EUDAT-GEF/GEF`.
- Go to the downloaded repository location with `cd $GOPATH/src/github.com/EUDAT-GEF/GEF`.
- Build the project with `make build`.
- Create a new self-signed certificate for the GEF server (with `make certificate`) or edit `config.json` to use your own (see section on `config.json`)
- Define the `GEF_SECRET_KEY` environment variable (a random string is preferred, remember it and keep it in a safe location), e.g. with `export GEF_SECRET_KEY="E60su8IL6VY6Ca2"`
- Define the `GEF_B2ACCESS_CONSUMER_KEY` and `GEF_B2ACCESS_SECRET_KEY` environment variables for connection to the B2ACCESS service. Since the development instance of B2ACCESS is used, random keys can be used here.
- Start the GEF with `make run_gef`.
- The GEF Server is now running and the user interface can be reached on default port 8443. Go to `https://localhost:8443` to see whether the GEF user interface is online.
- Adjust your webserver configuration to map this port to the HTTPS port 443 to make the GEF visible from the outside.

### Example Webserver Configuration (Apache Tomcat)

We give an example configuration of an Apache webserver used in development. If you are using Apache, you may want to use a configuration similar to the one below. Note that the webserver is used as a proxy for mapping the default GEF port 8443 to the HTTPS port 443.

~~~~
Listen 443
NameVirtualHost *:443
<VirtualHost *:443>
    SSLEngine on
    SSLProxyEngine on
    SSLProxyVerify none
    SSLProxyCheckPeerCN off
    SSLProxyCheckPeerName off
    SSLProxyCheckPeerExpire off
    ProxyRequests off
    SSLCertificateFile /etc/apache2/ssl/ssl.crt
    SSLCertificateKeyFile /etc/apache2/ssl/ssl.key
    ProxyPreserveHost On
    ProxyPass / https://127.0.0.1:8443/
    ProxyPassReverse / https://127.0.0.1:8443/
</VirtualHost>
~~~~

### GEF Configuration with `config.json`

The GEF can be configured by editing the `config.json` file found in the `GEF/gefserver` directory of your installation. In the file six thematic sections of key/value pairs can be found. They are `Docker`, `Pier`, `Server`, `EventSystem`, `Limits` and `Timeouts`. The `Server` section has three more subsection named `B2DROP`, `B2ACCESS`and `Administration`. The file offers the following settings:

#### `Docker` Section

Key name | Default value |Description
---------|---------------|-----------
Endpoint | unix:///var/run/docker.sock | Defines the Docker daemon location. The default sock indicates that the local machine is used.
TLSVerify (optional) | no default bool | Sets whether the Docker client checks TLS certiifcates.
CertPath (optional) | no default string | Path to client certificate.
KeyPath (optional) | no default string | Path to client key.
CAPath (optional) | no default string | Path to Certificate Authority (CA) root certificate.

See [here](https://docs.docker.com/engine/security/certificates/) to familiarise yourself with the way Docker employs client certificates.

#### `Pier` Section

Key name | Default value |Description
---------|---------------|-----------
InternalServicesFolder | ../services/_internal | Directory containing the GEF internal services’ content (Dockerfiles and corresponding files). The GEF has several internal services that are built while the system starts, if the images do not already exist (e.g. data staging, volume inspection, data download from a volume)

#### `Server` Section

Key name | Default value |Description
---------|---------------|-----------
Address | :8443 | Port address the GEF is using.
ReadTimeoutsSecs | 10 | Read timeout for the server (in seconds).
WriteTimeoutsSec | 10 | Write timeout for the server (in seconds).
TLSCertificateFilePath | ../ssl/server.crt | Path to a TLS certificate.
TLSKeyFilePath | ../ssl/server.key | Path to a TLS key.

#### `B2ACCESS` Section

Key name | Default value |Description
---------|---------------|-----------
BaseURL | https://unity.eudat-aai.fz-juelich.de | URL to B2Access instance used for authentication. By default for the GEF beta version, this is the B2ACCESS development instance, not the official one.
RedirectURL | https://localhost:8443/wui/b2access/ | GEF URL from which the user will be redirected to the B2Access authentication form.

#### `B2DROP` Section

Key name | Default value |Description
---------|---------------|-----------
BaseURL | https://b2drop.eudat.eu/ | URL to B2DROP instance to be used.

#### `Administration` Section

Key name | Default value |Description
---------|---------------|-----------
SuperAdminEmail | email@example.com | Email address of the super administrator for the GEF (put here the email you used during the registration at the B2Access development instance to become a super administrator).
ContactLink | https://www.eudat.eu/support-request?service=Other | URL of the contact link at the bottom of the page. Points to the official EUDAT contact request form.

#### `EventSystem` Section

Key name | Default value |Description
---------|---------------|-----------
Address | no default value | Address of the event system (rule engine system). By default the event system is off and the attribute is empty. The event system is still experimental.

#### `Limits` Section

Key name | Default value |Description
---------|---------------|-----------
CpuShares | 1024 | CPU shares (relative weight vs. other containers).
CpuPeriod | 100000 | The CPU period to be used for hardcapping (in microseconds). Set to 0 (zero) to use system default, which is 100 milliseconds.
CpuQuota | 50000 | The CPU hardcap limit (in microseconds). Allowed CPU time in a given period (e.g. set this value to 50000 to limit the container to 50% of a CPU resource).
Memory | 400024000 | Memory (in bytes) available for a container.
MemorySwap | 450024000 | Memory swap (in bytes) available for a container.

#### `Timeouts` Section

Key name | Default value |Description
---------|---------------|-----------
DataStaging | 1000 | Timeout for the data staging container (in seconds).
VolumeInspection | 1000 | Timeout for the volume inspection container (in seconds).
FileDownload | 1000 | Timeout for the file download container (in seconds).
Preparation | 100 | Container creation timeout (in seconds).
JobExecution | 7200 | Job execution timeout (in seconds).
CheckInterval | 10 | Timeouts are checked on a timer, here you can set its interval (in seconds).




### Packaging GEF Source Code For Deployment

For packaging the GEF source code into a binary you will only need to have the GEF source code and `docker` installed on your machine. After the GEF source code is modified you can build a 64-bit Linux binary
by running the `make pack` command. The command will create an archive which you can unpack and run it on a server as described in the installation instructions above. After that do `cd build/bin` and run `./gefserver`. The GEF server should be running now.

GEF User Roles
--------------

Four user groups exist that represent different levels of trust granted to users. The most trusted users are GEF Superadministrators, followed by Community Administrators and Community Users. All of these require a B2ACCESS user account to log in and access the GEF. The remaining group is the general user who was not authenticated through B2ACCESS and therefore also cannot log in as the user's trustworthiness cannot be ascertained.

### Superadministrators

GEF Superadministrators are designated by setting the B2ACCESS email address in the `Administration` section of the `config.json` file. They have the right to assign user roles to all other logged in users through a designated page on the GUI or via web API. The dropdown menu of your user icon on the top right reveals the 'Manage Roles' page which allows the Superadmins to assign users identified by their B2ACCESS email addresses to user groups. Superadmins can also create GEF services, run jobs and view the job history.

### Community Administrators

GEF Community Administrators are meant to be designated by a scientific community to be the owners of its GEF services. Community Admins therefore can create new GEF services, run jobs and view the job history. Users who are to become Community Admins need to be assigned this role by a GEF Superadmin.

### Community Members

GEF Community Members are all members of a scientific community who want to employ GEF services. They can run jobs from existing services and view the job history. They cannot create new GEF services. Users who wish to become Community Members need a B2ACCESS account.

### General Users

General Users have not been authenticated with B2ACCESS and are therefore unable to log in. They can only view the job history, but neither run jobs nor create new GEF services.

GEF User Interface
------------------

The GEF provides a browser-based GUI that individual researchers can use to execute computation jobs with GEF services or that administrators may employ to assemble GEF services. The browser interface and the HTTP API were designed to be functionally similar, but both interfaces may still change as GEF development continues. The browser interface allows navigation to pages for building a GEF service, browsing and inspecting all locally stored GEF services and their metadata, and browsing and inspecting all finished and running computation jobs, as well as viewing their input and output volumes. The HTTP API is designed to expose GEF functionality as a web service. The accessible functionality includes requests for creating GEF services, for running computation jobs, and for querying information about volumes and downloading computation results from specific data volumes. Several more API functions exist for user management.

### GEF HTTP API

| URL | Method | Input | Output | Description |
| ---: |:-------- | :------ | :------- | :------ |
| /api/info | GET |  | API version information in JSON | Information about API (welcome page), can be used to check if backend is running |
| /api/builds | POST |  | JSON object with information about the location and build ID | Creates a temporary folder when an image has to be created. It returns a buildID identifier and a folder location. This folder is used to store a Dockerfile and files needed for the image. BuildID is a string like a UID in Java (which is generated when required and it is unique) |
| /api/builds/{buildID} | POST | {buildID} build identifier and a multipart request body containing files (Dockerfile, files that have to be in the image) | JSON object with information about the image and the corresponding service | Builds an image provided a buildID (which points to the folder with the Dockerfile), returns JSON with the information about the image and the new service (partly taken from the metadata) |
| /api/services | GET |  | JSON with the list of all services | Lists all available services  |
| /api/services/{serviceID} | GET | {serviceID} an id of a service | JSON with information about a specific service | Returns information about a specific service |
| /api/services/{serviceID} | PUT | {serviceID} an id of a service, form data with new service metadata | JSON with information about a specific service | Modifies metadata of a specific service |
| /api/services/{serviceID} | DELETE | {serviceID} an id of a service | JSON with service information | Deletes a specific job |
| /api/jobs | POST | serviceID and pid | JSON object with information about the location and jobID | Executes a job. We submit a form to this URL when we want to run a job. PID is resolved, files are downloaded and saved to an input volume |
| /api/jobs | GET |  | JSON with the list of jobs | Lists available jobs |
| /api/jobs/{jobID} | GET | {jobID} id of a job | JSON with job information | Information about a specific job |
| /api/jobs/{jobID} | DELETE | {jobID} id of a job | JSON with job information | Deletes a specific job |
| /api/volumes/{volumeID}/{path:.*} | GET | {volumeID} is an id of a volume, {path} is a path inside this volume (root folder by default) | JSON object (nested) with the list of the files and folders in a given volume | Lists all files and folders (recursively) in a given volume |

### User Management API

| URL | Method | Input | Output | Description |
| ---: |:-------- | :------ | :------- | :------ |
| /api/user | GET |  | JSON with the information about the current user | Returns information about the current user |
| /api/user/tokens | POST | Form data with the name of a token {tokenName} | JSON with the new token | Adds a new token for the current user |
| /api/user/tokens | GET |  | JSON with the list of all user tokens | List all tokens for the current user |
| /api//user/tokens/{tokenID} | DELETE | {tokenID} an id of a token | Server response code | Removes a specific token from the current user |
| /api/roles | GET |  | JSON with the list of all roles | Lists all available roles |
| /api/roles/{roleID} | GET | {roleID} an id of a role | JSON with the list of users | Returns a list of users to which a certain role was assigned |
| /api/roles/{roleID} | POST | {roleID} an id of a role | Server response code | Assigns a specific role to the current user |
| /api/roles/{roleID}/{userID} | DELETE | {roleID} an id of a role assigned to the user with the user id {userID} | Server response code | Removes a role from a user |

GEF User Workflows
------------------------

The GEF and its Docker backend are currently designed to serve two basic user workflows. Together these workflows are sufficient for initiating data processing with the GEF on a given data set and retrieving the results. The first user workflow is meant to be the prerogative of community administrators and results in a new GEF service from the respective community. It is called the Service Building Workflow. The second workflow uses a given GEF service to perform a computation on input data specified by a community user. This one is called Service Enactment Workflow. Although service building is a precondition for service enactment, these two workflows do not need to be implemented in conjunction if the GEF service to be enacted already exists. Users can choose to interface with the GEF via the browser user interface or via the HTTP API to implement these workflows. Please note that user groups are not implemented yet and the GEF makes no distinction between users. The following describes these user workflows step by step.

### GEF Service Building Workflow

The GEF employs a description of the file, the environment, and the commands a Docker image is composed of to build GEF services. This description is called a Dockerfile. To build a GEF service from a given Dockerfile, a Community Administrator or Superadministrator needs to perform the following workflow steps:

1. Create a Dockerfile from which Docker can build an image that serves as a GEF service. This requires familiarity with Dockerfiles and their function within the Docker system. It also requires knowledge of the scientific tools employed within the community and how to install them. Metadata labels for the image that qualify it as GEF service have to be specified in this Dockerfile as well.

2. Upload the Dockerfile and other necessary files to the GEF.
3. Trigger building the new GEF service.

After the GEF service has been built successfully, it is available as a service to be enacted.

### GEF Service Enactment Workflow

In order to enact a GEF service on a given data set, a Community Administrator or a Community User needs to perform the following workflow steps:

1. Select the GEF service to be enacted.
2. Submit the PID/URL of the input data set, automatically triggering the computation in the form of a job.
3. Inspect the console or the output volume and download the results if the job completed the computation successfully.

The GEF does not yet allow chaining of individual jobs. Later GEF versions aim to support mounting an output volume of one job to be the input volume of another to orchestrate a sequence of computation jobs.
