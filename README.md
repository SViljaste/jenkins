# JENKINS SETUP #

## Jenkins UI:

- WebURL: http://localhost:8081/
- User: admin
- Password: !READ doc "Create Jenkins container"!

## Step by step setup before creating Jenkins container

```
mkdir -p /opt/jenkins_home && chown 1000:1000 /opt/jenkins_home

KEY="$(cat ./id_rsa.pub)"; ID="1001"; USERNAME="jenkins-slave"; NAME="Jenkins Slave"; KEY="${KEY}"; if grep -wq "${ID}\|${USERNAME}" /etc/group; then echo "${USERNAME} või GID ${ID} grupiga on midagi"; else groupadd -g ${ID} ${USERNAME}; fi; if grep -wq "${ID}\|${USERNAME}" /etc/passwd; then echo "${USERNAME} või UID ${ID} kasutajaga on midagi"; id ${USERNAME}; fi; if grep -wq 'sshusers' /etc/group; then echo "sshusers grupp on olemas"; grupid="sshusers"; echo $grupid; useradd -c "${NAME}" -s /bin/bash -u ${ID} -g ${ID} -G ${grupid} -d /home/${USERNAME} -m ${USERNAME} && mkdir /home/${USERNAME}/.ssh && echo "${KEY}" > /home/${USERNAME}/.ssh/authorized_keys && chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.ssh && chmod -R 700 /home/${USERNAME}/.ssh; id ${USERNAME}; else useradd -c "${NAME}" -s /bin/bash -u ${ID} -g ${ID} -d /home/${USERNAME} -m ${USERNAME} && mkdir /home/${USERNAME}/.ssh && echo "${KEY}" > /home/${USERNAME}/.ssh/authorized_keys && chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.ssh && chmod -R 700 /home/${USERNAME}/.ssh; id ${USERNAME}; fi;

cat ./id_rsa > /home/jenkins-slave/.ssh/id_rsa

chmod 600 /home/jenkins-slave/.ssh/id_rsa && chown jenkins-slave:jenkins-slave /home/jenkins-slave/.ssh/id_rsa
mkdir /opt/jenkins_slave && chown -R jenkins-slave:jenkins-slave /opt/jenkins_slave -R

usermod -aG docker jenkins-slave

wget -O /opt/OpenJDK21U-jdk_x64_linux_hotspot_21.0.3_9.tar.gz https://github.com/adoptium/temurin21-binaries/releases/download/jdk-21.0.3%2B9/OpenJDK21U-jdk_x64_linux_hotspot_21.0.3_9.tar.gz && tar -xvf /opt/OpenJDK21U-jdk_x64_linux_hotspot_21.0.3_9.tar.gz -C /opt/ && rm -f /opt/OpenJDK21U-jdk_x64_linux_hotspot_21.0.3_9.tar.gz

ln -s /opt/jdk-21.0.3+9/bin/java /usr/local/bin/java

sudo tee /etc/profile.d/java.sh<<EOM
JAVA_HOME="/opt/jdk-21.0.3+9"
export PATH=$JAVA_HOME/bin:$PATH
EOM

source /etc/profile.d/java.sh
```

## Create Jenkins container

`docker compose -f docker-compose.yml up -d --build --force-recreate jenkins`

Login to Jenkins

After creating a container with `docker compose -f docker-compose.yml up -d --build --force-recreate jenkins` you're required to give temporary token for server input from UI.
After container is up and running execute: `docker logs jenkins -f` and fetch that token!

## Install missing plugins

