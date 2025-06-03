>This document demonstrates how to use Ansible to automate Docker operations within a Dockerized environment. It includes tasks for installing Docker, pulling base images, building a custom Nginx image, provisioning and managing containers, and performing clean-up. The setup leverages `DOCKER_HOST=tcp://docker:2375` to interact with a Docker daemon running inside another container.

`install-docker.sh`
```bash
sudo apt update
sudo apt install -y docker.io
pip3 install docker
```

`envdocker`
```bash
export DOCKER_HOST=tcp://docker:2375
```

`source envdocker` - required to be run on this example because we are running a docker container install a docker container.

**Building image with customized nginx:**
```yaml
---
- hosts: ubuntu-c
  tasks:
    - name: Pull images
      docker_image:
        docker_host: tcp://docker:2375
        name: "{{ item }}"
        source: pull
      with_items:
        - centos
        - ubuntu
        - redis
        - nginx
        # n.b. large image, >1GB
        - wernight/funbox

    - name: Create a customised index.html
      copy:
        dest: /shared/index.html
        mode: 0644
        content: |
          Customised page for nginxcustomised

    - name: Create a customised Dockerfile
      copy:
        dest: /shared/Dockerfile
        mode: 0644
        content: |
          FROM nginx
          COPY index.html /usr/share/nginx/html/index.html

    - name: Build a customised image
      docker_image:
        docker_host: tcp://docker:2375
        name: nginxcustomised:latest
        source: build
        build:
          path: /shared
          pull: yes
        state: present
        force_source: yes

    - name: Create an nginxcustomised container
      docker_container:
        docker_host: tcp://docker:2375
        name: containerwebserver
        image: nginxcustomised:latest
        ports:
          - 80:80
        container_default_behavior: no_defaults
        recreate: yes
...
```

**Example of connecting to running container as a target**
```yaml
---
- hosts: ubuntu-c
  tasks:
    - name: Pull python image
      docker_image:
        docker_host: tcp://docker:2375
        name: python:3.8.5
        source: pull

    - name: Create 3 python containers
      docker_container:
        docker_host: tcp://docker:2375
        name: "python{{ item }}"
        image: python:3.8.5
        container_default_behavior: no_defaults
        command: sleep infinity
      with_sequence: 1-3
- hosts: containers
  gather_facts: False
  
  tasks:
    - name: Ping containers
      ping:
...
```
Container needs to be running and have python installed.

**Clean-up:**
```yaml
---
- hosts: ubuntu-c
  tasks:
    - name: Remove old containers
      docker_container:
        docker_host: tcp://docker:2375
        name: "{{ item }}"
        state: absent
        container_default_behavior: no_defaults
      with_items:
        - containerwebserver
        - python1
        - python2
        - python3

    - name: Remove images
      docker_image:
        docker_host: tcp://docker:2375
        name: "{{ item }}"
        state: absent
      with_items:
        - centos
        - ubuntu
        - redis
        - nginx
        - wernight/funbox
        - nginxcustomised
        - python:3.8.5

    - name: Remove files
      file:
        path: "{{ item }}"
        state: absent
      with_items:
        - /shared/Dockerfile
        - /shared/index.html
...
```