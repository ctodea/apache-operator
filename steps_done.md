## To give a better vision about how this was done.
1. get `operator-sdk_linux_amd64` installed
2. have a look at an existing project, like https://github.com/prometheus-operator/prometheus-operator
    (choosed because has 5k+ stars on GH) to get the feel of how it should look like
3. Docs state:

        > The scaffolded operator has the following structure:
        > Memcached Custom Resource Definition, and a sample Memcached resource.
        > A “Manager” that reconciles the state of the cluster to the desired state
        > A reconciler, which is an Ansible Role or Playbook.
        > A watches.yaml file, which connects the Memcached resource to the memcached Ansible Role.
4. meet prerequisites:

        > Go through the installation guide.
        > User authorized with cluster-admin permissions.
        > An accessible image registry for various operator images (ex. hub.docker.com, quay.io) and be logged in in your command line environment.
        > example.com is used as the registry Docker Hub namespace in these examples. Replace it with another value if using a different registry or namespace.
        > Authentication and certificates if the registry is private or uses a custom CA.

    I'll need a cluster where to test this, mini-shift or similar would do the job.

5. getting my test cluster up. Using KVM as hypervisor. minishift-1.34.3-linux-amd64/
```sh
ctodea@TPL450:~/Downloads$ minishift start
-- Starting profile 'minishift'
-- Check if deprecated options are used ... OK
-- Checking if https://github.com is reachable ... OK
-- Checking if requested OpenShift version 'v3.11.0' is valid ... OK
-- Checking if requested OpenShift version 'v3.11.0' is supported ... OK
-- Checking if requested hypervisor 'kvm' is supported on this platform ... OK
-- Checking if KVM driver is installed ... FAIL # this went KABOOM
See the 'Setting Up the Virtualization Environment' topic (https://docs.okd.io/latest/minishift/getting-started/setting-up-virtualization-environment.html) for more information
ctodea@TPL450:~/Downloads$ sudo usermod -a -G libvirt $(whoami)
ctodea@TPL450:~/Downloads$ newgrp libvirt
ctodea@TPL450:~/Downloads$ sudo su
root@TPL450:/home/ctodea/Downloads# curl -L https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.10.0/docker-machine-driver-kvm-ubuntu16.04 -o /usr/local/bin/docker-machine-driver-kvm
root@TPL450:/home/ctodea/Downloads# chmod +x /usr/local/bin/docker-machine-driver-kvm
root@TPL450:/home/ctodea/Downloads#
ctodea@TPL450:~/Downloads$ systemctl is-active libvirtd
```

6. I see in the logs minishift it's installing 3.11, and
  > Note
  > Minishift runs OpenShift 3.x clusters. Due to different installation methods, OpenShift 4.x clusters are not supported. To run OpenShift 4.x locally, use CodeReady Containers.
So I'm switching from minishift to CodeReady Containers.

7. register at https://cloud.redhat.com/openshift/create/local and get CodeReady Containers.

8. cry. Actually I think 3.11 would suffice for now, i dont think it'll be a huge difference, so i'll start developing in 3.11 with minishift if crc does not cooperate.
```sh
ctodea@TPL450:~/Downloads$ sudo mv crc /usr/local/bin/
[sudo] password for ctodea:
ctodea@TPL450:~/Downloads$ crc config set consent-telemetry no
Successfully configured consent-telemetry to no
ctodea@TPL450:~/Downloads$ crc start
only 8.228GB of memory found (9.664GB required)
```

9. get crc running on desktop

10. follow howto's for operators with ansible. `ctodea@ctsm:~/git/apache-operator$ make docker-build docker-push IMG="quay.io/ctodea/apache-operator:v0.0.1"`

11. dev and test a lot

12. modify controller role to allow creating services and routes.

13. change image to ctodea@ctsm:~$ docker pull registry.access.redhat.com/rhscl/httpd-24-rhel7
Using default tag: latest
latest: Pulling from rhscl/httpd-24-rhel7
b77f42d650dc: Pull complete 
a858833a9239: Pull complete 
608083cad012: Pull complete 
a4914c5316bf: Pull complete 
Digest: sha256:0cd55bc708b656e63f1db25e7ae8079b7b41398f11b936c9e7cd17cd286251e7
Status: Downloaded newer image for registry.access.redhat.com/rhscl/httpd-24-rhel7:latest
registry.access.redhat.com/rhscl/httpd-24-rhel7:latest
ctodea@ctsm:~$ docker tag registry.access.redhat.com/rhscl/httpd-24-rhel7 rhscl/httpd-24-rhel7
ctodea@ctsm:~$ docker run -itd rhscl/httpd-24-rhel7
deee489c7bface51087ec5513e403a23c6925919265ac1bc1bc7ace832c1a7d1


oc import-image rhscl/httpd-24-rhel7 --from=registry.access.redhat.com/rhscl/httpd-24-rhel7 --confirm # to import it within my test cluster
https://catalog.redhat.com/software/containers/rhscl/httpd-24-rhel7/57ea8d049c624c035f96f42e?container-tabs=overview&gti-tabs=unauthenticatedc

14. add config map with HTML, service to ansible's deployment. Mount the html from the configmap into the container, to be the index.html

15. add cm for httpd.conf