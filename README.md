# AIM

Simplest possible ELK stack deployable on an AWS EC2 instance to ingest events generated within the same AWS accounts and exposing a dashboard on the public internet.

# Scope

The focus is on the Elastic stack, not on AWS. The following are therefore out of scope:
* AWS Machine Image
* instance type
* instance configuration.

On the other hand, there are some constraints on the choices in the AWS setup to allow you to experiment with this ELK stack successfully:
* unless you are a new AWS user and can benefit from the initial 12 month free tier offer, all EC2 instances are chargeable. To minimize costs, it is therefore imperative that you stop the instance when it is not being used. This is not possible with a store-backed AMI. Therefore, choose an EBS-backed one. Be aware that you will be charged for the use of EBS storage for its entire lifetime as well as for the EC2 instance while it is running.
* we want to change as little as possible in the ELK default configuration settings. We are assuming a development environment. Per default, Elasticsearch sets the size of its heap to 1 GiB. On the other hand, its documentation states that heap size should be no more than half of the total RAM available. Therefore an instance type offering more than 2 GiB needs to be selected.

# Architecture

The deployment target is an inexpensive EC2 instance.

# Prerequisites

* Docker
* Docker Compose
* a running EC2 instance including the following packages:
  * git
  * Docker - it is advisable to add the EC2 user to the `docker` group. You may also want to configure the Docker engine to start at boot time.
  * Docker Compose

# Installation

```
git clone git@github.com:JohanPeeters/elk4ec2.git
cd elk4ec2
docker-compose up
```

This brings up 2 containers, 1 running Elasticsearch, the other Kibana. The next step is to configure access control. We want to avoid storing secrets in version controlled configuration files, so we will use the Elasticsearch and Kibana keystores.

But first, we need to set credentials for [Elasticsearch built-in users](https://www.elastic.co/guide/en/elastic-stack-overview/7.3/built-in-users.html).

```
cd elk4ec2
$ docker-compose exec es bash
# elasticsearch-setup-passwords auto  
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y
<passwords>
```

Connect to Kibana and set the password for Elasticsearch:
```
cd elk4ec2
$ docker-compose exec kibana bash
bash$ kibana-keystore create
Created Kibana keystore in /usr/share/kibana/data/kibana.keystore
bash$ kibana-keystore add elasticsearch.username
Enter value for elasticsearch.username: ******
bash$ kibana-keystore add elasticsearch.password
Enter value for elasticsearch.password: ********************
```
where the interactively entered `elasticsearch.username` is `kibana`. Use the Kibana password generated on `es` for `elasticsearch.password`.
For a more extensive explanation about the Kibana keystore, see the [Kibana secure settings manual](https://www.elastic.co/guide/en/kibana/7.3/secure-settings.html).

Kibana does not pick up the newly provided credentials, so you need to restart it. Probably easiest:
```
$ docker-compose restart
```

If you are running the ELK stack locally, you can now browse to Kibana at `http://localhost:5601` - it will prompt for a username and password. At the moment, there are no end-user accounts yet. In order to set these up via Kibana, you can log in with the `elastic` superuser that you created credentials previously. Subsequently, go to *Security Settings* under *Manage and Administer the Elastic Stack* and click the *Create user* button.

# References

* [List of Elastic Docker images](https://www.docker.elastic.co)
* [Elasticsearch 7.3 Docker installation guide](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/docker.html)
* [Kibana 7.3 Docker installation guide](https://www.elastic.co/guide/en/kibana/7.3/docker.html)
* [Manage Docker as a non-root user](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)
* [Configure Docker to start on boot](https://docs.docker.com/install/linux/linux-postinstall/#configure-docker-to-start-on-boot)

References are to the Elasticsearch version current at the time of creating this repository. Subsequent releases may introduce changes. Elastic usually maintains the same directory structure for their documentation across releases and the latest version should be available by substituting `current` for `7.3` in URLs.
