# Neos CMS | Docker image
[![Circle CI](https://circleci.com/gh/million12/docker-typo3-neos/tree/master.svg?style=svg)](https://circleci.com/gh/million12/docker-typo3-neos/tree/master)

This is the [million12/typo3-neos](https://registry.hub.docker.com/u/million12/typo3-neos/) Docker image with pre-packaged [Neos CMS](http://neos.io/).


## Available versions

* `million12/typo3-neos:latest` - latest **dev-master** version from Neos _master_ branch
* `million12/typo3-neos:2.0-dev` - latest **`2.0` development** version  from Neos _2.0_ branch
* `million12/typo3-neos:2.0` - latest **`2.0` stable** version

It is a demo of how you can build your own Neos image, perhaps from your own (private) repository, using the base [million12/typo3-flow-neos-abstract](https://github.com/million12/docker-typo3-flow-neos-abstract) image. Please read extensive documentation there to learn more.
 
This container contains PHP/Nginx installed/configured (see [million12/nginx](https://github.com/million12/docker-nginx) and [million12/nginx-php](https://github.com/million12/docker-nginx-php) base images. You only need a container with database. See the instructions below.

#### Switch to different version

You can easily switch to different version by setting Docker env variables `T3APP_BUILD_BRANCH=[branch-or-tag-name]` together with `T3APP_ALWAYS_DO_PULL=true` and then (re)starting the container. Learn more about it [here](https://github.com/million12/docker-typo3-flow-neos-abstract#build-variables). See the [neos.io](http://neos.io/) website or [git repository](https://github.com/neos/neos-base-distribution) for info about latest stable version.


## Usage

By default, the container starts and fully configure Neos, incl. importing default site package and creating the admin user (login: `admin`, password: `password`).

Launch required containers:

```
docker run -d --name=db --env="MARIADB_PASS=my-pass" million12/mariadb
docker run -d --name=neos -p=8080:80 --link=db:db --env="T3APP_VHOST_NAMES=neos dev.neos" million12/typo3-neos
```

You can inspect how the Neos provisioning is going on using `docker logs -f neos`. When you see something like `nginx entered RUNNING state`, you are ready to go.

Don't forget to **map vhost name(s)**, provided in above command via `T3APP_VHOST_NAMES` env variable, on your local machine. Do this by adding following line to your `/etc/hosts` file:  
```
YOUR_DOCKER_IP neos dev.neos
```

**Now go to [http://neos:8080/](http://neos:8080/)** (or respectively http://dev.neos:8080/ for *Development* environment) and enjoy playing with Neos CMS!

#### Development

Launch separate SSH container and link it with running `neos` container:
``` 
docker run -d --name=dev --link=db:db --link=neos:web --volumes-from=neos -p=1122:22 --env="IMPORT_GITHUB_PUB_KEYS=your-github-username-here" million12/php-app-ssh
```  
Please provide your GitHub username to `IMPORT_GITHUB_PUB_KEYS` env variable; your public SSH key will be imported from there. After container is launched, your key will be automatically added inside container and you can log in using **www user**:  
```
ssh -p 1122 www@YOUR_DOCKER_IP
```

You will find Neos CMS in `typo3-app` directory (by default). You can do all `./flow` commands from there, upload files via SFTP, in general: do all development using it. Learn more about the SSH container on [million12/docker-php-app-ssh](https://github.com/million12/docker-php-app-ssh) repository page.

Note: the good part is that this "side SSH container" is build on top of [million12/nginx-php](https://github.com/million12/docker-nginx-php) image, exactly the same which is used for [million12/typo3-flow-neos-abstract](https://github.com/million12/docker-typo3-flow-neos-abstract). Therefore you can be sure you have the same PHP configuration and the same software as inside container running Neos. In practise terms it means: no weird issues due to differences in environments.


## Usage with docker-compose

Instead of manually launching all containers like described above, you can use [Docker Compose](https://docs.docker.com/compose/). Docker Compose is an orchestration tool and it is very easy to use. If you do not have it yet, install it first. 

The [docker-compose.yml](docker-compose.yml) config file is already provided, so you can **launch Neos project with just one command**:  
```
docker-compose up [-d]
```

And you're done.

You can uncomment the `dev` container if you need SSH access. Remember to supply your GitHub username to `IMPORT_GITHUB_PUB_KEYS` env variable. Provided account has to have your public SSH key added.


## Running tests

It is very easy to run unit, functional and Behat tests against Neos with this container. Because some of Neos tests require Selenium with Firefox, we will launch [million12/php-testing](https://github.com/million12/docker-php-testing) image, link it with running Neos container and run the tests from there. Container million12/php-testing it is based on the same image with PHP as our Neos container, so we can be sure we are working in the same environment.

Here's how you can run all Neos test suites:  
```
docker run -d --name=db --env="MARIADB_PASS=my-pass" million12/mariadb
docker run -d --name=neos-testing --link=db:db --env="T3APP_VHOST_NAMES=behat.dev.neos" --env="T3APP_DO_INIT_TESTS=true" --env="T3APP_DO_INIT=false" million12/typo3-neos

# Wait till Neos container is fully provisioned (see 'docker logs -f neos-testing'). Then launch tests:
docker run -ti --volumes-from=neos-testing --link=neos-testing:web --link=db:db million12/php-testing "
    echo \$WEB_PORT_80_TCP_ADDR \$WEB_ENV_T3APP_VHOST_NAMES >> /etc/hosts && cat /etc/hosts && \
    su www -c \"
      cd ~/typo3-app && \
      echo -e '\n\n======== RUNNING NEOS TESTS =======\n\n' && \
      bin/phpunit -c Build/BuildEssentials/PhpUnit/UnitTests.xml && \
      bin/phpunit -c Build/BuildEssentials/PhpUnit/FunctionalTests.xml && \
      bin/behat -c Packages/Application/TYPO3.Neos/Tests/Behavior/behat.yml
    \"
"
```  
and you should see all tests nicely passing ;-)

## Authors

Author: Marcin Ryzycki (<marcin@m12.io>)  

---

**Sponsored by [Prototype Brewery](http://prototypebrewery.io/)** - the new prototyping tool for building highly-interactive prototypes of your website or web app. Built on top of [Neos CMS](https://www.neos.io/) and [Zurb Foundation](http://foundation.zurb.com/) framework.
