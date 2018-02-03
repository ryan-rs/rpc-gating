# Local Jenkins and JJB deploy
This setup is based on the following blog post:
https://technologyconversations.com/2017/06/16/automating-jenkins-docker-setup/

This directory contains the tooling to run Jenkins with the rpc-gating jobs
deployed locally via a docker swarm service. Running the service should
perform the following:
  * Build a docker image containing Jenkins with the defined set of Jenkins plugins.
  * Deploy the rpc-gating jobs to the Jenkins instance via jenkins job builder.

WIP: Ideally, the docker build would perform all of the operations below. At
the moment, deploying the rpc_jobs fails when executed in the Dockerfile, so
it remains a manual exercise for the user.

## Usage
All commands shown below are executed from within the root of your rpc-gating
checkout.

### Build Jenkins Docker Image
```
docker image build -t rpc-gating/jenkins_sandbox -f docker-jenkins-sandbox .
```

### Deploy Jenins Service
```
docker swarm init
echo "admin" | docker secret create jenkins-user -
echo "admin" | docker secret create jenkins-pass -
docker stack deploy -c jenkins_sandbox/jenkins.yml jenkins
```

### Deploy rpc_jobs to Jenkins
```
virtualenv venv_jenkins
. venv_jenkins/bin/activate
pip install -r jenkins_sandbox/requirements.txt
cat > jjb.ini <<CONF
[jenkins]
url=http://localhost:8080
user=admin
password=admin
CONF
jenkins-jobs --conf jjb.ini update -r rpc_jobs
```

### Decommission
When you are finished with the Jenkins instance, perform the following to
decommission the setup.
```
docker stack rm jenkins
docker secret rm jenkins-user
docker secret rm jenkins-pass
deactivate
rm -r venv_jenkins
rm jjb.ini
```
