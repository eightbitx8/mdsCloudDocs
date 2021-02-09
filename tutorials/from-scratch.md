# MDS Cloud "from scratch" Tutorial

In this tutorial you will be guided through installing and configuring all
components of MDS Cloud for local experimentation or development. All components
of the MDS Cloud environment, expect the CLI and Terraform provider, will be run
through docker in order to keep your system as clean as possible.

It is worth calling out that all components at the time of writing have only
been tested against Linux and MacOS. While it may be possible to run all of the
components in containers hosted on a Windows system no testing has been done by
anyone on the dev team and thus some dependencies or tools many not function
properly in that environment.

## First Time Setup

### Prerequisites

The following components must be installed and available on the system to
interact with MDS Cloud in-a-box. Feel free to install these items via their
direct links or your package manager of choice. If you can run `docker-compose`,
`node`, and `mds` from your command line you should be ready to run the run-book
below.

* [Docker](https://docs.docker.com/get-docker/)
* [Docker Compose](https://docs.docker.com/compose/)
* [Node and NPM](https://nodejs.org)
* [Terraform CLI](https://www.terraform.io/downloads.html)

### Installing the MDS Cloud CLI

One method of interacting with MDS Cloud is to use the command line interface in
order to create, manage, and inspect various assets within your deployment.
While it is planned to eventually have the MDS Cloud CLI available via NPM for
install we currently must use the NPM link feature to get the CLI available for
use.

* [Download or clone MDS CLI](https://github.com/MadDonkeySoftware/mdsCloudCli)
* Run the command `npm link` from the root of the source code.
  * This will be the directory containing the file `package.json`

### Installing the Terraform Provider

A custom Terraform provider is available for MDS Cloud to help you manage your
applications in the ecosystem. While configuring your own application is outside
the scope of this tutorial we will be using the MDS-Cloud-in-a-Box project as a
sample to see what MDS Cloud is currently capable of. Installing the Terraform
provider is as simple as copying a binary to the proper place on your system.

* [Download the latest release package](https://github.com/MadDonkeySoftware/mdsCloudTerraformProvider/releases)
  * Using a pre-release should be fine for this tutorial
* Create the directory needed for the provider
  * In the below command use the correct OS [architecture for your system](https://github.com/MadDonkeySoftware/mdsCloudTerraformProvider/blob/master/Makefile#L19-L30)
    * Ex: `linux_amd64` for 64-bit Ubuntu
  * `mkdir -p ~/.terraform.d/plugins/maddonkeysoftware.com/tf/mdscloud/0.2/{OS_ARCH}`
  * When complete a single file named `terraform-provider-mdscloud` should
    reside in the folder

### Configuring Docker to allow insecure registries

Docker by default will not use a registry that does not have a valid SSL 
certificate. The MDS Cloud in-a-box project uses an insecure registry in order
to house the images it builds for the serverless functions. The project utilizes
the host systems docker in order to push, pull, and build the images used. To
that end we must tell Docker that it is allowed to use insecure registries in
specific IP address ranges. The IP addresses used by docker are in a
"non-routable" IP range for added security.

* edit the `/etc/docker/daemon.json` on your host system.
  * If this file does not exist, create it.
* add/edit the below code block
  * The below networks are CIDR notation of the IPv4 non-routable address spaces
* restart docker

```json
{
  "insecure-registries": [
    "10.0.0.0/8",
    "172.16.0.0/12",
    "192.168.0.0/16"
  ]
}
```

### Running MDS Cloud In-A-Box

Now that we've gotten our dependencies in place it is time to get MDS Cloud
in-a-box up and running on our local system. MDS Cloud in-a-box comes with a
`prep.sh` script that will handle many things for you. Some of the high level
things include creating self-signed SSL certificates, creating database users,
and configuring each service with the proper system account credentials. Once
this is complete you will have an _almost_ functional stack. 

* [Download or clone MDS Cloud in-a-Box](https://github.com/MadDonkeySoftware/mdsCloudInABox)
* Execute the `prep.sh` script
* Open the `development-passwords.txt` in your editor of choice as we will
  reference this later.
* Run `docker-compose up` in a new terminal window and let it run in the
  background.

### Initial Configuration of MDS CLI

At this point we now have a MDS Cloud stack up and running but it is not yet
usable. There are some system items that we must create in order to allow some
services, like serverless functions, to function properly. First we will
configure our CLI to be able to speak with our MDS Cloud stack: 

* `mds config --env local wizard`
  * Account: `1001`
  * User Id: [your desired user name here]
  * Password: [your desired password here]
  * Allow self signed certificates: `yes`
  * [All URL values from here](https://github.com/MadDonkeySoftware/mdsCloudInABox#configure-mds-cli-environments)
    can be used for the various service values.
* `mds config --env localAdmin wizard`
  * Account: `1`
  * User Id: `mdsCloud`
  * Password: [Use value from `development-passwords.txt` for mdsCloud]
  * Allow self signed certificates: `yes`
  * [All URL values from here](https://github.com/MadDonkeySoftware/mdsCloudInABox#configure-mds-cli-environments)
    can be used for the various service values.

Now that you've gotten both environments configured it would be a great time to
double check that all the URLs you've entered are correct. You can run
`mds config --env local inspect all` or `mds config --env localAdmin inspect all`
to verify. Pay close attention to the need for "http" vs "https" and the port
numbers for each service URL.

We can double check that our configuration is correct and the MDS Cloud in-a-box
services are running properly by running a quick query. Try
`mds fs containers --env local` to verify that you do not get an HTTP/HTTPS
error.

### Initial System Configuration

Just one more big step before we can being to actually play with our MDS Cloud
deployment. In order for MDS Cloud to function various services utilize other
services in order to complete their work. One case in this is the serverless
functions service using the queue service. 

```sh
mds qs create --env localAdmin mdsCloudServerlessFunctions-FnProjectWork 
mds qs create --env localAdmin mdsCloudServerlessFunctions-FnProjectWork-dlq
mds qs create --env localAdmin mds-sm-pendingQueue
mds qs create --env localAdmin mds-sm-inFlightQueue
mds fs create --env localAdmin mdsCloudServerlessFunctionsWork
```

All of the above commands should complete without error. If you see any errors
at this stage it is definitely time to turn back and review what you have run
against the the steps so far.

### Account Registration and Running the Sample Application

Now we get to the fruit of our labor. In order to install our sample application
on our MDS Cloud deployment we must first create our user account. While this is
typically done through either the MDS CLI or other means we will be opting to
use a `curl` command to create our account. The reason we use this approach over
another is the ease in scripting our setup in the future so we can destroy and
recreate our stack quickly and easily. This is a topic that will be covered in
more detail later in our document though. 

First, lets go ahead and modify then execute this sample `curl` command. Before
running the command, make sure to update both the `userId` and `password` fields
with the values created in the [Initial Configuration of MDS CLI](#initial-configuration-of-mds-cli) step.

```sh
curl --insecure --request POST 'https://localhost:8081/v1/register' \
--header 'Content-Type: application/json' \
--data-raw '{
    "userId": "my-user",
    "email": "no@no.com",
    "password": "password",
    "friendlyName": "Local Developer",
    "accountName": "Local Development Account"
}'
```

The response from the command should include a success message with the account
id `1001`. If the account id is different we can update our MDS CLI local
configuration as well as the sample application to use the new value. We will
not cover that scenario in this document though as the likelihood of that
happening is quite small. 

Next, you will need to create a container that Terraform will use in order to
track state in a distributed way. If you are unfamiliar with Terraform do not
worry. The terraform configuration is already built for our sample application
so all we need to do is make a few small tweaks. It is worth noting that the
files we are editing will contain sensitive information, i.e. credentials. In
non-local-development environments it is strongly suggested that a strategy be
agreed upon for keeping this information encrypted and/or safe.

* Create the container via the MDS CLI
  * run `mds fs create --env local sample-tf-state` 
* [Download or clone the MDS Sample App](https://github.com/MadDonkeySoftware/mdsCloudSampleApp)

Once you have the sample application locally we will first need to update the
`test.tfvars` file in the `<root>/terraform` directory. Updating the credentials
here is our first objective. Next we move on to updating the root directory
`run-terraform` file with our credentials. Once these two files are up-to-date
we can execute the `run-terraform` script and watch Terraform install our
sample application!

### Testing the Sample Application

By this point you should have a fully functional test application deployed into
the MDS Cloud stack locally! Feel free to try out some of the commands below and
watch MDS Cloud and the sample application in action.

* Run the state machine
  * `mds sm list --env local`
  * `mds sm invoke --watch --env local orid:1:mdscloud:::1001:sm:1234 '{"name":"Frito"}'`
* Run a function 
  * `mds sf list --env local`
  * `mds sf invoke --input-type object --input '{"name":"Frito"}' --env local orid:1:mdscloud:::1001:sm:1234`

### Tearing Down

Once you're done playing around with your MDS Cloud deployment you can quickly
halt all the services by pressing `ctrl` + `c` or `docker-compose down`
depending on how you launched it. If you encounter any trouble starting the
stack in the future be sure to remove the volumes the docker compose file
specifies with the `docker-compose down -v` command. Alternatively you can
always run the `prep.sh` script again as it will handle this cleanup for you
before regenerating passwords and re-applying them to the local deployment.

## Tips and Tricks

### Scripting System Setup for Subsequent Runs

It can be handy to create a "helper" script to run after each run of the
`prep.sh` script. Assuming that the "local" and "localAdmin" environments of the
MDS CLI have been configured, you can simply run `prep.sh`, `docker-compose up`
then the below shell script to fully re-create a MDS Cloud in-a-box stack.

```sh
rm -f ~/.mds/cache

curl --insecure --request POST 'https://localhost:8081/v1/register' \
--header 'Content-Type: application/json' \
--data-raw '{
    "userId": "my-user",
    "email": "no@no.com",
    "password": "password",
    "friendlyName": "Local Developer",
    "accountName": "Local Development Account"
}'

mds qs create --env localAdmin mdsCloudServerlessFunctions-FnProjectWork 
mds qs create --env localAdmin mdsCloudServerlessFunctions-FnProjectWork-dlq
mds qs create --env localAdmin mds-sm-pendingQueue
mds qs create --env localAdmin mds-sm-inFlightQueue
mds fs create --env localAdmin mdsCloudServerlessFunctionsWork

mds fs create --env local sample-tf-state
```

### Configure Kibana to view logs

If you would like a little more insight into what is happening in the background
of your MDS Cloud stack, but would rather not read the docker logs on the
console, MDS Cloud includes an ELK stack. Below is a quick list of how to
configure Kibana so that logs emitted from MDS Cloud in-a-box can be filtered
and read more easily.

* Log in to the [Kibana UI](http://localhost:5601)
  * User: elastic
  * Password: changeme
* Use the left nav to go to the "Logs" section
* Click the "Settings" tab
* Under Indices, update the "Log Indices" field to include "logstash-*"
  * Ex: `filebeat-*,kibana_sample_data_logs*,logstash-*`
* Under Log Columns
  * Remove line with "Field"
  * Add Column, search for item "name".
  * Optionally drag "Message" field to bottom to make logs easier to read
* Optionally, it may be prudent to familiarize yourself with the
[Kibana Query Language](https://www.elastic.co/guide/en/kibana/current/kuery-query.html)

### Cleanup Docker Images from Builds

* Removing all orphaned docker images
  * `docker rmi $(docker images | grep '<none>' | awk '{print $3}')`

###### Last updated February 2021
