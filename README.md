[![Build Status](https://travis-ci.org/ansible/ansible-container.svg?branch=develop)](https://travis-ci.org/ansible/ansible-container)
[![Code Coverage](https://codecov.io/gh/ansible/ansible-container/coverage.svg)](https://codecov.io/gh/ansible/ansible-container)

# Ansible Container

Ansible Container is a tool for building Docker images and orchestrating containers using Ansible playbooks.

This fork adds the following changes to Ansible Container.

- When deploying to k8s, allow adding more options in the deployment's container spec. Eg a readinessProbe can be added as follows:
```yaml
k8s:
  service:
    type: NodePort
    session_affinity: ClientIP
  deployment:
    containers:
      readiness_probe:
        http_get:
          path: /
          port: 8000
          scheme: HTTP
        initial_delay_seconds: 120
        period_seconds: 60
```

- Use the default namespace in a k8s deploy if the `container.yml` specifies `default` as the name of the k8s_namespace, name attribute. Eg:
```yaml
  k8s_namespace:
    name: default
```
- In the above case, the k8s objects will be created in the default namespace and the project_name will be appended to all object names. This allows same service but different projects to live in the k8s default namespace. This is useful if you want to use one k8s ingress for all services and avoid using (and paying) for more loadbalancers in a cloud provider like GCE for example. The project_name can be specified in `container.yml` or passed as a flag to `ansible-container build` with `--project-name` or `-n` option. See `ansible-container --help`

- In relation to the above, when destroying with `ansible-container --engine k8s destroy`, this version will not destroy by removing the namespace but will destroy each object individually

- Allow specifying a per-service or per-container use of local python during build in addition to the global `--use-local-python` flag. See that flag for details.
  This can be specified as:
```yaml
  web:
    from: "python:2.7.14-jessie"
    local_python: True
    ports:
      - "80:80"
    command: ["/usr/bin/dumb-init", "/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
    dev_overrides:
      environment:
        - "DEBUG=1"
```

**Note:** You will have to install this from source and build a conductor image based on this version. See the install instructions in this document. Otherwise, ansible-container will use the default conductor images that contain the original ansible-container project

## How it works

Use Ansible Container to manage the container lifecycle from development, through testing, to production:

* `ansible-container init`

  Creates a directory *ansible* with files to get you started. Read the comments, and edit to suit your needs.

* `ansible-container install`

  Downloads Ansible-Container-ready roles from [Ansible Galaxy](https://galaxy.ansible.com), and installs them in your project.

* `ansible-container build`

  Creates images from your Ansible playbooks.

* `ansible-container run`

  Launches the containers specified in the orchestration document, *container.yml*, for testing the built images. The 
  format of *container.yml* is nearly identical to Docker Compose.

* `ansible-container deploy`

  Pushes the project's container images to a registry of your choice, and generates a playbook capable of deploying the project on a supported cloud provider.

## Installing

~~Install using *pip*, the Python package manager:~~

    $ sudo pip install ansible-container[docker,openshift]
    
~~Or, to install without root privileges, use [virtualenv](https://virtualenv.pypa.io/en/stable/) to first create a 
Python sandbox:~~
    
    $ virtualenv ansible-container
    $ source ansible-container/bin/activate
    $ pip install ansible-container[docker,openshift]

~~For more details, prerequisite, and instructions on installing the latest development release, please view our 
[Installation Guide](https://docs.ansible.com/ansible-container/installation.html).~~

Installing from Source:

    $ git clone https://github.com/bituls/ansible-container.git
    $ cd ansible-container
    $ sudo pip install -e .[docker,openshift]
    
Build the conductor container

    $ python ./bakery.py
    
Or, to build a conductor image based on a specific base image eg. debian-jessie

    $ BASE_DISTRO='debian:jessie' python ./bakery.py
    
Do not worry if the conductor image fails to push to docker. You already have the conductor images locally.

Alternatively, instead of building your own base image, just tell ansible-container to run the installed version inside the default conductor image by adding `--dev` option when running ansible-container. Eg. to build use:

    $ ansible-container --dev build

## Getting started

For examples and a quick tour of Ansible Container visit [Getting Started](http://docs.ansible.com/ansible-container/getting_started.html) at our docs site.

Visit the [Ansible Container Demo](https://ansible.github.io/ansible-container-demo/) for a complete walk-through of managing an application from development through cloud deployment.

## Get Involved

* Visit [Community Information and Contributing](https://docs.ansible.com/ansible-container/community/index.html) 
  for all kinds of ways to contribute to and interact with the project. We welcome your feedback and ideas!
* Review [CONTRIBUTORS.md](./CONTRIBUTORS.md), if you're considering submitting code.
* [Join the  mailing list](https://groups.google.com/forum/#!forum/ansible-container)
* [Open an issue](https://github.com/ansible/ansible-container/issues)
* Join the #ansible-container channel on irc.freenode.net.  

## Branch Information

 * The *develop* branch is the release actively under development.
 * The *master* branch corresponds to the latest stable release available at [PyPi](https://pypi.org/project/ansible-container/).
 * Submit pull requests for bug fixes and new features to *develop*.
 * View [the roadmap](./ROADMAP.rst) for a list of features currently under development.
 * Contributors welcome! Get started by reviewing [CONTRIBUTORS.md](./CONTRIBUTORS.md).

## Authors

View [AUTHORS](./AUTHORS) for a list contributors to Ansible Container. Thanks everyone!

Ansible Container is an [Ansible by Red Hat](https://ansible.com) sponsored project.
