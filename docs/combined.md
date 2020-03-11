# Welcome to the docker-splunk documentation!

Welcome to Splunk's official documentation regarding Dockerfiles for building Splunk Enterprise and Universal Forwarder images using containerization technology. This repository supports all Splunk roles and deployment topologies, and currently works on any Linux-based platform. 

The provisioning of these disjoint containers is handled by the [splunk-ansible](https://github.com/splunk/splunk-ansible) project. Please refer to [Ansible documentation](http://docs.ansible.com/) for more details about Ansible concepts and how it works. 

#### What is Splunk Enterprise?
Splunk Enterprise is a platform for operational intelligence. Our software lets you collect, analyze, and act upon the untapped value of big data that your technology infrastructure, security systems, and business applications generate. It gives you insights to drive operational performance and business results.

Please refer to [Splunk products](https://www.splunk.com/en_us/software.html) for more knowledge about the features and capabilities of Splunk, and how you can bring it into your organization.

#### What is docker-splunk?
This is the official source code repository for building Docker images of Splunk Enterprise and Splunk Universal Forwarder. By introducing containerization, we can marry the ideals of infrastructure-as-code and declarative directives to manage and run Splunk and its other product offerings.

This repository should be used by people interested in running Splunk in their container orchestration environments. With this Docker image, we support running a standalone development Splunk instance as easily as running a full-fledged distributed production cluster, all while maintaining the best practices and recommended standards of operating Splunk at scale.

---

#### Table of Contents

* [Introduction](INTRODUCTION.md)
* [Getting Started](SETUP.md)
    * [Install](SETUP.md#install)
    * [Configure](SETUP.md#configure)
    * [Run](SETUP.md#run)
* [Examples](EXAMPLES.md)
* [Advanced Usage](ADVANCED.md)
    * [Runtime configuration](ADVANCED.md#runtime-configuration)
    * [Entrypoint Functions](ADVANCED.md#entrypoint-functions)
    * [Install apps](ADVANCED.md#install-apps)
    * [Apply Splunk license](ADVANCED.md#apply-splunk-license)
    * [Enabling SmartStore](ADVANCED.md#enabling-smartstore)
    * [Deploy distributed topology](ADVANCED.md#deploy-distributed-topology)
    * [Build from source](ADVANCED.md#build-from-source)
* [Persistent Storage](STORAGE_OPTIONS.md)
* [Architecture](ARCHITECTURE.md)
* [Troubleshooting](TROUBLESHOOTING.md)
* [Contributing](CONTRIBUTING.md)
* [Support](SUPPORT.md)
* [License](LICENSE.md)
## The Need for Containers
Splunk Enterprise is most commonly deployed with dedicated hardware, and in configurations to support the size of your organization. Expanding your Splunk Enterprise service using only dedicated hardware involves procuring new hardware, installing the operating system, installing and then configuring Splunk Enterprise. Expanding to meet the needs of your users rapidly becomes difficult and overly complex in this model. 

The overhead of this operation normally leads people down the path of creating virtual machines using a hypervisor. A hypervisor provides a significant improvement to the speed of spinning up more compute resources, but comes with one major drawback: the overhead of running multiple operating systems on one host.

<img src="container-vm-whatcontainer_2.png" width="370"/>

## The Advent of Docker
In recent years, [Docker](https://www.docker.com) has become the de-facto tool designed make it easier to create, deploy, and run applications through the use of containers.

Containers allow an application to be the only process that runs in a VM-like, isolated environment. Unlike a hypervisor, a container-based system does not require the use of a guest operating system. This allows a single host to dedicate more resources towards the application. 

For more information on how containers or Docker works, we'll let [Docker do the talking](https://www.docker.com/resources/what-container).

<img src="docker-containerized-appliction-blue-border_2.png" width="370"/>

The Splunk user community has asked us to support containerization as a platform for running Splunk. The promise of running applications in a microservice-oriented architecture evangelizes the principles of infrastructure-as-code and declarative directives, and we aimed to bring those benefits with the work in this codebase. This project delivers on that request: to provide the rich functionality that Splunk Enterprise offers with the user-friendliness and production-readiness of container-native software.

## History
In 2015, Denis Gladkikh (@outcoldman) created an open-source GitHub repository for installing Splunk Enterprise, Splunk Universal Forwarder and Splunk Light inside containers.

Universal Forwarders and standalone instances were being brought online at a rapid pace, which introduced a new level of complexity into the enterprise environment. In 2018, a new container image was created to improve the flexibility with which Splunk Enterprise could be operated in larger and more dynamic environments. Splunk's new container can now start with a small environment and grow with the deployment. This however has caused a divergence from the open-source community edition of the Splunk Enterprise container. 

As a result, containers for Splunk Enterprise versions prior to 7.1 can not be used with, or in conjunction with, this new version as it is not backward compatible. We are also unable to support version updates from any prior container to the current version released with Splunk Enterprise and Splunk Universal Forwarder 7.2, as the older versions are not forward compatible. We are sorry for any inconvenience this may cause. 
## Navigation

* [Install](#install)
* [Configure](#configure)
* [Run](#run)
    * [Splunk Enterprise](#splunk-enterprise)
    * [Splunk Universal Forwarder](#splunk-universal-forwarder)
* [Summary](#summary)

## Install
In order to run this Docker image, you will need the following prerequisites and dependencies installed on each node you plan on deploying the container:
1. Linux-based operating system (Debian, CentOS, etc.)
2. Chipset: 
    * `splunk/splunk` image supports x86-64 chipsets
    * `splunk/universalforwarder` image supports both x86-64 and s390x chipsets
3. Kernel version > 4.0
4. Docker engine
    * Docker Enterprise Engine 17.06.2 or later
    * Docker Community Engine 17.06.2 or later
5. `overlay2` Docker daemon storage driver
    * Create a file /etc/docker/daemon.json on Linux systems, or C:\ProgramData\docker\config\daemon.json on Windows systems. Add {"storage-driver": "overlay2"} to the daemon.json. If you already have an existing json, please only add "storage-driver": "overlay2" as a key, value pair.

For more details, please see the official [supported architectures and platforms for containerized Splunk environments](https://docs.splunk.com/Documentation/Splunk/latest/Installation/Systemrequirements#Containerized_computing_platforms) as well as [hardware and capacity recommendations](https://docs.splunk.com/Documentation/Splunk/latest/Installation/Systemrequirements). 

## Configure
Before we can run the containers, we should pull it down from DockerHub. Run the following commands to pull the images into your local environment:
```
$ docker pull splunk/splunk:latest
$ docker pull splunk/universalforwarder:latest
```

## Run
Before we stand up any containers, let's first create a network to enable communication between each of the services. This step is not necessary if you only wish to create a single, isolated instance.
```
$ docker network create --driver bridge --attachable skynet
```

##### Splunk Enterprise
Use the following command to start a single standalone instance of Splunk Enterprise:
```
$ docker run --network skynet --name so1 --hostname so1 -p 8000:8000 -e "SPLUNK_PASSWORD=<password>" -e "SPLUNK_START_ARGS=--accept-license" -it splunk/splunk:latest
```

Let's break down what this command does:
1. Start a Docker container using the `splunk/splunk:latest` image
2. Launch the container in the formerly-created bridge network `skynet`
3. Name the container + hostname as `so1`
4. Expose a port mapping from the host's `8000` to the container's `8000`
5. Specify a custom `SPLUNK_PASSWORD` - be sure to replace `<password>` with any string that conforms to the [Splunk Enterprise password requirements](https://docs.splunk.com/Documentation/Splunk/latest/Security/Configurepasswordsinspecfile)
6. Accept the license agreement with `SPLUNK_START_ARGS=--accept-license` - this must be explicitly accepted on every container, otherwise Splunk will not start

After the container starts up successfully, you should be able to access SplunkWeb at http://localhost:8000 with `admin:<password>`.

##### Splunk Universal Forwarder
Use the following command to start a single standalone instance of Splunk Universal Forwarder:
```
$ docker run --network skynet --name uf1 --hostname uf1 -e "SPLUNK_PASSWORD=<password>" -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_STANDALONE_URL=so1" -it splunk/universalforwarder:latest
```

Now let's run the same analysis on what we just did:
1. Start a Docker container using the `splunk/universalforwarder:latest` image
2. Launch the container in the formerly-created bridge network `skynet`
3. Name the container + hostname as `uf1`
4. Specify a custom `SPLUNK_PASSWORD` - be sure to replace `<password>` with any string that conforms to the [Splunk Enterprise password requirements](https://docs.splunk.com/Documentation/Splunk/latest/Security/Configurepasswordsinspecfile)
5. Accept the license agreement with `SPLUNK_START_ARGS=--accept-license` - this must be explicitly accepted on every container, otherwise Splunk will not start
6. Connect it to the standalone created earlier to automatically send logs to `so1`

**NOTE:** The Splunk Universal Forwarder product does not have a web interface - if you require access to the Splunk installation in this particular container, please refer to the [REST API](https://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTprolog) or use `docker exec` to access the [Splunk CLI](https://docs.splunk.com/Documentation/Splunk/latest/Admin/CLIadmincommands).

## Summary
You've successfully used `docker-splunk`! 

If everything went smoothly, you can login to the standalone Splunk with your browser pointed at `http://localhost:8000`, then run a search to confirms the logs are available. For example, a query such as `index=_internal` should return all the internal Splunk logs for both `host=so1` and `host=uf1`.

Ready for more? Now that your feet are wet, go learn more about the [design and architecture](ARCHITECTURE.md) or run through more [complex scenarios](ADVANCED.md).
## Changelog

## Navigation

* [8.0.2](#802)
* [8.0.1](#801)
* [8.0.0](#800)
* [7.3.4](#734)
* [7.3.3](#733)
* [7.3.2](#732)
* [7.3.1](#731)
* [7.3.0](#730)
* [7.2.9](#729)
* [7.2.8](#728)
* [7.2.7](#727)
* [7.2.6](#726)
* [7.2.5.1](#7251)
* [7.2.5](#725)
* [7.2.4](#724)
* [7.2.3](#723)
* [7.2.2](#722)
* [7.2.1](#721)
* [7.2.0](#720)

---

## 8.0.2

#### What's New?
* New Splunk Enterprise release of 8.0.2

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/8.0.2/ReleaseNotes/Fixedissues
* Bugfixes and increasing test coverage for new features

#### splunk-ansible changes:
* * Revised Splunk forwarding/receiving plays to optionally support SSL (see documentation on [securing data from forwarders](https://docs.splunk.com/Documentation/Splunk/latest/Security/Aboutsecuringdatafromforwarders))
* Initial support for forwarder management using [Splunk Monitoring Console](https://docs.splunk.com/Documentation/Splunk/latest/DMC/DMCoverview)
* New environment variables exposed to control replication/search factor for clusters, key/value pairs written to `splunk-launch.conf`, and replacing default security key (pass4SymmKey)

**NOTE** Changes made to support new features may break backwards-compatibility with former versions of the `default.yml` schema. This was deemed necessary for maintainability and extensibility for these additional features requested by the community. While we do test and make an effort to support previous schemas, it is strongly advised to regenerate the `default.yml` if you plan on upgrading to this version.

**DEPRECATION WARNING** As mentioned in the changelog, the environment variables `SPLUNK_SHC_SECRET` and `SPLUNK_IDXC_SECRET` will now be replaced by `SPLUNK_SHC_PASS4SYMMKEY` and `SPLUNK_IDXC_PASS4SYMMKEY` respectively. Both are currently supported and will be mapped to the same setting now, but in the future we will likely remove both `SPLUNK_SHC_SECRET` and `SPLUNK_IDXC_SECRET`

---

## 8.0.1

#### What's New?
* New Splunk Enterprise release of 8.0.1

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/8.0.1/ReleaseNotes/Fixedissues
* Bugfixes and increasing test coverage for new features

#### splunk-ansible changes:
* Service name fixes for AWS
* Bugfixes around forwarding and SHC-readiness
* Additional options to control SmartStore configuration
**NOTE** If you are currently using SmartStore, this change does break backwards-compatibility with former versions of the `default.yml` schema. This was necessary to expose the additional features asked for by the community. Please regenerate the `default.yml` if you plan on upgrading to this version.

---

## 8.0.0

#### What's New?
* New Splunk Enterprise release of 8.0.0

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/8.0.0/ReleaseNotes/Fixedissues
* Reduced base image size due to package management inflation
* Additional Python 2/Python 3 compatibility changes

#### splunk-ansible changes:
* Increasing delay intervals to better handle different platforms
* Adding vars needed for Ansible Galaxy
* Bugfix for pre-playbook tasks not supporting URLs

---

## 7.3.4

#### What's New?
* New Splunk Enterprise release of 7.3.4

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.3.4/ReleaseNotes/Fixedissues
* See [8.0.1](#801) changes

#### splunk-ansible changes:
* See [8.0.1](#801) changes

---

## 7.3.3

#### What's New?
* New Splunk Enterprise release of 7.3.3

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.3.3/ReleaseNotes/Fixedissues
* Better management of deployment server apps
* Support for variety of Splunk package types
* Bugfixes around app installation

#### splunk-ansible changes:
* Removing unnecessary apps in distributed ITSI installations
* Partioning apps in serverclass.conf when using the deployment server
* Adding support for activating Splunk Free license on boot
* Support for cluster labels via environment variables
* Bugfixes around app installation (through default.yml and pathing)

---

## 7.3.2

#### What's New?
* New Splunk Enterprise release of 7.3.2

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.3.2/ReleaseNotes/Fixedissues
* Support for Redhat 8 UF on s390x
* Various bugfixes

#### splunk-ansible changes:
* Python 2 and Python 3 cross compatibility support
* Support SPLUNK_SECRET as an environment variable
* Prevent double-installation issue when SPLUNK_BUILD_URL is supplied
* Various bugfixes

---

## 7.3.1

#### What's New?
* New Splunk Enterprise release of 7.3.1

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.3.1/ReleaseNotes/Fixedissues
* Documentation update
* Minor bug fixes

#### splunk-ansible changes:
* Fixed Enterprise Security application installation issues
* Refactored Systemd
* Fixed Ansible formatting issue
* Cleaned up Python files before install

---

## 7.3.0

#### What's New?
* Adding base `debian-10` and `redhat-8` platform
* Changing default `splunk/splunk` from `debian-9` to `debian-10` for enhanced security
* Overarching changes to build structure to support multi-stage builds for various consumers

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.3.0/ReleaseNotes/Fixedissues
* Changing default `splunk/splunk` from `debian-9` to `debian-10` for enhanced security
* Overarching changes to build structure to support multi-stage builds for various consumers
* Minor documentation changes

#### splunk-ansible changes:
* Adding ability to dynamically change `SPLUNK_ROOT_ENDPOINT` at start-up time
* Adding ability to dynamically change SplunkWeb HTTP port at start-up time
* Modified manner in which deployment server installs + distributes app bundles
* More multi-site functionality
* Support for Cygwin-based Windows environments
* Minor documentation changes

---

## 7.2.9

#### What's New?
* Releasing new images to support Splunk Enterprise maintenance patch.
* Bundling in changes to be consistent with the release of [8.0.0](#800)

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.2.9/ReleaseNotes/Fixedissues
* See [8.0.0](#800) changes

#### splunk-ansible changes:
* See [8.0.0](#800) changes

---

## 7.2.8

#### What's New?
Nothing - releasing new images to support Splunk Enterprise maintenance patch.

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.2.8/ReleaseNotes/Fixedissues

#### splunk-ansible changes:
* Nothing - releasing new images to support Splunk Enterprise maintenance patch

---

## 7.2.7

#### What's New?
* Adding base `debian-10` and `redhat-8` platform
* Changing default `splunk/splunk` from `debian-9` to `debian-10` for enhanced security
* Overarching changes to build structure to support multi-stage builds for various consumers

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.2.7/ReleaseNotes/Fixedissues
* Changing default `splunk/splunk` from `debian-9` to `debian-10` for enhanced security
* Overarching changes to build structure to support multi-stage builds for various consumers
* Minor documentation changes

#### splunk-ansible changes:
* Adding ability to dynamically change `SPLUNK_ROOT_ENDPOINT` at start-up time
* Adding ability to dynamically change SplunkWeb HTTP port at start-up time
* Modified manner in which deployment server installs + distributes app bundles
* More multi-site functionality
* Support for Cygwin-based Windows environments
* Minor documentation changes

---

## 7.2.6

#### What's New?
Nothing - releasing new images to support Splunk Enterprise maintenance patch.

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.2.6/ReleaseNotes/Fixedissues

#### splunk-ansible changes:
* Nothing - releasing new images to support Splunk Enterprise maintenance patch

---

## 7.2.5.1

#### What's New?
Nothing - releasing new images to support Splunk Enterprise maintenance patch.

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.2.5/ReleaseNotes/Fixedissues

#### splunk-ansible changes:
* Nothing - releasing new images to support Splunk Enterprise maintenance patch

---

## 7.2.5

#### What's New?
* Introducing multi-site to the party
* Added `splunk_deployment_server` role
* Minor bugfixes

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.2.5/ReleaseNotes/Fixedissues
* Documentation overhaul
* Adding initial framework to support multi-site deployments
* Removing built-in Docker stats app for Splunk Universal Forwarder due to lack of use and violation of permission model

#### splunk-ansible changes:
* Adding support for `splunk_deployment_server` role
* Adding initial framework to support multi-site deployments
* Small refactor of upgrade logic
* Ansible syntactic sugar and playbook clean-up
* Documentation overhaul
* Adding CircleCI to support automated regression testing

---

## 7.2.4

#### What's New?
* Support for Java installation in standalones and search heads
* Hardening of asyncronous SHC bootstrapping procedures
* App installation across all topologies
* Adding CircleCI to support automated regression testing
* Minor bugfixes

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.2.4/ReleaseNotes/Fixedissues
* Adding Clair scanner for automated security scanning
* Adding CircleCI to support automated regression testing
* Minor documentation changes

#### splunk-ansible changes:
* Changing replication port from 4001 to 9887 for PS and field best practices
* Adding support for multiple licenses via URL and filepath globs
* Adding support for java installation
* Hardening SHC-readiness during provisioning due to large-scale deployment syncronization issues
* Extracting out `admin` username to be dynamic and flexible and enabling it to be user-defined
* App installation support for distributed topologies (SHC, IDXC, etc.) and some primitive premium app support
* Supporting Splunk restart only when required (via Splunk internal restart_required check)
* Minor documentation changes

---

## 7.2.3

#### What's New?
Nothing - releasing new images to support Splunk Enterprise maintenance patch.

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.2.3/ReleaseNotes/Fixedissues

#### splunk-ansible changes:
* Nothing - releasing new images to support Splunk Enterprise maintenance patch

---

## 7.2.2

#### What's New?
* Permission model refactor
* Minor bugfixes

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.2.2/ReleaseNotes/Fixedissues
* Adding base `centos-7` platform
* Packages added to all base platforms: `acl` and `ping`
* Minor documentation changes
* Significant permission model refactor such that `splunkd` will be run under the `splunk:splunk` user/group and the `ansible-playbook` setup will be run under `ansible:ansible` user/group
* Introducing new environment variable `CONTAINER_ARTIFACT_DIR` for various artifacts

#### splunk-ansible changes:
* Writing ansible logs to container artifact directory
* Adding templates for various OS/distributions to define default `default.yml` settings
* Adding `no_log` to prevent password exposure
* Support new permission model with `become/become_user` elevation/de-elevation when interacting with `splunkd`
* Support for out-of-the-box SSL-enabled SplunkWeb
* Adding ability to generate any configuration file in `$SPLUNK_HOME/etc/system/local`
* Introducing user-defined pre- and post- playbook hooks that can run before and after (respectively) site.yml
* Minor documentation changes

---

## 7.2.1

#### What's New?
* Initial SmartStore support
* App installation for direct URL link, local tarball, and from SplunkBase for standalone and forwarder

#### docker-splunk changes:
* Bumping Splunk version. For details, see: https://docs.splunk.com/Documentation/Splunk/7.2.1/ReleaseNotes/Fixedissues
* Adding `python-requests` to base Docker image
* Adding app installation features (direct link, local tarball, and SplunkBase)
* Minor documentation changes

#### splunk-ansible changes:
* Minor documentation changes
* Introducing support for [SmartStore](https://docs.splunk.com/Documentation/Splunk/latest/Indexer/AboutSmartStore) and index creation via `defaults.yml`
* Checks for first-time run to drive idempotency
* Adding capability to enable boot-start of splunkd as a service
* Support for user-defined splunk.secret file
* Adding app installation features (direct link, local tarball, and SplunkBase)
* Fixing bug where HEC receiving was not enabled on various Splunk roles
* Ansible syntactic sugar and playbook clean-up
* Minor documentation changes

---

## 7.2.0

#### What's New?
Everything :)

#### docker-splunk changes:
* Initial release!
* Support for Splunk Enterprise and Splunk Universal Forwarder deployments on Docker
* Supporting standalone and distributed topologies

#### splunk-ansible changes:
* Initial release!
* Support for Splunk Enterprise and Splunk Universal Forwarder deployments on Docker
* Supporting standalone and distributed topologies
## Advanced

Let's dive into the nitty-gritty on how to tweak the setup of your containerized Splunk deployment. This section goes over in detail various features and functionality that a traditional Splunk Enterprise solution is capable of.

## Navigation

* [Runtime configuration](#runtime-configuration)
    * [Valid Splunk env vars](#valid-splunk-env-vars)
    * [Valid UF env vars](#valid-uf-env-vars)
    * [Using default.yml](#using-default.yml)
        * [Generation](#generation)
        * [Usage](#usage)
        * [Spec](#spec)
* [Entrypoint Functions](#entrypoint-functions)
* [Install apps](#install-apps)
* [Apply Splunk license](#apply-splunk-license)
* [Create custom configs](#create-custom-configs)
* [Enable SmartStore](#enable-smartstore)
* [Using deployment servers](#using-deployment-servers)
* [Deploy distributed topology](#deploy-distributed-topology)
* [Enable SSL internal communication](#enable-ssl-internal-communication)
* [Build from source](#build-from-source)
    * [base-debian-9](#base-debian-9)
    * [splunk-debian-9](#splunk-debian-9)
    * [uf-debian-9](#uf-debian-9)

## Runtime configuration
Splunk's Docker image has several functions that can be configured. These options are specified by either supplying a `default.yml` file or by passing in environment variables. 

Passed in environment variables and/or default.yml are consumed by the inventory script in [splunk-ansible project](https://github.com/splunk/splunk-ansible).

Please refer to [Environment Variables List](https://splunk.github.io/splunk-ansible/ADVANCED.html#inventory-script)

#### Using default.yml
The purpose of the `default.yml` is to define a standard set of variables that controls how Splunk gets set up. This is particularly important when deploying clustered Splunk topologies, as there are frequent variables that you need to be consistent across all members of the cluster (ex. keys, passwords, secrets).

##### Generation
The image contains a script to enable dynamic generation of this file automatically. Run the following command to generate a `default.yml`:
```
$ docker run --rm -it splunk/splunk:latest create-defaults > default.yml
```

You can also pre-seed some settings based on environment variables during this `default.yml` generation process. For instance, you can define `SPLUNK_PASSWORD` as so:
```
$ docker run --rm -it -e SPLUNK_PASSWORD=<password> splunk/splunk:latest create-defaults > default.yml
```
##### Usage
When starting the docker container, this `default.yml` can be mounted in `/tmp/defaults/default.yml` or it can be fetched dynamically with `SPLUNK_DEFAULTS_URL`, and the provisioning done by Ansible will read in and honor these settings. Note that environment variables specified at runtime will take precendence over things defined in `default.yml`.
```
# Volume-mounting option
$ docker run -d -p 8000:8000 -v default.yml:/tmp/defaults/default.yml -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=<password> splunk/splunk:latest

# URL option
$ docker run -d -p 8000:8000 -v -e SPLUNK_DEFAULTS_URL=http://company.net/path/to/default.yml -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=<password> splunk/splunk:latest
```

##### Spec
Root items influence the behavior of everything in the container; they have global scope inside the container.
Example:
```
---
retry_num: 100
```

| Variable Name | Description | Parent Object | Default Value | Required for Standalone | Required for Search Head Clustering | Required for Index Clustering |
| --- | --- | --- | --- | --- | --- | --- |
| retry_num | Default number of loop attempts to connect containers | none | 100 | yes | yes | yes |

The major object "splunk" in the YAML file will contain variables that influence how Splunk operates. Example:
```
splunk:
  opt: /opt
  home: /opt/splunk
  user: splunk
  group: splunk
  exec: /opt/splunk/bin/splunk
  pid: /opt/splunk/var/run/splunk/splunkd.pid
  password: "{{ splunk_password | default(<password>) }}"
  svc_port: 8089
  s2s_port: 9997
  http_port: 8000
  hec_port: 8088
  hec_disabled: 0
  hec_enableSSL: 1
  # The hec_token here is used for INGESTION only. By that I mean receiving Splunk events.
  # Setting up your environment to forward events out of the cluster is another matter entirely
  hec_token: <default_hec_token>
  # This option here is to enable the SmartStore feature
  smartstore: null
```

| Variable Name | Description | Parent Object | Default Value | Required for Standalone | Required for Search Head Clustering | Required for Index Clustering |
| --- | --- | --- | --- | --- | --- | --- |
| opt | Parent directory where Splunk is running | splunk | /opt | yes | yes | yes |
| home | Location of the Splunk Installation | splunk | /opt/splunk | yes | yes | yes |
| user | Operating System User to Run Splunk Enterprise As | splunk | splunk | yes | yes | yes |
| group | Operating System Group to Run Splunk Enterprise As | splunk | splunk | yes | yes | yes |
| exec | Path to the Splunk Binary | splunk | /opt/splunk/bin/splunk | yes | yes | yes |
| pid | Location to the Running PID File | splunk | /opt/splunk/var/run/splunk/splunkd.pid | yes | yes | yes
| root_endpoint | Set root endpoint for SplunkWeb (for reverse proxy usage) | splunk | **none** | no | no | no |
| password | Password for the admin account | splunk | **none** | yes | yes | yes |
| svc_port | Default Admin Port | splunk | 8089 | yes | yes | yes |
| s2s_port | Default Forwarding Port | splunk | 9997 | yes | yes | yes |
| http_port | Default SplunkWeb Port | splunk | 8000 | yes | yes | yes |
| hec_port | Default HEC Input Port | splunk | 8088 | no | no | no |
| hec_disabled | Enable / Disable HEC | splunk | 0 | no | no | no |
| hec_enableSSL | Force HEC to use encryption | splunk | 1 | no | no | no |
| hec_token | Token to enable for HEC inputs | splunk | **none** | no | no | no |
| smartstore | Configuration params for [SmartStore](https://docs.splunk.com/Documentation/Splunk/latest/Indexer/AboutSmartStore) bootstrapping | splunk | null | no | no | no |

The app_paths section is located as part of the "splunk" parent object. The settings located in this section will directly influence how apps are installed inside the container. Example:
```
  app_paths:
    default: /opt/splunk/etc/apps
    shc: /opt/splunk/etc/shcluster/apps
    idxc: /opt/splunk/etc/master-apps
    httpinput: /opt/splunk/etc/apps/splunk_httpinput
```

| Variable Name | Description | Parent Object | Default Value | Required for Standalone | Required for Search Head Clustering | Required for Index Clustering |
| --- | --- | --- | --- | --- | --- | --- |
| default | Normal apps for standalone instances will be installed in this location | splunk.app_paths | **none** | no | no | no |
| shc | Apps for search head cluster instances will be installed in this location (usually only done on the deployer)| splunk.app_paths | **none** | no | no | no |
| idxc | Apps for index cluster instances will be installed in this location (usually only done on the cluster master)| splunk.app_paths | **none** | no | no | no |
| httpinput | App to use and configure when setting up HEC based instances.| splunk.app_paths | **none** | no | no | no |

Search Head Clustering can be configured using the "shc" sub-object. Example:
```
  shc:
    enable: false
    secret: <secret_key>
    replication_factor: 3
    replication_port: 9887
```
| Variable Name | Description | Parent Object | Default Value | Required for Standalone | Required for Search Head Clustering | Required for Index Clustering |
| --- | --- | --- | --- | --- | --- | --- |
| enable | Instructs the container to create a search head cluster | splunk.shc | false | no | yes | no |
| secret | A secret phrase to use for all SHC communication and binding. Please note, once set this can not be changed without rebuilding the cluster. | splunk.shc | **none** | no | yes | no |
| replication_factor | Consult docs.splunk.com for valid settings for your use case | splunk.shc | 3 | no | yes | no |
| replication_port | Default port for the SHC to communicate on | splunk.shc | 9887 | no | yes | no |

Lastly, Index Clustering is configured with the `idxc` sub-object. Example:
```
  idxc:
    secret: <secret_key>
    search_factor: 2
    replication_factor: 3
    replication_port: 9887
```
| Variable Name | Description | Parent Object | Default Value | Required for Standalone| Required for Search Head Clustering | Required for Index Clustering |
| --- | --- | --- | --- | --- | --- | --- |
| secret | Secret used for transmission between the cluster master and indexers | splunk.idxc | **none** | no | no | yes |
| search_factor | Search factor to be used for search artifacts | splunk.idxc | 2 | no | no | yes |
| replication_factor | Bucket replication factor used between index peers | splunk.idxc | 3 | no | no | yes |
| replication_port | Bucket replication Port between index peers | splunk.idxc | 9887 | no | no | yes |

## Install apps 
Briefly, apps can be installed by using the `SPLUNK_APPS_URL` environment variable when creating the Splunk container:
```
$ docker run -it --name splunk -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=<password> -e SPLUNK_APPS_URL=http://company.com/path/to/app.tgz splunk/splunk:latest
```

See the [full app installation walkthrough](advanced/APP_INSTALL.md) to understand how to specify multiple apps and how to apply it in a distributed environment.

## Apply Splunk license
Briefly, licenses can be added with the `SPLUNK_LICENSE_URI` environment variable when creating the Splunk container:
```
$ docker run -it --name splunk -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=<password> -e SPLUNK_LICENSE_URI=http://company.com/path/to/splunk.lic splunk/splunk:latest
```

See the [full license installation guide](advanced/LICENSE_INSTALL.md) to understand how to specify multiple licenses and how to use a central, containerized license manager.

## Create custom configs
When Splunk boots, it registers all the config files in various locations on the filesystem under `${SPLUNK_HOME}`. These are settings that control how Splunk operates. For more information, please see the [documentation from Splunk](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Aboutconfigurationfiles).

Using the Docker image, it is also possible for users to create their own config files, following the same INI-style that drives Splunk. This is a power-user/admin-level feature, as invalid config files may break or prevent start-up of your Splunk installation!

User-specified configuration files are only possible through the use of a `default.yml`. Please set a `conf` key under the greater `splunk` key using the format shown below.
```
---
splunk:
  conf:
    user-prefs:
      directory: /opt/splunkforwarder/etc/users/admin/user-prefs/local
      content:
        general:
          default_namespace: appboilerplate
          search_syntax_highlighting: dark
  ...
```

This will generate a file owned by the correct Splunk user and group, named `user-prefs.conf` and located within the `directory` (in this case, `/opt/splunkforwarder/etc/users/admin/user-prefs/local`). Because it follows INI-format, the contents of the final file will resemble the following:
```
[general]
search_syntax_highlighting = dark
default_namespace = appboilerplate
```

For multiple custom configuration files, please add more entries under the `conf` key of the `default.yml`.

**CAUTION:** Using this method of configuration file generation may not create a configuration file the way Splunk expects. Verify the generated configuration file to avoid errors. Use at your own discretion.

## Enable SmartStore
SmartStore utilizes S3-compliant object storage in order to store indexed data. This is a capability only available if you're using an indexer cluster (cluster_master + indexers). For more information, please see the [blog post](https://www.splunk.com/blog/2018/10/11/splunk-smartstore-cut-the-cord-by-decoupling-compute-and-storage.html) as well as [technical overview](https://docs.splunk.com/Documentation/Splunk/latest/Indexer/AboutSmartStore).

This docker image is capable of support SmartStore, as long as you bring-your-own backend storage provider. Due to the complexity of this option, this is only enabled if you specify all the parameters in your `default.yml` file. 

Here's an overview of what this looks like if you want to persist *all* your indexes (default) with a SmartStore backend:
```
---
splunk:
  smartstore:
    index:
      - indexName: default
        remoteName: remote_store
        scheme: s3
        remoteLocation: <bucket-name>
        s3:
          access_key: <access_key>
          secret_key: <secret_key>
          endpoint: http://s3-us-west-2.amazonaws.com
  ...
```

Some cache management options are also available. Options defined under the index stanza correspond to options in `indexes.conf` https://docs.splunk.com/Documentation/Splunk/latest/admin/Indexesconf. While options defined outside the index correspond to options in `server.conf` https://docs.splunk.com/Documentation/Splunk/latest/admin/Serverconf, note that currently only `[cachemanager]` stanza is supported. This is an example config that defines cache settings and retention policy:
```
smartstore:
  cachemanager:
    max_cache_size: 500
    max_concurrent_uploads: 7
  index:
    - indexName: custom_index
      remoteName: my_storage
      scheme: http
      remoteLocation: my_storage.net
      maxGlobalDataSizeMB: 500
      maxGlobalRawDataSizeMB: 200
      hotlist_recency_secs: 30
      hotlist_bloom_filter_recency_hours: 1
```

## Using deployment servers
Briefly, deployment servers can be used to manage otherwise unclustered/disjoint Splunk instances. A primary use-case would be to stand up a deployment server to manage app or configuration distribution to a fleet of 100 universal forwarders.

See the [full deployment server guide](advanced/DEPLOYMENT_SERVER.md) to understand how you can leverage this role in your topology.

## Deploy distributed topology
While running a standalone Splunk instance may be fine for testing and development, you may eventually want to scale out to enable better performance of running Splunk at scale. This image does support a fully-vetted distributed Splunk environment, by using environment variables that enable certain containers to assume certain roles, and to network everything together.

See the [instructions on standing up a distributed environment](advanced/DISTRIBUTED_TOPOLOGY.md) to understand how to get started.

## Enable SSL Internal Communication
For users looking to secure the network traffic from one Splunk instance to another Splunk instance (ex: forwarders to indexers), you can enable forwarding and receiving to use SSL certificates. 

If you wish to enable SSL on one tier of your Splunk topology, it's very likely all instances will need it. To achieve this, we recommend you generate your server and CA certificates and add them to the `default.yml` which gets shared across all Splunk docker containers. Use this example `default.yml` snippet for the configuration of Splunk TCP with SSL.  
```
splunk:
  ...
  s2s:
    ca: /mnt/certs/ca.pem
    cert: /mnt/certs/cert.pem
    enable: true
    password: abcd1234
    port: 9997
    ssl: true
  ...
```

For more instructions on how to bring your own certificates, please see: https://docs.splunk.com/Documentation/Splunk/latest/Security/ConfigureSplunkforwardingtousesignedcertificates

## Build from source
While we don't support or recommend you building your own images from source, it is entirely possible. This can be useful if you want to incorporate very experimental features, test new features, and if you have your own registry for persistent images.

To build images directly from this repository, there is a supplied `Makefile` in the root of the project with commands and variables to control the build:
1. Fork the [docker-splunk GitHub repository](https://github.com/splunk/docker-splunk/issues)
2. Clone your fork using git and create a branch off develop
    ```
    $ git clone git@github.com:YOUR_GITHUB_USERNAME/docker-splunk.git
    $ cd docker-splunk
    ```
3. Run all the tests to verify your environment
    ```
    $ make splunk-redhat-8
    $ make uf-redhat-8
    ```

Additionally, there are multiple images and layers that are produced by the previous commands: `base-redhat-8`, `splunk-redhat-8`, and `uf-redhat-8`.

#### base-redhat-8
The directory `base-redhat-8` contains a Dockerfile to create a base image on top of which all the other images are built. In order to minimize image size and provide a stable foundation for other images to build on, we elected to use `registry.access.redhat.com/ubi8/ubi-minimal:8.0` (90MB) for our base image. In the future, we plan to add support for additional operating systems.
```
$ make base-redhat-8
```

**WARNING:** Modifications made to the "base" image can result in Splunk being unable to start or run correctly.

#### splunk-redhat-8
The directory `splunk/common-files` contains a Dockerfile that extends the base image by installing Splunk and adding tools for provisioning. Advanced Splunk provisioning capabilities are provided through the utilization of an entrypoint script and playbooks published separately via the [splunk-ansible project](https://github.com/splunk/splunk-ansible).
```
$ make splunk-redhat-8
```

#### uf-redhat-8
The directory `uf/common-files` contains a Dockerfile that extends the base image by installing Splunk Universal Forwarder and adding tools for provisioning. This image is similar to the Splunk Enterprise image (`splunk-redhat-8`), except the more lightweight Splunk Universal Forwarder package is installed instead.
```
$ make uf-redhat-8
```
## Data Storage ##
This section will cover examples of different options for configuring data persistence. This includes both indexed data and 
configuration items. Splunk only supports data persistence to volumes mounted outside of the container. Data persistence for 
folders inside of the container is not supported. The following are intended as only as examples and unofficial guidelines. 

### Storing indexes and search artifacts ###
Splunk Enterprise, by default, Splunk Enterprise uses the var directory for indexes, search artifacts, etc. In the public image, the Splunk Enterprise 
home directory is /opt/splunk, and the indexes are configured to run under var/. If you want to persist the indexed 
data, then mount an external directory into the container under this folder.

If you do not want to modify or persist any configuration changes made outside of what has been defined in the docker 
image file, then use the following steps for your service.

#### Step 1: Create a named volume ####
To create a simple named volume in your Docker environment, run the following command
```
docker volume create so1-var
```
See Docker's official documentation for more complete instructions and additional options.

#### Step 2: Define the docker compose YAML  and start the service####
Using the Docker Compose format, save the following contents into a docker-compose.yml file

```
version: "3.6"

networks:
  splunknet:
    driver: bridge
    attachable: true

volumes:
  so1-var:

services:
  so1:
    networks:
      splunknet:
        aliases:
          - so1
    image: splunk-debian-9:latest
    container_name: so1
    environment:
      - SPLUNK_START_ARGS=--accept-license
      - SPLUNK_PASSWORD=changem3N0w!
      - DEBUG=true
    ports:
      - 8000
      - 8089
    volumes:
      - so1-var:/opt/splunk/var
```

This mounts only the contents of /opt/splunk/var, so anything outside of this folder will not persist. Any configuration changes will not 
remain when the container exits.  Note that changes will persist between starting and stopping a container. See 
Docker's documentation for more discussion on the difference between starting, stopping, and exiting if the difference
between them is unclear.

In the same directory as docker-compose.yml run the following command
```
docker-compose up
```
to start the service.

#### Viewing the contents of the volume ####
To view the data outside of the container run
```
docker volume inspect so1-var
```
The output of that command should list where the data is stored.

### Storing indexes, search artifacts, and configuration changes ###
In this section, we build off of the previous example to save the configuration as well. This can make it easier to save modified 
configurations, but simultaneously allows configuration drift to occur. If you want to keep configuration drift from 
happening, but still want to be able to persist some of the data, you can save off the specific "local" folders that 
you want the data to be persisted for (such as etc/system/local). However, be careful when doing this because you will 
both know what folders you need to save off and the number of volumes can proliferate rapidly - depending on the 
deployment. Please take the "Administrating Splunk" through Splunk Education prior to attempting this configuration.

In these examples, we will assume that the entire etc folder is being mounted into the container.

#### Step 1: Create a named volume ####
Again, create a simple named volume in your Docker environment, run the following command
```
docker volume create so1-etc
```
See Docker's official documentation for more complete instructions and additional options.

#### Step 2: Define the docker compose YAML ####
Notice that this differs from the previous example by adding in the so1-etc volume references.
In the following example, save the following data into a file named docker-compose.yml

```
version: "3.6"

networks:
  splunknet:
    driver: bridge
    attachable: true

volumes:
  so1-var:
  so1-etc:

services:
  so1:
    networks:
      splunknet:
        aliases:
          - so1
    image: splunk-debian-9:latest
    container_name: so1
    environment:
      - SPLUNK_START_ARGS=--accept-license
      - SPLUNK_PASSWORD=<password>
      - DEBUG=true
    ports:
      - 8000
      - 8089
    volumes:
      - so1-var:/opt/splunk/var
	    - so1-etc:/opt/splunk/etc
```

In the directory where the docker-compose.yml file is saved, run 
```
docker-compose up
```
to start the service.

When the volume is mounted the data will persist after the container exits. If a container has exited and restarted, 
but no data shows up, then check the volume definition and verify that the container did not create a new volume 
or that the volume mounted is in the same location. 

#### Viewing the contents of the volume ####
To view the etc directory outside of the container run one or both of the commands
```
docker volume inspect so1-etc
```
The output of that command should list the directory associated with the volume mount.

#### Volume Mount Guidelines ####
**Do not mount the same folder into two different Splunk Enterprise instances, this can cause inconsistencies in the 
indexed data and undefined behavior within Splunk Enterprise itself.**

### Upgrading Splunk instances in your containers ###
Upgrading Splunk instances requires volumes to be mounted for /opt/splunk/var and /opt/splunk/etc.

#### Step 1: Persist your /opt/splunk/var and /opt/splunk/etc ####
Follow the named volume creation tutorial above in order to have /opt/splunk/var and /opt/splunk/etc mounted for persisting data.

#### Step 2: Update your yaml file with a new image and SPLUNK_UPGRADE=true ####
In the same yaml file you initially used to deploy Splunk instances, update the specified image to the next version of Splunk image. Then, set **SPLUNK_UPGRADE=true** in the environment of all containers you wish to upgrade. Make sure to state relevant named volumes so persisted data can be mounted to a new container.

Below is an example yaml with SPLUNK_UPGRADE=true

```
version: "3.6"

networks:
  splunknet:
    driver: bridge
    attachable: true

volumes:
  so1-var:
  so1-etc:

services:
  so1:
    networks:
      splunknet:
        aliases:
          - so1
    image: <NEXT_VERSION_SPLUNK_IMAGE>
    container_name: so1
    environment:
      - SPLUNK_START_ARGS=--accept-license
      - SPLUNK_PASSWORD=<password>
      - DEBUG=true
      - SPLUNK_UPGRADE=true
    ports:
      - 8000
      - 8089
    volumes:
      - so1-var:/opt/splunk/var
	    - so1-etc:/opt/splunk/etc
```

#### Step 3: Deploy your containers using the updated yaml ####
Similar to how you initially deployed your containers, run the command with the updated yaml that contains a reference to the new image and SPLUNK_UPGRADE=true in the environment. Make sure that you do NOT destory previously existing network and volumes. After running the command with the yaml file, your containers should be recreated with the new version of Splunk and persisted data properly mounted to /opt/splunk/var and /opt/splunk/etc.

#### Different types of volumes ####
Using named volume is recommended so it is easier to attach and detach volumes to different Splunk instances while persisting your data. If you use anonymous volumes, Docker gives them random and unique names so you can still reuse anonymous volumes on different containers. If you use bind mounts, make sure that the mounts are setup properly to persist /opt/splunk/var and opt/splunk/etc. Starting new containers without proper mounts will result in a loss of your data.

Note [Docker Volume Documentation](https://docs.docker.com/storage/volumes/#create-and-manage-volumes) for more details about managing volumes. 
## Architecture
From a design perspective, the containers brought up with the `docker-splunk` images are meant to provision themselves locally and asynchronously. The execution flow of the provisioning process is meant to gracefully handle interoperability in this manner, while also maintaining idempotency and reliability. 

## Navigation

* [Networking](#networking)
* [Design](#design)
    * [Remote networking](#remote-networking)
* [Supported platforms](#supported-platforms)

## Networking
By default, the Docker image exposes a variety of ports for both external interaction as well as internal use. 
```
EXPOSE 8000 8065 8088 8089 8191 9887 9997
```

Below is a table detailing the purpose of each port, which can be used as a reference for determining whether the port should be published for external consumption.

| Port Number | Description |
| --- | --- |
| 8000 | SplunkWeb UI |
| 8065 | Splunk app server |
| 8088 | HTTP Event Collector (HEC) |
| 8089 | SplunkD management port (REST API access) |
| 8191 | Key-value store replication |
| 9887 | Index replication |
| 9997 | Indexing/receiving |

## Design

##### Remote networking 
Particularly when bringing up distributed Splunk topologies, there is a need for one Splunk instances to make a request against another Splunk instance in order to construct the cluster. These networking requests are often prone to failure, as when Ansible is executed asyncronously there are no guarantees that the requestee is online/ready to receive the message.

While developing new playbooks that require remote Splunk-to-Splunk connectivity, we employ the use of `retry` and `delay` options for tasks. For instance, in this example below, we add indexers as search peers of individual search head. To overcome error-prone networking, we have retry counts with delays embedded in the task. There are also break-early conditions that maintain idempotency so we can progress if successful:
```
- name: Set all indexers as search peers
  command: "{{ splunk.exec }} add search-server https://{{ item }}:{{ splunk.svc_port }} -auth {{ splunk.admin_user }}:{{ splunk.password }} -remoteUsername {{ splunk.admin_user }} -remotePassword {{ splunk.password }}"
  become: yes
  become_user: "{{ splunk.user }}"
  with_items: "{{ groups['splunk_indexer'] }}"
  register: set_indexer_as_peer
  until: set_indexer_as_peer.rc == 0 or set_indexer_as_peer.rc == 24
  retries: "{{ retry_num }}"
  delay: 3
  changed_when: set_indexer_as_peer.rc == 0
  failed_when: set_indexer_as_peer.rc != 0 and 'already exists' not in set_indexer_as_peer.stderr
  notify:
    - Restart the splunkd service
  no_log: "{{ hide_password }}"
  when: "'splunk_indexer' in groups"
```

Another utility you can add when creating new plays is an implicit wait. For more information on this, see the `roles/splunk_common/tasks/wait_for_splunk_instance.yml` play which will wait for another Splunk instance to be online before making any connections against it.
```
- name: Check Splunk instance is running
  uri:
    url: https://{{ splunk_instance_address }}:{{ splunk.svc_port }}/services/server/info?output_mode=json
    method: GET
    user: "{{ splunk.admin_user }}"
    password: "{{ splunk.password }}"
    validate_certs: false
  register: task_response
  until:
    - task_response.status == 200
    - lookup('pipe', 'date +"%s"')|int - task_response.json.entry[0].content.startup_time > 10
  retries: "{{ retry_num }}"
  delay: 3
  ignore_errors: true
  no_log: "{{ hide_password }}"
```

## Supported platforms
At the current time, this project only officially supports running Splunk Enterprise on `debian:stretch-slim`. We do have plans to incorporate other operating systems and Windows in the future.

## Navigation

* [Troubleshooting](#troubleshooting)
* [System Validation](#system-validation)
* [Splunk Validation](#splunk-validation)
* [Container Debugging](#container-debugging)
    * [Getting logs](#getting-logs)
    * [Interactive shell](#interactive-shell)
    * [Debug variables](#debug-variables)
    * [No-provision](#no-provision)
    * [Generate Splunk diag](#generate-splunk-diag)
* [Contact](#contact)

## Troubleshooting
As with most asynchronous design patterns, troubleshooting can be an arduous task. However, there are some built-in utilities to that you can employ to make your lives easier.

## System Validation
The most important step in troubleshooting is to validate the environment your container runs in. Please ensure that the following questions are answered:
* Is Docker installed properly?
* Is the Docker daemon running?
* Are you using the overlay2 storage-driver?
* Can you run simple Linux images (ex. debian, centos, alpine, etc.)?
* Are you using the latest Splunk image? Or is this an older image?
* Do the image hashes match?
* Are there any settings that could influence or limit the container's behavior (ex. kernel, hardware, etc.)?
* Are there other containers that might be impacting the running Splunk containers (i.e. noisy neighbors)?

## Splunk Validation
Please refer to the [setup/installation page](SETUP.md#install) for comprehensive documentation on what is required on the host before launching your Splunk container. However, you may also want to verify that:
* Container environment variable names are spelled correctly
* Container environment variable values are spelled correctly
* Environment variables that accept filepaths and URLs exist

## Container Debugging
If you find your container gets into a state when it crashes immediately on boot, or it hits a crash-loop backoff failure mode, you may need to do some live debugging. Here is some advice to help you proceed and figure out exactly why your container keeps crashing

#### Getting logs
The logs are by far the best way to analyze what went wrong with your deployment or container. Because we rely heavily on the [splunk-ansible](https://github.com/splunk/splunk-ansible) project underneath the hood of this image, it's very likely that your container hit a specific failure in the Ansible level. Grabbing the container's stdout/stderr will tell you at what point and at what play this exception occurred.

To check for container logs, you can run the following command:
```
$ docker logs <container_name/container_id>
```

Alternatively, it's also possible to stream logs so you can watch what happens during the provisioning process in real-time:
```
$ docker logs --follow <container_name/container_id>
$ docker logs -f <container_name/container_id>
```

#### Interactive shell
If your container is still running but in a bad state, you can try to debug by putting yourself within the context of that process. 


To gain interactive shell access to the container's runtime as the splunk user, you can run:
```
$ docker exec -it -u splunk <container_name/container_id> /bin/bash
```

#### Debug variables
There are some built-in environment variables to assist with troubleshooting. Please be aware that when using these variables, it is possible for sensitive keys and information to be shown on the container's stdout/stderr. If you are using any custom logging driver or solution that persists this information, we recommend disabling it for the duration of this debug session.

The two primary environment variables that may be of assistance are `DEBUG` and `ANSIBLE_EXTRA_FLAGS`.

In order to use `DEBUG`, create a container and define `DEBUG=true` as so:
```
$ docker run -it -e DEBUG=true -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=<password> splunk/splunk:latest
ansible-playbook 2.7.7
  config file = /opt/ansible/ansible.cfg
  configured module search path = [u'/opt/ansible/library', u'/opt/ansible/apps/library', u'/opt/ansible/ansible_commands']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 2.7.13 (default, Sep 26 2018, 18:42:22) [GCC 6.3.0 20170516]
{
    "_meta": {
        "hostvars": {
            "localhost": {
                "ansible_connection": "local"
            }
        }
    },
    ...
```

If you check the container logs for this particular container, you'll notice the entire object generated by the dynamic inventory script `environ.py` is printed out at the top. This is particularly useful if you need to validate that certain variables are defined accordingly and map to exactly what has been explicitly set. It also displays the Python version and Ansible version used, which will greatly help if you plan on submitting a [GitHub issue](https://github.com/splunk/docker-splunk/issues) to make it easy for others to reproduce.

The `ANSIBLE_EXTRA_FLAGS` is another help environment variable that can be used to display more information or output from Ansible. For instance, one practical application of this could be:
```
$ docker run -it -e "ANSIBLE_EXTRA_FLAGS=-vv" -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=<password> splunk/splunk:latest
ansible-playbook 2.7.7
  config file = /opt/ansible/ansible.cfg
  configured module search path = [u'/opt/ansible/library', u'/opt/ansible/apps/library', u'/opt/ansible/ansible_commands']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 2.7.13 (default, Sep 26 2018, 18:42:22) [GCC 6.3.0 20170516]
Using /opt/ansible/ansible.cfg as config file
/opt/ansible/inventory/environ.py did not meet host_list requirements, check plugin documentation if this is unexpected

PLAYBOOK: site.yml ************************************************************
1 plays in site.yml

PLAY [Run default Splunk provisioning] ****************************************
Thursday 21 February 2019  00:50:55 +0000 (0:00:00.036)       0:00:00.036 *****

TASK [Gathering Facts] ********************************************************
task path: /opt/ansible/site.yml:2
ok: [localhost]
META: ran handlers
Thursday 21 February 2019  00:50:56 +0000 (0:00:01.148)       0:00:01.185 *****
```
With the above, you'll notice how much more rich and verbose the Ansible output becomes, simply by adding more verbosity to the actual Ansible execution. 

#### No-provision
The `no-provision` is a fairly useless supported command - after launching the container, it won't run Ansible so Splunk will not get installed or even setup. Instead, it tails a file to keep the instance up and running. 

This `no-provision` keyword is an argument that gets passed into the container's entrypoint script, so you can use it in the following manner:
```
$ docker run -it -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=<password> -e SPLUNK_HEC_TOKEN=abcd1234 --name spldebug splunk/splunk:latest no-provision
```

While by itself this seems fairly useless, it can be a great way to debug and troubleshoot Ansible problems locally. In the case above, we've set a `SPLUNK_HEC_TOKEN` environment variable. Let's go inside of this container and kick off Ansible manually, and make sure that the HEC token is set properly and that Splunk uses it:
```
$ docker exec -it spldebug bash
ansible@5f60f3164e69:/$ echo $SPLUNK_HEC_TOKEN
abcd1234
ansible@5f60f3164e69:/$ /sbin/entrypoint.sh start

PLAY [Run default Splunk provisioning] ****************************************
Thursday 21 February 2019  01:09:50 +0000 (0:00:00.034)       0:00:00.034 *****

TASK [Gathering Facts] ********************************************************
ok: [localhost]
```

#### Generate Splunk diag
A Splunk diagnostic file (diag) is a dump of a Splunk environment that shows how the instance is configured and how it has been operating. If you plan on working with Splunk Support, you may be requested to generate a diag for them to assist you. For more information on what is contained in a diag, refer to this [topic](https://docs.splunk.com/Documentation/Splunk/latest/Troubleshooting/Generateadiag).

Generating a diag is only an option if:
1. The Splunk container is active and running
2. Administrators are able to `docker exec` into said container
3. Splunk itself has a bug/performance problem

To create this diag, run the following command:
```
$ docker exec -it -u splunk <container_name/container_id> "${SPLUNK_HOME}/bin/splunk diag"
```

Additionally, if your Docker container/hosts have access to https://www.splunk.com you can now send the file directly to Splunk Support by using the following command:
```
$ docker exec -it -u splunk <container_name/container_id> "${SPLUNK_HOME}/bin/splunk diag --upload --case-number=<case_num> --upload-user=<your_splunk_id> --upload-password=<passwd> --upload-description='Monday diag, as requested'"
```

However, if you don't have direct access, you can manually copy the diag back to your host via `docker cp`:
```
$ docker cp <container_name/container_id>:/opt/splunk/<filename> <location on your local machine>
```

## Contact
If you require additional assistance, please see the [support guidelines](SUPPORT.md#contact) on how to reach out to Splunk Support with issues or questions.  
# Contributing to the Project

This document is the single source of truth on contributing towards this codebase. Please feel free to browse the open issues and file new ones - all feedback is welcome!

## Topics

* [Prerequisites](#prerequisites)
    * [Contributor License Agreement](#contributor-license-agreement)
    * [Code of Conduct](#code-of-conduct)
    * [Setup development environment](#setup-development-environment)
* [Contribution Workflow](#contribution-workflow)
    * [Feature Requests and Bug Reports](#feature-requests-and-bug-reports)
    * [Fixing Issues](#fixing-issues)
    * [Pull Requests](#pull-requests)
    * [Code Review](#code-review)
    * [Testing](#testing)
    * [Documentation](#documentation)
* [Maintainers](#maintainers)

## Prerequisites
When contributing to this repository, please first discuss the change you wish to make via a GitHub issue, Slack message, email, or via other channels with the owners of this repository.

##### Contributor License Agreement
At the moment, we can only accept pull requests submitted from either:
* Splunk employees or
* Individuals that have signed our contribution agreement

If you wish to be a contributing member of our community, please see the agreement [for individuals](https://www.splunk.com/goto/individualcontributions) or [for organizations](https://www.splunk.com/goto/contributions).

##### Code of Conduct
Please make sure to read and observe our [Code of Conduct](contributing/code-of-conduct.md). Please follow it in all of your interactions involving the project.

##### Setup Development Environment
TODO

## Contribution Workflow
Help is always welcome! For example, documentation can always use improvement. There's always code that can be clarified, functionality that can be extended, and tests to be added to guarantee behavior. If you see something you think should be fixed, don't be afraid to own it.

##### Feature Requests and Bug Reports
Have ideas on improvements? See something that needs work? While the community encourages everyone to contribute code, it is also appreciated when someone reports an issue. Please report any issues or bugs you find through [GitHub's issue tracker](https://github.com/splunk/docker-splunk/issues). 

If you are reporting a bug, please include:
* Your operating system name and version
* Any details about your local setup that might be helpful in troubleshooting (ex. Python interpreter version, Ansible version, etc.)
* Detailed steps to reproduce the bug

We'd also like to hear about your propositions and suggestions. Feel free to submit them as issues and:
* Explain in detail how they should work
* Keep the scope as narrow as possible - this will make it easier to implement

##### Fixing Issues
Look through our [issue tracker](https://github.com/splunk/docker-splunk/issues) to find problems to fix! Feel free to comment and tag corresponding stakeholders or full-time maintainers of this project with any questions or concerns.

##### Pull Requests
What is a "pull request"? It informs the project's core developers about the changes you want to review and merge. Once you submit a pull request, it enters a stage of code review where you and others can discuss its potential modifications and even add more commits to it later on. 

If you want to learn more, please consult this [tutorial on how pull requests work](https://help.github.com/articles/using-pull-requests/) in the [GitHub Help Center](https://help.github.com/).

Here's an overview of how you can make a pull request against this project:
1. Fork the [docker-splunk GitHub repository](https://github.com/splunk/docker-splunk/issues)
2. Clone your fork using git and create a branch off develop
    ```
    $ git clone git@github.com:YOUR_GITHUB_USERNAME/docker-splunk.git
    $ cd docker-splunk

    # This project uses 'develop' for all development activity, so create your branch off that
    $ git checkout -b your-bugfix-branch-name develop
    ```
3. Run all the tests to verify your environment
    ```
    $ cd docker-splunk
    $ make test
    ```
4. Make your changes, commit and push once your tests have passed
    ```
    $ git commit -m "<insert helpful commit message>"
    $ git push 
    ```
5. Submit a pull request through the GitHub website using the changes from your forked codebase

##### Code Review
There are two aspects of code review: giving and receiving.

To make it easier for your PR to receive reviews, consider the reviewers will need you to:
* Follow the project coding conventions
* Write good commit messages
* Break large changes into a logical series of smaller patches which individually make easily understandable changes, and in aggregate solve a broader issue

Reviewers, the people giving the review, are highly encouraged to revisit the [Code of Conduct](contributing/code-of-conduct.md) and must go above and beyond to promote a collaborative, respectful community.

When reviewing PRs from others [The Gentle Art of Patch Review](http://sage.thesharps.us/2014/09/01/the-gentle-art-of-patch-review/) suggests an iterative series of focuses which is designed to lead new contributors to positive collaboration without inundating them initially with nuances:
* Is the idea behind the contribution sound?
* Is the contribution architected correctly?
* Is the contribution polished?

For this project, we require that at least 2 approvals are given and a build from our continuous integration system is successful off of your branch. Please note that any new changes made with your existing pull request during review will automatically unapprove and retrigger another build/round of tests.

##### Testing
Testing is the responsibility of all contributors. In general, we try to adhere to [Google's test sizing philosophy](https://testing.googleblog.com/2010/12/test-sizes.html) when structuring tests. 

There are multiple types of tests. The location of the test code varies with type, as do the specifics of the environment needed to successfully run the test.

1. **Small:** Very fine-grained; exercises low-level logic at the scope of a function or a class; no external resources (except possibly a small data file or two, but preferably no file system dependencies whatsoever); very fast execution on the order of seconds
    ```
    $ make small-tests
    ```

2. **Medium:** Exercises interaction between discrete components; may have file system dependencies or run multiple processes; runs on the order of minutes
    ```
    $ make medium-tests
    ```

3. **Large:** Exercises the entire system, end-to-end; used to identify crucial performance and basic functionality that will be run for every code check-in and commit; may launch or interact with services in a datacenter, preferably with a staging environment to avoid affecting production
    ```
    $ make large-tests
    ```

Continuous integration will run all of these tests either as pre-submits on PRs, post-submits against master/release branches, or both.

##### Documentation
We could always use improvements to our documentation! Anyone can contribute to these docs - whether you’re new to the project, you’ve been around a long time, and whether you self-identify as a developer, an end user, or someone who just can’t stand seeing typos. What exactly is needed?
1. More complementary documentation. Have you perhaps found something unclear?
2. More examples or generic templates that others can use.
3. Blog posts, articles and such – they’re all very appreciated.

You can also edit documentation files directly in the GitHub web interface, without creating a local copy. This can be convenient for small typos or grammer fixes.

## Maintainers

If you need help, feel free to tag one of the active maintainers of this project in a post or comment. We'll do our best to reach out to you as quickly as we can.

```
# Active maintainers marked with (*)

(*) Nelson Wang
(*) Tony Lee
(*) Brent Boe
(*) Matthew Rich
(*) Jonathan Vega
(*) Jack Meixensperger
(*) Brian Bingham
(*) Scott Centoni
(*) Mike Dickey
```
## Navigation

* [Preface](#preface)
* [System Requirements](#system-requirements)
* [Contact](#contact)
* [Support Violation](#support-violation)

## Preface
Splunk Enterprise contains many settings that allow customers to tailor their Splunk environment. However, because not all settings apply to all customers, Splunk will only support the most common subset of all configurations. Throughout this document, the term "supported" means you can contact Splunk Support for assistance with issues.

## System Requirements
At the current time, this image only supports the Docker runtime engine and requires the following system prerequisites:
1. Linux-based operating system (Debian, CentOS, etc.)
2. Chipset: 
    * `splunk/splunk` image supports x86-64 chipsets
    * `splunk/universalforwarder` image supports both x86-64 and s390x chipsets
3. Kernel version > 4.0
4. Docker engine
    * Docker Enterprise Engine 17.06.2 or later
    * Docker Community Engine 17.06.2 or later
5. `overlay2` Docker daemon storage driver
    * Create a file /etc/docker/daemon.json on Linux systems, or C:\ProgramData\docker\config\daemon.json on Windows systems. Add {"storage-driver": "overlay2"} to the daemon.json. If you already have an existing json, please only add "storage-driver": "overlay2" as a key, value pair.

For more details, please see the official [supported architectures and platforms for containerized Splunk environments](https://docs.splunk.com/Documentation/Splunk/latest/Installation/Systemrequirements#Containerized_computing_platforms) as well as [hardware and capacity recommendations](https://docs.splunk.com/Documentation/Splunk/latest/Installation/Systemrequirements). 

If you intend for this containerized Splunk Enterprise deployment to be supported in your Enterprise Support Agreement, you must verify you meet all of the above "supported" requirements. Failure to do so will render your deployment in an "unsupported" state. 

## Contact
Splunk Support only provides support for the single instance Splunk Validated Architectures (S-Type), Universal Forwarders and Heavy Forwarders. For all other configurations, please contact Splunk Professional Services. Please contact them according to the instructions [here](https://www.splunk.com/en_us/support-and-services.html).

If you have additional questions or need more support, you can:
* Post a question to [Splunk Answers](http://answers.splunk.com)
* Join the [#docker](https://splunk-usergroups.slack.com/messages/C1RH09ERM/) room in the [Splunk Slack channel](http://splunk-usergroups.slack.com)
* If you are a Splunk Enterprise customer with a valid support entitlement contract and have a Splunk-related question, you can also open a support case on the https://www.splunk.com/ support portal

## Support Violation
In the following conditions, Splunk Support reserves the right to deem your installation unsupported and not provide assistance when issues arise: 
* You do not have an active support contract
* You are running Splunk Enterprise/Splunk Universal Forwarder in a container on a platform not officially supported by Splunk
* You are using features not officially supported by Splunk

In the event you fall into an unsupported state, you may find support on Splunk Answers, or through the open source communities found in this [docker-splunk](https://github.com/splunk/docker-splunk) GitHub project or the related [splunk-ansible](https://www.github.com/splunk/splunk-ansible) GitHub project.
Apache License

Version 2.0, January 2004

http://www.apache.org/licenses/

TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

#### 1. Definitions.

"License" shall mean the terms and conditions for use, reproduction, and distribution as defined by Sections 1 through 9 of this document.

"Licensor" shall mean the copyright owner or entity authorized by the copyright owner that is granting the License.

"Legal Entity" shall mean the union of the acting entity and all other entities that control, are controlled by, or are under common control with that entity. For the purposes of this definition, "control" means (i) the power, direct or indirect, to cause the direction or management of such entity, whether by contract or otherwise, or (ii) ownership of fifty percent (50%) or more of the outstanding shares, or (iii) beneficial ownership of such entity.

"You" (or "Your") shall mean an individual or Legal Entity exercising permissions granted by this License.

"Source" form shall mean the preferred form for making modifications, including but not limited to software source code, documentation source, and configuration files.

"Object" form shall mean any form resulting from mechanical transformation or translation of a Source form, including but not limited to compiled object code, generated documentation, and conversions to other media types.

"Work" shall mean the work of authorship, whether in Source or Object form, made available under the License, as indicated by a copyright notice that is included in or attached to the work (an example is provided in the Appendix below).

"Derivative Works" shall mean any work, whether in Source or Object form, that is based on (or derived from) the Work and for which the editorial revisions, annotations, elaborations, or other modifications represent, as a whole, an original work of authorship. For the purposes of this License, Derivative Works shall not include works that remain separable from, or merely link (or bind by name) to the interfaces of, the Work and Derivative Works thereof.

"Contribution" shall mean any work of authorship, including the original version of the Work and any modifications or additions to that Work or Derivative Works thereof, that is intentionally submitted to Licensor for inclusion in the Work by the copyright owner or by an individual or Legal Entity authorized to submit on behalf of the copyright owner. For the purposes of this definition, "submitted" means any form of electronic, verbal, or written communication sent to the Licensor or its representatives, including but not limited to communication on electronic mailing lists, source code control systems, and issue tracking systems that are managed by, or on behalf of, the Licensor for the purpose of discussing and improving the Work, but excluding communication that is conspicuously marked or otherwise designated in writing by the copyright owner as "Not a Contribution."

"Contributor" shall mean Licensor and any individual or Legal Entity on behalf of whom a Contribution has been received by Licensor and subsequently incorporated within the Work.

#### 2. Grant of Copyright License. 
Subject to the terms and conditions of this License, each Contributor hereby grants to You a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare Derivative Works of, publicly display, publicly perform, sublicense, and distribute the Work and such Derivative Works in Source or Object form.

#### 3. Grant of Patent License. 
Subject to the terms and conditions of this License, each Contributor hereby grants to You a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable (except as stated in this section) patent license to make, have made, use, offer to sell, sell, import, and otherwise transfer the Work, where such license applies only to those patent claims licensable by such Contributor that are necessarily infringed by their Contribution(s) alone or by combination of their Contribution(s) with the Work to which such Contribution(s) was submitted. If You institute patent litigation against any entity (including a cross-claim or counterclaim in a lawsuit) alleging that the Work or a Contribution incorporated within the Work constitutes direct or contributory patent infringement, then any patent licenses granted to You under this License for that Work shall terminate as of the date such litigation is filed.

#### 4. Redistribution. 
You may reproduce and distribute copies of the Work or Derivative Works thereof in any medium, with or without modifications, and in Source or Object form, provided that You meet the following conditions:

You must give any other recipients of the Work or Derivative Works a copy of this License; and
You must cause any modified files to carry prominent notices stating that You changed the files; and
You must retain, in the Source form of any Derivative Works that You distribute, all copyright, patent, trademark, and attribution notices from the Source form of the Work, excluding those notices that do not pertain to any part of the Derivative Works; and
If the Work includes a "NOTICE" text file as part of its distribution, then any Derivative Works that You distribute must include a readable copy of the attribution notices contained within such NOTICE file, excluding those notices that do not pertain to any part of the Derivative Works, in at least one of the following places: within a NOTICE text file distributed as part of the Derivative Works; within the Source form or documentation, if provided along with the Derivative Works; or, within a display generated by the Derivative Works, if and wherever such third-party notices normally appear. The contents of the NOTICE file are for informational purposes only and do not modify the License. You may add Your own attribution notices within Derivative Works that You distribute, alongside or as an addendum to the NOTICE text from the Work, provided that such additional attribution notices cannot be construed as modifying the License. 

You may add Your own copyright statement to Your modifications and may provide additional or different license terms and conditions for use, reproduction, or distribution of Your modifications, or for any such Derivative Works as a whole, provided Your use, reproduction, and distribution of the Work otherwise complies with the conditions stated in this License.
5. Submission of Contributions. Unless You explicitly state otherwise, any Contribution intentionally submitted for inclusion in the Work by You to the Licensor shall be under the terms and conditions of this License, without any additional terms or conditions. Notwithstanding the above, nothing herein shall supersede or modify the terms of any separate license agreement you may have executed with Licensor regarding such Contributions.

6. Trademarks. This License does not grant permission to use the trade names, trademarks, service marks, or product names of the Licensor, except as required for reasonable and customary use in describing the origin of the Work and reproducing the content of the NOTICE file.

7. Disclaimer of Warranty. Unless required by applicable law or agreed to in writing, Licensor provides the Work (and each Contributor provides its Contributions) on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE. You are solely responsible for determining the appropriateness of using or redistributing the Work and assume any risks associated with Your exercise of permissions under this License.

8. Limitation of Liability. In no event and under no legal theory, whether in tort (including negligence), contract, or otherwise, unless required by applicable law (such as deliberate and grossly negligent acts) or agreed to in writing, shall any Contributor be liable to You for damages, including any direct, indirect, special, incidental, or consequential damages of any character arising as a result of this License or out of the use or inability to use the Work (including but not limited to damages for loss of goodwill, work stoppage, computer failure or malfunction, or any and all other commercial damages or losses), even if such Contributor has been advised of the possibility of such damages.

9. Accepting Warranty or Additional Liability. While redistributing the Work or Derivative Works thereof, You may choose to offer, and charge a fee for, acceptance of support, warranty, indemnity, or other liability obligations and/or rights consistent with this License. However, in accepting such obligations, You may act only on Your own behalf and on Your sole responsibility, not on behalf of any other Contributor, and only if You agree to indemnify, defend, and hold each Contributor harmless for any liability incurred by, or claims asserted against, such Contributor by reason of your accepting any such warranty or additional liability.

END OF TERMS AND CONDITIONS