# Jenkins DevOps Docker Image for DevOps CI/CD

Builds a Docker image from latest [`jenkins:alpine`](https://hub.docker.com/_/jenkins) Docker image. Installs common DevOps tooling. Jenkins will be running on [`http://localhost:8083`](http://localhost:8083), by default.

![Jenkins DevOps Docker Image Architecture](architecture.png)

## Installed Tools

Based on latest packages as of 3/24/2017 build:

- [AWS CLI](https://aws.amazon.com/cli/) v1.11.66
- [git](https://git-scm.com/)
- [HashiCorp Packer](https://www.packer.io/) v0.12.3
- [HashiCorp Terraform](https://www.terraform.io/) v0.9.1
- [jq](https://stedolan.github.io/jq/) v1.5
- [OpenNTPD](http://www.openntpd.org/) (time sync)
- [pip3](https://pip.pypa.io/en/stable/#)
- [Python3](https://www.python.org/) v3.5.2
- [tzdata](https://www.iana.org/time-zones) (time sync)

## Creating Image

### Adding Jenkins Plugins

The `Dockerfile` loads plugins from the `plugin.txt`. Currently, it installs two backup plugins. You can add more plugins to this file, before building Docker image. See the Jenkins [Plugins Index](https://plugins.jenkins.io/) for more.

```text
thinBackup:1.9
backup:1.6.1
```

### Create Image

Create the new `garystafford/jenkins-devops:latest` image from the Dockerfile.

```bash
image="garystafford/jenkins-devops"
docker build -t ${image}:latest .
```

## Using the Docker Image

### Preliminary Steps

Delete previous Jenkins container

```bash
docker rm -f jenkins-devops
```

Create bind-mounted `jenkins_home` directory on host

```bash
mkdir -p /tmp/jenkins_home/
```

Backup process with Jenkins backup plugin. Backups will be placed in the bind-mounted host directory.

```bash
mkdir -p /tmp/backup/hudson
# docker exec -it jenkins-devops mkdir -p /tmp/backup/hudson
```

### Run the Container

Run new container from `garystafford/jenkins-devops:latest` image

```bash
docker run -d \
  --name jenkins-devops \
  -p 8083:8080 \
  -p 50000:50000 \
  -v /tmp/jenkins_home:/var/jenkins_home \
  -v /tmp/backup/hudson:/tmp/backup/hudson \
  garystafford/jenkins-devops:latest
```

Check container log for issues

```bash
docker logs jenkins-devops --follow
```

### AWS SSL Keys

Copy any required AWS SSL key pairs to bind-mounted `jenkins_home` directory.

```bash
mkdir -p /tmp/jenkins_home/.ssh

# used for git SCM Sync plugin
cp ~/.ssh/id_rsa /tmp/jenkins_home/.ssh

# used for Consul cluster project
cp ~/.ssh/consul_aws_rsa* /tmp/jenkins_home/.ssh
```

### AWS Credentials

Copy any required AWS credentials to bind-mounted `jenkins_home` directory

```bash
# used to connect to AWS with Packer/Terraform
cp ~/credentials/jenkins_credentials.env /tmp/jenkins_home/
```

## Troubleshooting

Fix time skew with container time:

```bash
docker run -it --rm --privileged \
  --pid=host debian nsenter -t 1 -m -u -n -i \
  date -u $(date -u +%m%d%H%M%Y)
```

## References

- [Jenkins by Docker](https://store.docker.com/images/d55eda09-d7f0-47b0-8780-3407f2f9142c?tab=description)
- [SCM Sync configuration plugin](https://wiki.jenkins-ci.org/display/JENKINS/SCM+Sync+configuration+plugin)
- [thinBackup](https://wiki.jenkins-ci.org/display/JENKINS/thinBackup)
- [Backup Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Backup+Plugin)
