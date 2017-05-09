Master: [![Build Status](https://travis-ci.org/usdot-jpo-ode/jpo-cvdp.svg?branch=master)](https://travis-ci.org/usdot-jpo-ode/jpo-cvdp)

# jpo-cvdp

The United States Department of Transportation Joint Program Office (JPO)
Connected Vehicle Data Privacy (CVDP) Project is developing a variety of methods
to enhance the privacy of individuals who generated connected vehicle data.

Connected vehicle technology uses in-vehicle wireless transceivers to broadcast
and receive basic safety messages (BSMs) that include accurate spatiotemporal
information to enhance transportation safety. Integrated Global Positioning
System (GPS) measurements are included in BSMs.  Databases, some publicly
available, of BSM sequences, called trajectories, are being used to develop
safety and traffic management applications. **BSMs do not contain explicit
identifiers that link trajectories to individuals; however, the locations they
expose may be sensitive and associated with a very small subset of the
population; protecting these locations from unwanted disclosure is extremely
important.** Developing procedures that minimize the risk of associating
trajectories with individuals is the objective of this project.

# The Operational Data Environment (ODE) Privacy Protection Module (PPM)

The PPM operates on streams of raw BSMs generated by the ODE. It determines
whether individual BSMs should be retained or suppressed (deleted) based on the
information in that BSM and auxiliary map information used to define a geofence.
BSM geoposition (latitude and longitude) and speed are used to determine the
disposition of each BSM processed. The PPM also redacts other BSM fields.

## PPM Limitations

Protecting against inference-based privacy attacks on spatiotemporal
trajectories (i.e., sequences of BSMs from a single vehicle) in **general** is
a challenging task. An example of an inference-based privacy attack is
identifying the driver that generated a sequence of BSMs using specific
locations they visit during their trip, or other features discernable from the
information in the BSM sequence. **This PPM treats a specific use case: a
geofenced area where residences do not exist, e.g., a highway corridor, with
certain speed restrictions.** Do not assume this strategy will work in general.
There are alternative strategies that must be employed to handle cases where
loitering locations can aid in learning the identity of the driver.

# Release Notes

## ODE Sprint 11

- (Partial Complete) ODE-282 Implement a Module that Interfaces with the ODE.
- (Partially Complete) ODE-77 Implement a PPM that uses a Geofence to Filter BSMs.

## ODE Sprint 12

- Complete documentation for ODE-77

# Documentation

The following documents will help practitioners build, test, deploy, and understand the PPM's functions:

- [Privacy Protection Module User Guide](docs/ppm_user_manual.docx)
- [Coding Standards](docs/coding-standards.md)

All stakeholders are invited to provide input to these documents. Stakeholders should direct all input on this document
to the JPO Product Owner at DOT, FHWA, or JPO. To provide feedback, we recommend that you create an "issue" in this
repository (https://github.com/usdot-jpo-ode/jpo-cvdp/issues). You will need a GitHub account to create an issue. If you
don’t have an account, a dialog will be presented to you to create one at no cost.

## Code Documentation

Code documentation can be generated using [Doxygen](https://www.doxygen.org) by following the commands below:

```bash
$ sudo apt install doxygen
$ cd <install root>/jpo-cvdp
$ doxygen
```

The documentation is in HTML and is written to the `<install root>/jpo-cvdp/docs/html` directory. Open `index.html` in a
browser.

# Collaboration Tools

## Source Repositories - GitHub
- Main repository on GitHub (public)
	- https://github.com/usdot-jpo-ode/jpo-cvdp
	- git@github.com:usdot-jpo-ode/jpo-cvdp.git

## Agile Project Management - Jira
https://usdotjpoode.atlassian.net/secure/Dashboard.jspa

# Getting Started

## Prerequisites

You will need Git to obtain the code and documents in this repository.
Furthermore, we recommend using Docker to build the necessary containers to
build, test, and experiment with the PPM. The [Docker](#docker) instructions can be found in that section.

- [Git](https://git-scm.com/)
- [Docker](https://www.docker.com)

More detail on installing these tools are outlined below in the [Installation Setup and Testing Section](#installation-setup-and-testing)

## Getting the Source Code

See the installation and setup instructions unless you just want to examine the code.

**Step 1.** Disable Git `core.autocrlf` (Only the First Time)

   **NOTE**: If running on Windows, please make sure that your global git config is
   set up to not convert End-of-Line characters during checkout. This is important
   for building docker images correctly.
   
```bash
git config --global core.autocrlf false
```

**Step 2.** Clone the source code from GitHub repositories using Git commands:

```bash
git clone https://github.com/usdot-jpo-ode/jpo-cvdp.git
```

# Installation Setup and Testing

The following instructions represent the "hard" way to install and test the PPM. A docker image can be built to make
this easier (see X). *The directions that follow were developed for a clean installation of Ubuntu.*

- Install Vim so you have a decent editor ;)

```bash
$ sudo apt install vim
```

- Install [Git](https://git-scm.com/)

```bash
$ sudo apt install git
```

- Install Oracle’s Java

```bash
$ sudo add-apt-repository -y ppa:webupd8team/java
$ sudo apt update
$ sudo apt install oracle-java8-installer -y
$ sudo java -version
```

- Install [CMake](https://cmake.org) to build the PPM

```bash
$ sudo apt install cmake
```

- Install [Docker](https://www.docker.com)

    - When following the website instructions, setup the Docker repos and follow the Linux post-install instructions.
    - The CE version seems to work fine.
    - [Docker installation instructions](https://docs.docker.com/engine/installation/linux/ubuntu/#install-using-the-repository)
    - *ORNL specific, but may apply to others with organizational security*
        - Correct for internal Google DNS blocking
        - As root (`$ sudo su`), create a `daemon.json` file in the `/etc/docker` directory with the following information:
```bash
          {
              "debug": true,
              "default-runtime": "runc",
              "dns": ["160.91.126.23","160.91.126.28”],  // these are ORNL specific.
              "icc": true,
              "insecure-registries": [],
              "ip": "0.0.0.0",
              "log-driver": "json-file",
              "log-level": "info",
              "max-concurrent-downloads": 3,
              "max-concurrent-uploads": 5,
              "oom-score-adjust": -500
          }
```

- Restart the docker daemon to consume the new configuration file.

```bash
$ service docker stop
$ service docker start
```

- Check the configuration using the command below to confirm the updates above are taken if needed:
```bash
$ docker info
```

- Install Docker Compose
    - Comprehensive instructions can be found on this [website](https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-16-04)
    - Follow steps 1 and 2.

- Create a base directory from which to install all the necessary components to test the PPM.

```bash
$ export BASE_PPM_DIR=~/some/dir/you/want/to/put/this/stuff
```

- Install [`kafka-docker`](https://github.com/wurstmeister/kafka-docker) so kafka and zookeeper can run in a separate container.
    - These services will need to be running to test the PPM.
```bash
$ cd $BASE_PPM_DIR
$ git clone https://github.com/wurstmeister/kafka-docker.git
$ ifconfig                                              // to get the host ip address (device ens33 maybe)
$ export DOCKER_HOST_IP=<HOST IP>
$ cd kafka-docker
$ vim docker-compose.yml	                        // Set karka: ports: to 9092:9092
$ docker-compose up --no-recreate -d                    // to startup kafka and zookeeper containers and not recreate
$ docker-compose ps                                     // to check that they are running.
```
    
- When you want to stop kafka and zookeeper

```bash
$ cd $BASE_PPM_DIR/kafka-docker
$ docker-compose down
```

- Download and install the Kafka **binary** for the particular version of scala you are using.
    -  The Kafka binary provides a producer and consumer tool that can act as surrogates for the ODE (among other items).
    -  [Kafka Binary](https://kafka.apache.org/downloads)
    -  [Kafka Quickstart](https://kafka.apache.org/quickstart) is a very useful reference.
    -  Move and unpack the Kafka code as follows:

```bash
$ cd $BASE_PPM_DIR
$ wget http://apache.claz.org/kafka/0.10.2.1/kafka_2.12-0.10.2.1.tgz   // mirror and kafka version may change; check website.
$ tar -xzf kafka_2.12-0.10.2.1.tgz			               // the kafka version may be different.
$ mv kafka_2.12-0.10.2.1 kafka
```
   
- Download and install [`librdkafka`](https://github.com/edenhill/librdkafka), the C++ Kafka library we use to build the PPM.
    - Upon completion of the instructions below, the header files for `librdkafka` should be located in `/usr/local/include/librdkafka` 
      and the libraries (static and dynamic) should be located in `/usr/local/lib`. If you put them in another location
      the PPM may not build.

```bash
$ cd $BASE_PPM_DIR
$ git clone https://github.com/edenhill/librdkafka.git
$ cd librdkafka
$ ./configure
$ make
$ sudo make install
```

- Download, build, and install the Privacy Protection Module (PPM)

```bash
$ cd $BASE_PPM_DIR
$ git clone https://github.com/usdot-jpo-ode/jpo-cvdp.git
$ cd jpo-cvdp
$ mkdir build && cd build
$ cmake ..
$ make
```

- The PPM uses [RapidJSON](https://github.com/miloyip/rapidjson), but it is a header-only library included in the repository.

## PPM Configuration

The PPM configuration file is a text file with a specific format. It can be used to configure Kafka as well as the PPM.
Comments can be added to the configuration file by starting a line with the '#' character. Configuration lines consist
of two strings separated by a '=' character; lines are terminated by newlines. The names of configuration files can be
anything; extensions do not matter.

The following is an example of a portion of a configuration file:

    # Configuration details for privacy ID redaction.
    privacy.redaction.id=ON
    privacy.redaction.id.value=FFFFFFFF
    privacy.redaction.id.inclusions=ON
    privacy.redaction.id.included=BEA10000,BEA10001

Example configuration files can be found in the [jpo-cvdp/config](config) directory, e.g., [example.properties](config/example.properties) is an example of a complete configuration file.

The details of the settings and how they affect the function of the PPM follow:

- `privacy.filter.velocity` : enables or disables BSM filtering based on the speed within the BSM.
    - `ON` : enables BSM filtering.
    - Any other value : disables BSM filtering.

- `privacy.filter.velocity.min` : *When velocity fitering is enabled*, BSMs having velocities below this value will be
  suppressed. The units are in meters per second.

- `privacy.filter.velocity.max` : *When velocity fitering is enabled*, BSMs having velocities above this value will be
  suppressed. The units are in meters per second.

- `privacy.redaction.id` : enables or disables the PPM's redaction function for the BSM `id` field (`TemporaryID` field in J2735). 
    - `ON` : enables redaction
    - Any other value : disables redaction.

- `privacy.redaction.id.value` : *If redaction is enabled*, this value will replace the current value in the `id` field of the raw BSM.
    - According to the J2735, this value is 4 hexidecimal-encoded bytes. The configured value should **NOT** be
      enclosed in quotes or be preceded by 0x.

- `privacy.redaction.id.inclusions` : *If redaction is enabled*, this parameter enables or disables the ability to specify 
   **which** identifier values should be redacted.
    - `ON` : enables use of a redaction *inclusion* set. The values in the set are defined in
      the `privacy.redaction.id.included` configuration parameter.
    - Any other value : **causes all identifiers to be redacted.** Ignores the inclusion set.

- `privacy.redaction.id.included` : *If redaction and redaction inclusions are enabled*, the parameter is the list of BSM 
   identifiers (right now TemporaryID) that **will be redacted**; BSMs having identifiers that are not in this set will remain in 
   the BSM output by the PPM if retained.
    - Similar to the `privacy.redaction.id.value`, these are 4 hexadecimal-encoded bytes.
    - More than one id can be specified by separating them by commas.

- `privacy.filter.geofence` : enables or disables geofence-based filtering.
    - `ON` : enables the geofence.
    - Any other value : disables geofence filtering.

- `privacy.filter.geofence.mapfile` : *If redaction is enabled*, specifies the absolute or relative path and filename of a file that contains the
  map information needed to define the geofence.
  
- Geofence Boundary Configuration Parameters: The geofence is stored in a geographically-defined data structured called
  a quadtree. The following bounding box coordinates define the quadtree's region. The data that is stored in this data
  structure is limited to those segments provided in the mapfile, e.g., `privacy.filter.geofence.mapfile`. As an example
  of this relationship, the coordinates specified below could bound the entire state of Wyoming; however, only the
  segments for the I-80 corridor would be stored within a quadtree covering Wyoming and used to define the geofence. One
  the other hand, these coordinates can be used to **further restrict** which segments are used to define the geofence
  instead of having to modify the mapfile. 

      - `privacy.filter.geofence.sw.lat` : The latitude of the lower-left corner of the quadtree region.
      - `privacy.filter.geofence.sw.lon` : The longitude of the lower-left corner of the quadtree region.
      - `privacy.filter.geofence.ne.lat` : The latitude of the upper-right corner of the quadtree region.
      - `privacy.filter.geofence.ne.lon` : The longitude of the upper-right corner of the quadtree region.

- `privacy.topic.consumer` : The Kafka topic name used by the Operational Data Environment (or other BSM JSON producer) that will be
  consumed by the PPM. The source of the data stream to be filtered by the PPM.
- `privacy.topic.producer` : The Kafka topic name where the PPM will write the filtered BSMs.

- `group.id` : The group identifier for the PPM consumer.  Consumers label
  themselves with a consumer group name, and each record published to a topic is
  delivered to one consumer instance within each subscribing consumer group.
  Consumer instances can be in separate processes or on separate machines.

- `privacy.kafka.partition` : The partition(s) that this PPM will consume records from. A Kafka topic can be divided, 
  or partitioned, into several "parallel" streams. A topic may have many partitions so it can handle an arbitrary 
  amount of data.

- `metadata.broker.list` : This is the IP address of the Kafka topic broker leader.

- `compression.type` : The type of compression to use for writing to Kafka topics. Currently, this should be set to none.

## Map Files

The map file is used to define the geofence. It defines a set of shapes, one
per line. For road geofence use, the edge shape is used. The map file for the
I-80 WYDOT corridor is located in the [jpo-cvdp/data](data) directory; it is named: `I_80.edges`

The following is a small portion of the `I_80.edges` file:

```bash
type,id,geography,attributes
edge,0,0;41.24789403;-111.0467118:1;41.24746145;-111.0455124,way_type=user_defined:way_id=80
edge,1,1;41.24746145;-111.0455124:2;41.24733395;-111.0451337,way_type=user_defined:way_id=80
edge,2,2;41.24733395;-111.0451337:3;41.24726205;-111.044904,way_type=user_defined:way_id=80
edge,3,3;41.24726205;-111.044904:4;41.24713975;-111.0444827,way_type=user_defined:way_id=80
```

This file has four comma-separated elements:

- type : `edge`
- shape identifier : unique 64-bit integer identifier
- geography : A sequence of colon-split triples representing points; each point is semi-colon split as follows:
    - `<point uid>;<latitude>;<longitude>`
- attributes : A sequence of colon-split `key=value` attributes.
    - The attribute `way_type` determines the width of the geofence around a road segment.

For the WYDOT use case, WYDOT provided a set of edge definitions for I-80 that were converted into the above format.

## Test Files

There are several example JSON BSM test files in the [jpo-cvdp/data](data) directory.  These files can be edited to generate
your own test cases. Each line in the file should be a well-formed BSM JSON
object.**Each BSM should be on a separate line in the file.** **If a JSON object cannot be parsed it is suppressed.**

## Testing the PPM

These instructions describe how to run a collection of BSM test JSON objects through the PPM and examine its operation.
Using *GNU screen* for this work is really handy; you will need several shells.

Startup `kafka-docker` in its own shell:

```bash
$ cd $BASE_PPM_DIR/kafka-docker
$ docker-compose up --no-recreate -d                    // to startup kafka and zookeeper containers
$ docker-compose ps                                     // to check that they are running.
```

In another shell, create the simulated ODE produced topic (`j2735BsmRawJson`) and PPM produced topic (`j2735BsmFilteredJson`)

```bash
$ cd $BASE_PPM_DIR/kafka
$ bin/kafka-topic.sh --create --zookeeper <HOST IP>:2181 --replication-factor 1 --partitions 1 --topic j2735BsmRawJson
$ bin/kafka-topic.sh --create --zookeeper <HOST IP>:2181 --replication-factor 1 --partitions 1 --topic j2735BsmFilteredJson
```

Startup the simulated ODE consumer in the same shell you used to create the topics.

```bash
$ cd $BASE_PPM_DIR/kafka                                
$ bin/kafka-console-consumer.sh --bootstrap-server <HOST IP>:9092 --topic j2735BsmFilteredJson
```

- This process should just wait for input from the PPM module.

In another shell, startup the PPM

```bash
$ cd $BASE_PPM_DIR/jpo-cvdp/build
$ ./bsmjson_privacy -c ../config/<testconfig>.properties
```

- At this point the PPM will wait for streaming messages from the simulated ODE
  producer.  When a message is received output will be generated describing how
  the PPM handled the BSM.

In another shell, send test JSON-encoded BSMs to the PPM.

```bash
$ cd $BASE_PPM_DIR/kafka                                
$ cat $BASE_PPM_DIR/jpo-cvdp/data/<testfile> | bin/kafka-console-producer.sh --broker-list <HOST IP>:9092 --topic j2735BsmRawJson
```

- After the messages are written to the `j2735BsmRawJson` topic the shell process should return.

You can confirm PPM operations in two ways:

- Return to the PPM shell and examine the output; it should behave based on PPM configuration settings and the test BSMs.
- Return to the simulated ODE consumer shell and examine the output JSON; this is more difficult because the JSON does not render well on the screen.

## Testing All Capabilities

- To execute the following tests, you will stop the PPM module, if running, with `<CTRL>-C` and then start it up with one of the following `<testconfig>.properties` files.
    - `test.allon.properties`
    - `test.geofenceonly.properties`
    - `test.idredactonly.properties`
    - `test.spdonly.properties`
- After starting the PPM module, you will use the shell you created above to send the testfile, [jpo-cvdp/data/testing_data.json](data/testing_data.json) to the Kafka producer:

```bash
$ cat $BASE_PPM_DIR/jpo-cvdp/data/testing_data.json | bin/kafka-console-producer.sh --broker-list <HOST IP>:9092 --topic j2735BsmRawJson
```

- In the PPM shell the output should look something like the following:
```bash
$ ./bsmjson_privacy -c ../config/test.allon.properties
>> Created Consumer: rdkafka#consumer-1
>> Created Producer: rdkafka#producer-2
Retaining BSM: Pos: (41.7381, -106.587), Spd: 7.02 Id: FFFFFFFF
Filtering BSM [parse] : Pos: (90, 180), Spd: -1 Id: UNASSIGNED
Filtering BSM [speed] : Pos: (41.6087, -109.227), Spd: 1.12 Id: FFFFFFFF
Filtering BSM [parse] : Pos: (90, 180), Spd: -1 Id: UNASSIGNED
Filtering BSM [speed] : Pos: (41.3111, -110.513), Spd: 97.16 Id: BEA10009
Retaining BSM: Pos: (41.2466, -111.027), Spd: 7.44 Id: BEA10009
Retaining BSM: Pos: (41.6004, -106.223), Spd: 7.44 Id: BEA10004
Filtering BSM [parse] : Pos: (90, 180), Spd: -1 Id: UNASSIGNED
Filtering BSM [geoposition] : Pos: (42.2979, -83.7203), Spd: -1 Id: BEA10004
Filtering BSM [geoposition] : Pos: (42.2979, -83.7203), Spd: -1 Id: FFFFFFFF
Filtering BSM [geoposition] : Pos: (42.2458, -83.6234), Spd: -1 Id: FFFFFFFF
Filtering BSM [geoposition] : Pos: (42.2458, -83.6234), Spd: -1 Id: BEA10005
Filtering BSM [geoposition] : Pos: (42.2458, -83.6234), Spd: -1 Id: FFFFFFFF
```
- The above output will vary depending on which configuration file you use.

### Continuous Integration and Delivery

### Deployment

## Docker

To run a series of configuration tests:

    # ./do_test.sh

**Running this command for the first time can take awhile, as the dependencies need to be built for the PPM image.** 
To run the standalone PPM test using Docker containers, from the jpo-cvdp root directory:
    
    $ ./start_kafka.sh

This will build and start the required kafka containers, including the PPM image. 
Next run:

    $ ./test-scripts/standalone.sh [MAP_FILE] [CONFIG] [TEST_FILE] [OFFSET]

Where MAP_FILE is a [map file](#map-files), CONFIG is [PPM Configuration file](#ppm-configuration) and TEST_FILE is a [JSON BSM test file](#test-files). Offset refers to the offset in the filtered BSM topic; this is where the consumer will look for new output. The default offset is zero (the beginning of the topic), which should for the first time running the test. This will start the PPM Kafka container and use the supplied files to test the BSM filtering. For example, running:

    $ ./test-scripts/standalone.sh data/I_80.edges config/test/I_80_vel_filter.properties data/I_80_test.json

yields:

    **************************
    Running standalone test with data/I_80.edges config/test/I_80_vel_filter.properties data/I_80_test.json
    **************************
    **************************
    Producing Raw BSMs...
    **************************
    Producing BSM with ID=BEA10000, speed=7.02, position=41.738136, -106.587029
    Producing BSM with ID=BEA10000, speed=7.12, position=41.608656, -109.226824
    Producing BSM with ID=BEA10000, speed=7.16, position=41.311097, -110.512927
    Producing BSM with ID=BEA10000, speed=7.44, position=41.246647, -111.027436
    Producing BSM with ID=BEA10000, speed=7.44, position=41.600371, -106.22341
    Producing BSM with ID=BEA10000, speed=1.78, position=42.29789, -83.72035
    Producing BSM with ID=BEA10000, speed=0.7, position=42.29789, -83.72034
    Producing BSM with ID=BEA10000, speed=6.86, position=42.24576, -83.62337
    Producing BSM with ID=BEA10000, speed=6.84, position=42.24576, -83.62337
    Producing BSM with ID=BEA10000, speed=6.74, position=42.24576, -83.62338
    **************************
    Consuming Filtered BSMs at offset 0 ...
    **************************
    Consuming BSM with ID=BEA10000, speed=7.02, position=41.738136, -106.587029
    Consuming BSM with ID=BEA10000, speed=7.12, position=41.608656, -109.226824
    Consuming BSM with ID=BEA10000, speed=7.16, position=41.311097, -110.512927
    Consuming BSM with ID=BEA10000, speed=7.44, position=41.246647, -111.027436
    Consuming BSM with ID=BEA10000, speed=7.44, position=41.600371, -106.22341
    Consuming BSM with ID=BEA10000, speed=6.86, position=42.24576, -83.62337
    Consuming BSM with ID=BEA10000, speed=6.84, position=42.24576, -83.62337
    Consuming BSM with ID=BEA10000, speed=6.74, position=42.24576, -83.62338