```
Bitbucket Plugin
Version241.v6d24a_57f9359
Integrates with BitBucket
Report an issue with this plugin


Bitbucket Push and Pull Request
Version3.0.1
Integrates with Bitbucket Cloud (rest api version >=2.0) Server triggering on push and pull requests.
Report an issue with this plugin


Bitbucket Server Integration
Version4.0.0
This plugin integrates Bitbucket Server with Jenkins.
Report an issue with this plugin


Command Agent Launcher Plugin
Version107.v773860566e2e
Allows agents to be launched using a specified command.
Report an issue with this plugin


Copy Artifact
Version722.v0662a_9b_e22a_c
Adds a build step to copy artifacts from another project.
Report an issue with this plugin
This plugin is up for adoption! We are looking for new maintainers. Visit our Adopt a Plugin initiative for more information.


Embeddable Build Status
Version467.v4a_954796e45d
This plugin adds the embeddable build status badge to Jenkins so that you can easily hyperlink/show your build status from elsewhere.
Report an issue with this plugin


Generic Webhook Trigger
Version2.0.0
Can receive any HTTP request, extract any values from JSON or XML and trigger a job with those values available as variables. Works with GitHub, GitLab, Bitbucket, Jira and many more.
Report an issue with this plugin


Job DSL
Version1.87
This plugin allows Jobs and Views to be defined via DSLs
Report an issue with this plugin
This plugin is up for adoption! We are looking for new maintainers. Visit our Adopt a Plugin initiative for more information.


JSch dependency plugin
Version0.2.16-86.v42e010d9484b_
Jenkins plugin that brings the JSch library as a plugin dependency, and provides an SSHAuthenticatorFactory for using JSch with the ssh-credentials plugin.
Report an issue with this plugin


Mercurial plugin
Version1260.vdfb_723cdcc81
This plugin integrates Mercurial SCM with Jenkins. It includes repository browsing support for hg serve/hgweb, as well as hosted services like Google Code. Features include guaranteed clean builds, named branch support, module lists, Mercurial tool installation, and automatic caching.
Report an issue with this plugin
This plugin is up for adoption! We are looking for new maintainers. Visit our Adopt a Plugin initiative for more information.


Multiple SCMs plugin
Version0.8
This plugin enables the selection of multiple source code management systems for a build. For example, it enables checking out the source code from one SCM while checking out legacy or third-party code from another.
Report an issue with this plugin
This plugin is deprecated. In general, this means that it is either obsolete, no longer being developed, or may no longer work. Learn more.


Oracle Java SE Development Kit Installer Plugin
Version73.vddf737284550
Allows the Oracle Java SE Development Kit (JDK) to be installed via download from Oracle's website.
Report an issue with this plugin


Popper.js 2 API Plugin
Version2.11.6-4
Provides Popper.js for Jenkins Plugins. Popper can easily position tooltips, popovers or anything else with just a line of code.
Report an issue with this plugin
This plugin is deprecated. In general, this means that it is either obsolete, no longer being developed, or may no longer work. Learn more.


Slack Notification
Version684.v833089650554
Integrates Jenkins with Slack, allows publishing build statuses, messages and files to Slack channels.
Report an issue with this plugin


SSH Agent
Version346.vda_a_c4f2c8e50
This plugin allows you to provide SSH credentials to builds via a ssh-agent in Jenkins.
Report an issue with this plugin


SSH server
Version3.322.v159e91f6a_550
Adds SSH server functionality to Jenkins, exposing CLI commands through it.
Report an issue with this plugin
```

## Configure your repo ing GitHub Nginx_Linux_Docker_www

Add SSH public key (https://github.com/SViljaste/jenkins/blob/main/id_ed25519.pub) for your repo: https://github.com/silving1/Nginx_Linux_Docker_www

## Configure Jenkins UI

### Change security settings

http://localhost:8081/manage/configureSecurity/
Host Key Verification Strategy -> No verification
click Save

### Create nad configure new Jenkins agent

http://localhost:8081/manage/computer/
click New Node
Node name -> jenkins-slave
Type -> Permanent Agent
click Create
Number of executors -> 2
Remote root directory -> /opt/jenkins_slave
Labels -> jenkins-slave
Launch method -> Launch agents via SSH
Host -> 172.27.235.130
Credentials -> 
click Add then Jenkins
Kind -> SSH Username with private key
Scope -> System (Jenkins and nodes only)
Description -> SSH private key for Jenkins Slave connection user in local Jenkins Slave agent.
Username -> jenkins-slave
Private Key -> Enter directly
click Add
copy key from https://github.com/SViljaste/jenkins/blob/main/id_ed25519 and paste to Jenkins
click Add
Under "Launch method" -> "Credentials", choose: jenkins-slave
Host Key Verification Strategy -> Non verifying Verification Strategy
click Advanced
Connection Timeout in Seconds -> 60
Maximum Number of Retries -> 3
Seconds To Wait Between Retries -> 5
click Save (at the end of the page)

### Create new boilerplate for pipeline

click Dashboard
click New Item
Enter an item name -> www
choose "Pipeline"
click OK
Under Pipeline -> Definition, choose: Pipeline script from SCM
SCM -> Git
Repository URL -> ssh://git@github.com/silving1/Nginx_Linux_Docker_www.git
Credentials ->
click Add then Jenkins
Kind -> SSH Username with private key
Description -> SSH private key for github.com
Username -> jenkins-slave
Private Key -> Enter directly
click Add
copy key from https://github.com/SViljaste/jenkins/blob/main/id_ed25519 and paste to Jenkins
click Add
Under "SCM" -> "Repositories" -> "Credentials", choose: jenkins (SSH private key for github.com)


