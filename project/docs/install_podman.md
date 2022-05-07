## What is Podman

Podman is a daemon-less, open-source tool to manage, deploy and build application containers. Podman is using Open Containers Initiative (OCI) Containers and Container Images, which means that containers crated with/for Docker or CRI-o will work with Podman as well and vice versa. It is coming with a command-line interface (CLI) which offers pretty much the same commands as Docker does. How similar are Podman commands to Docker? Let's put it this way: many Podman users just alias `docker` to `podman`. Like other runtimes, Podman also relies on an OCI compliant container runtime to interface with the operating system. Podamn also has a RESTful API to manage containers, offers automatic updates of containers, and has a fantastic `systemd` integration.

## Getting containers running

### Rootfull or rootless

As Podman is daemonless, running containers doesn't require running them with root but can be started with a user as well. Granted, rootless containers will have no access to networking and can't use privileged ports, but it is a great way to keep the system secure. For instance, if an application escapes the container ran as a user it will be able to cause less damage and would have limited access to the system than if it would run as root. Rootfull containers are recommended for running applications that require access over privileged ports like a public facing `Nginx reverse proxy. Even in that case, there could be some rules implemented in the firewall that forwards requests, but that's a different story

#### Rootless environment configuration 

The next steps will allow for the deployment of a *rootless* configuration with Podman. Important to note that these changes should apply for other distros as well: 
1. **Make sure that `fuse-overlayfs` is installed and it is available in the user `$PATH`.** (In openSUSE `fuse-overlayfs` is a dependency of Podman therefore when Podman is being installed it will install `fuse-overlayfs` as well.) 
``` 
# which fuse-overlayfs  
/usr/bin/fuse-overlayfs 
``` 
2. **Enable user namespaces.** On openSUSE this step should not be necessary. Check the number of available namespaces that are allowed in the system in `/proc/sys/user/max_user_namespaces`. To set a value or a different value you need to run `sysctl user.max_user_namespaces=28633` as **root**. 
3. **/etc/subuid and /etc/subgid configuration.** This is very important so instead of using my own words I move this out from the official Podman documentation and insert it here (source: [rootless tutorial](https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md))    

Rootless Podman requires the user running it to have a range of UIDs listed in /etc/subuid and /etc/subgid files.  The `shadow-utils` or `newuid` package provides these files on different distributions and  they must be installed on the system.  These files will need someone with root privileges on the system to add or update the entries withi
n them.  The following is a summarization from the [How does rootless Podman work?](https://opensource.com/article/19/2/how-does-rootless-podman-work) article by Dan Walsh on [opensource.com](https://opensource.com) 

Update the /etc/subuid and /etc/subgid with fields for each user that will be allowed to create containers that look like the following.  Note that the values for each user must be unique and without any overlap.  If there is an overlap, there is a potential for a user to use another’s namespace and they could corrupt it. 

``` 
cat /etc/subuid 
johndoe:100000:65536 
test:165536:65536 
``` 

The format of this file is USERNAME:UID:RANGE 

* username as listed in /etc/passwd or getpwent. 
* The initial uid allocated for the user. 
* The size of the range of UIDs allocated for the user. 

This means the user johndoe is allocated UIDS 100000-165535 as well as their standard UID in the /etc/passwd file.  NOTE: this is not currently supported with network installs.  These files must be available locally to the host machine.  It is not possible to configure this with LDAP or Active Directory. 

If you update either the /etc/subuid or the /etc/subgid file, you need to stop all the running containers owned by the user and kill the pause process that is running on the system for that user.  This can be done automatically by using the [podman system migrate](https://github.com/containers/podman/blob/master/docs/podman-system-migrate.1.md) command which will stop all the containers for the user and will kill the pause process. 

Rather than updating the files directly, the usermod program can be used to assign UIDs and GIDs to a user. 
```
usermod --add-subuids 200000-201000 --add-subgids 200000-201000 johndoe 
grep johndoe /etc/subuid /etc/subgid 
/etc/subuid:johndoe:200000:1001 
/etc/subgid:johndoe:200000:1001 
```

Here I would also add that you need to be careful with the `subuid` and `subgid` allocation as if the same `UID`/`GID` values are given to `users y` as well as for `user x` then `user y` will have access to the containers ran by `user x`.  

### Registries 

Podman supports every OCI compliant registries like Dockerhub, Quay.io, Treescale etc. To check, add, and remove the registries the system uses one can simply edit `/etc/containers/registries.conf`.

```
# For more information on this configuration file, see containers-registries.conf(5).
#
# Registries to search for images that are not fully-qualified.
# i.e. foobar.com/my_image:latest vs my_image:latest
[registries.search]
registries = ["registry.opensuse.org", "docker.io"]

# Registries that do not use TLS when pulling images or uses self-signed
# certificates.
[registries.insecure]
registries = []

# Blocked Registries, blocks the `docker daemon` from pulling from the blocked registry.  If you specify
# "*", then the docker daemon will only be allowed to pull from registries listed above in the search
# registries.  Blocked Registries is deprecated because other container runtimes and tools will not use it.
# It is recommended that you use the trust policy file /etc/containers/policy.json to control which
# registries you want to allow users to pull and push from.  policy.json gives greater flexibility, and
# supports all container runtimes and tools including the docker daemon, cri-o, buildah ...
[registries.block]
registries = []
```
On openSUSE you will have `registry.opensuse.org` added to your registries out of the box which is the official registry of the openSUSE project. You can add for example `quay.io` to this list to be able to search, pull, and push from/to Quay. When searching or pulling a container from a registry, `Podman` will check the registries from `/etc/containers/registries.conf` listed under `[registries.search]` block. `Podman` will check the URI in the order they're added in `[registries.search]`.

#### Login to registry

Just like with Docker you can log in to a registry with your username and password using `podman login [registry]`. Once logged in the credentials will be stored in `${XDG_RUNTIME_DIR}/containers/auth.json`. For the sake of automation you could make a copy of this `auth.json` file to be used in different scripts for building or deploying containerized applications, but keep the file with your credentials safe.

### Searching for a container

To search with Podman for an application use:
```
$ podman search nginx                                                                                                                                                     
INDEX         NAME                                                                                                                  DESCRIPTION                                      STARS   OFFICIAL  AUTOMATED
opensuse.org  registry.opensuse.org/devel/kubic/containers/container/opensuse/nginx                                                                                                  0                 
opensuse.org  registry.opensuse.org/home/dcassany/obs-service/container/kubic-nginx-ingress-controller                                                                               0                 
opensuse.org  registry.opensuse.org/home/dcassany/obs-service/container/kubic/nginx-ingress-controller                                                                               0                 
opensuse.org  registry.opensuse.org/home/dmolkentin/video/containers/opensuse-nginx                                                                                                  0                 
opensuse.org  registry.opensuse.org/home/illuusio/images/images_15.2/opensuse/ilmi/nginx-fpm                                                                                         0                 
opensuse.org  registry.opensuse.org/home/illuusio/images/images_15.2/opensuse/ilmi/nginx-ssl                                                                                         0                 
opensuse.org  registry.opensuse.org/home/illuusio/images/images_15.2/opensuse/ilmi/nginx-uwsgi                                                                                       0                 
opensuse.org  registry.opensuse.org/home/mook_work/scf/nginx-buildpack-release/images/images/nginx-buildpack                                                                         0                 
opensuse.org  registry.opensuse.org/home/onielsen/virtualization/kubic-container/15.0/containers/kubic/nginx                                                                         0                 
opensuse.org  registry.opensuse.org/home/pgeorgiadis/branches/devel/caasp/kubic-container/container/kubic/nginx-ingress-controller                                                   0                 
opensuse.org  registry.opensuse.org/home/sjamgade/helm/opensuse_leap_15.2/nginx                                                                                                      0                 
opensuse.org  registry.opensuse.org/opensuse/factory/arm/totest/containers/opensuse/nginx                                                                                            0                 
opensuse.org  registry.opensuse.org/opensuse/factory/powerpc/totest/containers/opensuse/nginx                                                                                        0                 
opensuse.org  registry.opensuse.org/opensuse/factory/powerpc/totest/images/opensuse/nginx                                                                                            0                 
opensuse.org  registry.opensuse.org/opensuse/factory/totest/containers/opensuse/nginx                                                                                                0                 
opensuse.org  registry.opensuse.org/opensuse/factory/zsystems/images/opensuse/nginx                                                                                                  0                 
opensuse.org  registry.opensuse.org/opensuse/factory/zsystems/totest/containers/opensuse/nginx                                                                                       0                 
opensuse.org  registry.opensuse.org/opensuse/nginx                                                                                                                                   0                 
docker.io     docker.io/library/nginx                                                                                               Official build of Nginx.                         14378   [OK]      
docker.io     docker.io/jwilder/nginx-proxy                                                                                         Automated Nginx reverse proxy for docker con...  1951              [OK]
```

To search in a specific repository:
```
$ podman search registry.opensuse.org/nginx                                                                                                                               
INDEX         NAME                                                                                                                  DESCRIPTION  STARS   OFFICIAL  AUTOMATED
opensuse.org  registry.opensuse.org/devel/kubic/containers/container/opensuse/nginx                                                              0                 
opensuse.org  registry.opensuse.org/home/dcassany/obs-service/container/kubic-nginx-ingress-controller                                           0                 
opensuse.org  registry.opensuse.org/home/dcassany/obs-service/container/kubic/nginx-ingress-controller                                           0                 
opensuse.org  registry.opensuse.org/home/dmolkentin/video/containers/opensuse-nginx                                                              0                 
opensuse.org  registry.opensuse.org/home/illuusio/images/images_15.2/opensuse/ilmi/nginx-fpm                                                     0                 
opensuse.org  registry.opensuse.org/home/illuusio/images/images_15.2/opensuse/ilmi/nginx-ssl                                                     0                 
opensuse.org  registry.opensuse.org/home/illuusio/images/images_15.2/opensuse/ilmi/nginx-uwsgi                                                   0                 
opensuse.org  registry.opensuse.org/home/mook_work/scf/nginx-buildpack-release/images/images/nginx-buildpack                                     0                 
opensuse.org  registry.opensuse.org/home/onielsen/virtualization/kubic-container/15.0/containers/kubic/nginx                                     0                 
opensuse.org  registry.opensuse.org/home/pgeorgiadis/branches/devel/caasp/kubic-container/container/kubic/nginx-ingress-controller               0                 
opensuse.org  registry.opensuse.org/home/sjamgade/helm/opensuse_leap_15.2/nginx                                                                  0                 
opensuse.org  registry.opensuse.org/opensuse/factory/arm/totest/containers/opensuse/nginx                                                        0                 
opensuse.org  registry.opensuse.org/opensuse/factory/powerpc/totest/containers/opensuse/nginx                                                    0                 
opensuse.org  registry.opensuse.org/opensuse/factory/powerpc/totest/images/opensuse/nginx                                                        0                 
opensuse.org  registry.opensuse.org/opensuse/factory/totest/containers/opensuse/nginx                                                            0                 
opensuse.org  registry.opensuse.org/opensuse/factory/zsystems/images/opensuse/nginx                                                              0                 
opensuse.org  registry.opensuse.org/opensuse/factory/zsystems/totest/containers/opensuse/nginx                                                   0                 
opensuse.org  registry.opensuse.org/opensuse/nginx                                                                                               0                 
```

## Pulling a container

To pull a container:
```
$ podman pull nginx                                                                                                                                                      Completed short name "nginx" with unqualified-search registries (origin: /etc/containers/registries.conf)
Trying to pull registry.opensuse.org/nginx:latest...
  name unknown
Trying to pull docker.io/library/nginx:latest...
Getting image source signatures
Copying blob f72584a26f32 done  
Copying blob 0732ab25fa22 done  
Copying blob 7125e4df9063 done  
Copying blob a076a628af6f done  
Copying blob d7f36f6fe38f done  
Copying config f6d0b4767a done  
Writing manifest to image destination
Storing signatures
f6d0b4767a6c466c178bf718f99bea0d3742b26679081e52dbf8e0c7c4c42d74
```
To pull from a specific registry:
```
$ podman pull registry.opensuse.org/opensuse/nginx                                                                                                                       
Trying to pull registry.opensuse.org/opensuse/nginx:latest...
Getting image source signatures
Copying blob fc491b4cab32 done  
Copying blob fd4952530c86 done  
Copying config 36fa073c57 done  
Writing manifest to image destination
Storing signatures
36fa073c57cc5251ac2df80de6a2e052dad9a10d6ffc9a9e65ba6eaa0ddea6ff
```
To pull from a registry authenticated with an `auth.json` file stored in `~/.secret`:
```
$ podman pull --authfile=~/.secret/auth.json registry.opensuse.org/opensuse/nginx                                                                                
Trying to pull registry.opensuse.org/opensuse/nginx:latest...
Getting image source signatures
Copying blob fc491b4cab32 done  
Copying blob fd4952530c86 done  
Copying config 36fa073c57 done  
Writing manifest to image destination
Storing signatures
19162aa1f28d9b92c87f788939bad1e3f6100ac8a1aeeed9448f642e8eb695d2
```
### Starting a container

Starting a container that runs in the background and exposing port 8000 to be accessible from the outside:
```
$ podman run -d -p 8000:8000 nginx                                                                                                      
41e3829d1c800d40feb9d88ac962d29c8256b60ec5747f732717893d93b0f35e
```
To list running containers:
```
[~] podman ps                                                                                                                                              
CONTAINER ID  IMAGE                           COMMAND               CREATED        STATUS            PORTS                   NAMES
41e3829d1c80  docker.io/library/nginx:latest  nginx -g daemon o...  3 seconds ago  Up 3 seconds ago  0.0.0.0:8000->8000/tcp  suspicious_feistel
```
This list provides a lot of information about running containers. 
To list every container, running or not:
```
$ podman ps -a
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS                      PORTS                   NAMES
81a872daad56  docker.io/library/nginx:latest  nix                   6 seconds ago   Exited (127) 6 seconds ago  0.0.0.0:8001->8001/tcp  wonderful_rubin
41e3829d1c80  docker.io/library/nginx:latest  nginx -g daemon o...  18 minutes ago  Up 18 minutes ago           0.0.0.0:8000->8000/tcp  suspicious_feistel
bc5030bffd6c  docker.io/library/nginx:latest  nginx -g daemon o...  18 minutes ago  Created                     0.0.0.0:80->8000/tcp    jolly_johnson
```
If needed the output format of `podman ps` can be manipulated with the `--format` flag. For example to list only the container ID and the image name:
```
[~] podman ps --format "{{.ID}} {{.Image}}"                                                                                                                           
41e3829d1c80 docker.io/library/nginx:latest
```
This pretty print containers to JSON. The following placeholders are available:
* .ID: Container ID                             
* .Image:Image Name/ID
* .ImageID: Image ID
* .Command: Quoted command used                      
* .CreatedAt: Creation time for container              
* .RunningFor: Time elapsed since container was started 
* .Status: Status of container
* .Pod: Pod the container is associated with     
* .Ports: Exposed ports                            
* .Size: Size of container
* .Names: Name of container                        
* .Labels: All the labels assigned to the container
* .Mounts: Volumes mounted in the container            
For more visit the `podman ps` man page.

#### Deleting containers and images
Running containers or images that are being used can't be deleted unless it is forced. The best course of action is to stop the container then delete it, and then delete the image.
```
[~] podman ps #List running containers                                                                                                                          
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS             PORTS                   NAMES
41e3829d1c80  docker.io/library/nginx:latest  nginx -g daemon o...  22 minutes ago  Up 22 minutes ago  0.0.0.0:8000->8000/tcp  suspicious_feistel
$ podman stop suspicious_feistel #stop container by name                                                                             
41e3829d1c800d40feb9d88ac962d29c8256b60ec5747f732717893d93b0f35e
$ podman rm suspicious_feistel #Delete container by name
41e3829d1c800d40feb9d88ac962d29c8256b60ec5747f732717893d93b0f35e
[~] podman rmi nginx #Delete container image, this fails as there are other containers still using it                      
Error: 1 error occurred:
        * could not remove image f6d0b4767a6c466c178bf718f99bea0d3742b26679081e52dbf8e0c7c4c42d74 as it is being used by 2 containers: image is being used
[~] podman rm jolly_johnson wonderful_rubin #Remove containers that are still using the nginx image                                    
81a872daad5661a7704e47c4f5025db502852cf7cc0aaaa439e71a9fa915e9b9
bc5030bffd6c604192c31911141d728378b69832cafbf81d97d976d59fc5f86d
[~] podman rmi nginx #Delete the image                                                                         
Untagged: docker.io/library/nginx:latest
Deleted: f6d0b4767a6c466c178bf718f99bea0d3742b26679081e52dbf8e0c7c4c42d74
$ podman images #List available, pulled images
REPOSITORY                                 TAG     IMAGE ID      CREATED      SIZE
registry.opensuse.org/opensuse/tumbleweed  latest  111d1befeae4  2 days ago   94.4 MB
registry.opensuse.org/opensuse/nginx       latest  36fa073c57cc  3 days ago   103 MB
docker.io/prom/prometheus                  latest  19162aa1f28d  2 weeks ago  174 MB
```

#### Managing images

To list all available images use `podman images`. This will provide a list of every image that is currently available in the system and ready to be used without needing to pull it from a registry. To delete an image use `podman rmi`. Visit the `podman images` man page for more.

### Podman Pods

The `pod` concept has been introduced in Kubernetes and Docker users usually don't even think of the possibility or benefits to run pods in a local runtime or a development environment. Pods are great if, for example, you need a database (DB) container that you don't want to bind to a routable network. In comes the idea of the `pod`. Instead of binding the DB to a routable network, you can bind it to `localhost` therefore other containers within the pod can connect to it using `localhost` as they share the network namespace. Another use case can be to group containers in pods.
 Every pod has an 'infra' container that functions as the namespace holder, stores data on port bindings, cgroup-parent values, and kernel namespaces are all assigned to the “infra” container. Once the pod is created these values can not be changed. The pod can also allow starting, stop, pause multiple containers at once just by doing the same as would with a single container, but instead by container the commands can be executed by pods resulting in the desired effect, but faster and a lot more comfortably.
For details on pod related cli commands visit the `podman pod` man page. 

#### Pod creation

You can create the pods by naming them and exposing the required ports. This way we're not required to setup containers individually in terms of publishing ports just adding the containers to them.   
```
$ podman pod create -n yaltb -p 8000:8000                                                                        
7244e77552d9bee402383a37906813b4194cef89c930c87d4d590d92fce827b0
$ podman pod list                                                                            
POD ID        NAME    STATUS   CREATED        INFRA ID      # OF CONTAINERS
7244e77552d9  yaltb   Created  8 seconds ago  4a696258e500  1
```
A little breakdown here:
* `-n` or `--name` will allow to set the name of the pod.
* `-p` or `--publish` will expose the desired port(s) that will be used to access container(s) from outside the host.
* `podman pod list` is listing the available pods with their basic information.
For more information visit the `podman pod create` man page.

#### Adding containers to a pod

Pods can hold as many containers as you want. To add containers just set the `--pod` flag when creating the container.
```
$ podman run -d --pod yaltb nginx                                                                                                                                         
6645234da2d18f60632fdcfed3a7825a81649c606dfeba70fec11b9d5479174d
$ podman pod list                                                                                                                                                         
POD ID        NAME    STATUS   CREATED         INFRA ID      # OF CONTAINERS
7244e77552d9  yaltb   Running  10 minutes ago  4a696258e500  2
$ podman ps -a --pod                                                                                                                                                      
CONTAINER ID  IMAGE                 COMMAND          CREATED         STATUS             PORTS                   NAMES               POD ID        PODNAME
6645234da2d1  nginx                 /usr/sbin/nginx  15 seconds ago  Up 15 seconds ago  0.0.0.0:8000->8000/tcp  clever_williamson   7244e77552d9  yaltb
4a696258e500  k8s.gcr.io/pause:3.2                   10 minutes ago  Up 15 seconds ago  0.0.0.0:8000->8000/tcp  7244e77552d9-infra  7244e77552d9  yaltb
```
Breaking it down:
* `podman run -d --pod yaltb nginx` will create and add an `nginx` container to the previously create `yaltb` pod. The pod has been defined by the `--pod` flag followed by the name of the pod. Notice that we didn't expose ports as they are already exposed in the created port. **Important:** network can't be configured when it is shared with a pod therefore you need to expose/publish ports upon pod creation.
* `podman ps -a --pod` will list every container that is running in pods.

#### Managing containers in pods

Containers in pods can be managed individually regardless they're running within a pod. Do check above for the options for managing containers. This means that if you need to remove a container from a pod 

#### Exporting pods

Pods can be exported with `podman generate` command. This would allow one to take the exported pod and deploy it the same way it is deployed on the local runtime on a different Podman server or even on a Kubernetes cluster.
```
$ podman generate kube yaltb                                                                                                                                              
# Generation of Kubernetes YAML is still under development!
#
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-2.2.1
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-02-05T05:28:08Z"
  labels:
    app: yaltb
  name: yaltb
spec:
  containers:
  - command:
    - /usr/sbin/nginx
    env:
    - name: PATH
      value: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    - name: TERM
      value: xterm
    - name: container
      value: podman
    - name: HOSTNAME
      value: yaltb
    image: nginx
    name: condescendinggalois
    ports:
    - containerPort: 8000
      hostPort: 8000
      protocol: TCP
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true
      capabilities: {}
      privileged: false
      readOnlyRootFilesystem: false
      seLinuxOptions: {}
    workingDir: /
  - command:
    - /usr/sbin/nginx
    env:
    - name: PATH
      value: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    - name: TERM
      value: xterm
    - name: container
      value: podman
    - name: HOSTNAME
      value: yaltb
    image: nginx
    name: cleverwilliamson
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true
      capabilities: {}
      privileged: false
      readOnlyRootFilesystem: false
      seLinuxOptions: {}
    workingDir: /
  restartPolicy: Never
status: {}
---
metadata:
  creationTimestamp: null
spec: {}
status:
  loadBalancer: {}

$ podman pod stop yaltb                                                                                                                                                   
7244e77552d9bee402383a37906813b4194cef89c930c87d4d590d92fce827b0
$ podman pod rm yaltb                                                                                                                                                     
7244e77552d9bee402383a37906813b4194cef89c930c87d4d590d92fce827b0

$ podman play kube yaltb_pod.yml                                                                                                                                          
Pod:
ce21bc0689409795800d7e0f699f4503b6be8f7f81bcc4fabc33e5da32bba164
Containers:
13e9f16b1aaaa7be1f19c87c21d99deaca3862d97293b98c20bf792d72738911
c5f8e4364b0e0caca047b3797ffc0aaf268460aee9249efc4d46259c4693eddd

[~] podman pod ps                                                                                                                                                           
POD ID        NAME    STATUS    CREATED         INFRA ID      # OF CONTAINERS
ce21bc068940  yaltb   Degraded  11 seconds ago  c78bebfb3cc3  3
[~] podman ps -a --pod                                                                                                                                                      
CONTAINER ID  IMAGE                 COMMAND          CREATED         STATUS                     PORTS                   NAMES                      POD ID        PODNAME
13e9f16b1aaa  nginx                 /usr/sbin/nginx  19 seconds ago  Up 19 seconds ago          0.0.0.0:8000->8000/tcp  yaltb-condescendinggalois  ce21bc068940  yaltb
c5f8e4364b0e  nginx                 /usr/sbin/nginx  19 seconds ago  Exited (1) 16 seconds ago  0.0.0.0:8000->8000/tcp  yaltb-cleverwilliamson     ce21bc068940  yaltb
c78bebfb3cc3  k8s.gcr.io/pause:3.2                   19 seconds ago  Up 19 seconds ago          0.0.0.0:8000->8000/tcp  ce21bc068940-infra         ce21bc068940  yaltb
[~] podman ps  --pod                                                                                                                                                        
CONTAINER ID  IMAGE                 COMMAND          CREATED         STATUS             PORTS                   NAMES                      POD ID        PODNAME
13e9f16b1aaa  nginx                 /usr/sbin/nginx  30 seconds ago  Up 30 seconds ago  0.0.0.0:8000->8000/tcp  yaltb-condescendinggalois  ce21bc068940  yaltb
c78bebfb3cc3  k8s.gcr.io/pause:3.2                   30 seconds ago  Up 30 seconds ago  0.0.0.0:8000->8000/tcp  ce21bc068940-infra         ce21bc068940  yaltb
 
```
Lets break down the above:
* `podman generate kube yaltb` will generated a `YAML` file which one can take to another server and `play` it providing the same exact pod that it is running on the local runtime.
* `podman pod stop yaltb` will stop the `yaltb` pod and all the containers in it.
* `podman pod rm yaltb` will delete the `yaltb` pod.
* `podman play kube yaltb_pod.yml` will start the exported pods based on the `YAML` file. The pod and the containers in the pod listed in the following commands.

### Podman and systemd

Podman has fantastic integration with `systemd`. Users are even able to run `systemd` **within** a container which makes perfect sense. Why would we have to use anything different inside a container than we already got used to outside of a container? Anyhow, we are not covering this, but managing containers with `systemd`. 
As we deploy more and more containers managing them using the Podman cli tool only may quickly become burdensome. It is much easier to manage the pods and containers with `systemd` for example to restart, stop, or to start them. Using `systemd` will also enable us to utilize `podman auto-update` for updating the containers.

#### Generating a systemd unit

Podman has the ability to generate the unit files for us which is probably a relief as it makes life incredibly comfortable. Criteria is that it outputs the generated units into a file using the container's name or ID. In this example I will go for the ID which I will cover why in the `Podman auto-update` section.
```
$ podman generate systemd --files --new admiring_franklin                                                                                                                 
/home/yaltb/container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.service   

$ cat container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.service                                                                                  
# container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.service
# autogenerated by Podman 2.2.1
# Fri Feb  5 12:52:43 WIB 2021

[Unit]
Description=Podman container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.service
Documentation=man:podman-generate-systemd(1)
Wants=network.target
After=network-online.target

[Service]
Environment=PODMAN_SYSTEMD_UNIT=%n
Restart=on-failure
ExecStartPre=/bin/rm -f %t/container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.pid %t/container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.ctr-id
ExecStart=/usr/bin/podman run --conmon-pidfile %t/container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.pid --cidfile %t/container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.ctr-id --cgroups=no-conmon -d -p 8000:8000 nginx
ExecStop=/usr/bin/podman stop --ignore --cidfile %t/container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.ctr-id -t 10
ExecStopPost=/usr/bin/podman rm --ignore -f --cidfile %t/container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.ctr-id
PIDFile=%t/container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.pid
KillMode=none
Type=forking

[Install]
WantedBy=multi-user.target default.target

```
Breaking all that down:
* `podman generate systemd` which will generate `systemd` units
* ` --files` which will output the generated unit to a file as `container-[$CONTAINERNAME]
* ` --new admiring_franklin` will allow `systemd` to create and run updated images followed by the container's name.

This unit file can be added to `systemd`, enabled and pretty much it. Podman and systemd will take care of everything following like starting the container on boot and updating it. When running a rootless Podman session:
```
$ mv container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.service ~/.config/systemd/user/
$ systemctl --user enable --now container-aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7.service
```

### Podman inspect and logs

To reveal the parameters a container or a pod has been created with we can use the `podman inspect` command followed by the name/id of the container or pod.
```
$ podman inspect admiring_franklin                                                                                                                                        
[
    {
        "Id": "aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7",
        "Created": "2021-02-05T12:52:23.83123341+07:00",
        "Path": "/usr/local/bin/entrypoint.sh",
        "Args": [
            "/usr/sbin/nginx"
        ],
        "State": {
            "OciVersion": "1.0.2-dev",
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 26725,
            "ConmonPid": 26714,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-02-05T12:52:24.009065815+07:00",
            "FinishedAt": "0001-01-01T00:00:00Z",
            "Healthcheck": {
                "Status": "",
                "FailingStreak": 0,
                "Log": null
            }
        },
        "Image": "36fa073c57cc5251ac2df80de6a2e052dad9a10d6ffc9a9e65ba6eaa0ddea6ff",
        "ImageName": "nginx",
        "Rootfs": "",
        "Pod": "",
        "ResolvConfPath": "/run/user/1000/containers/overlay-containers/aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7/userdata/resolv.conf",
        "HostnamePath": "/run/user/1000/containers/overlay-containers/aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7/userdata/hostname",
        "HostsPath": "/run/user/1000/containers/overlay-containers/aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7/userdata/hosts",
        "StaticDir": "/home/apinter/.local/share/containers/storage/overlay-containers/aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7/userdata",
        "OCIConfigPath": "/home/apinter/.local/share/containers/storage/overlay-containers/aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7/userdata/config.json",
        "OCIRuntime": "runc",
        "LogPath": "/home/apinter/.local/share/containers/storage/overlay-containers/aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7/userdata/ctr.log",
        "LogTag": "",
        "ConmonPidFile": "/run/user/1000/containers/overlay-containers/aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7/userdata/conmon.pid",
        "Name": "admiring_franklin",
        "RestartCount": 0,
        "Driver": "overlay",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "EffectiveCaps": [
            "CAP_AUDIT_WRITE",
            "CAP_CHOWN",
            "CAP_DAC_OVERRIDE",
            "CAP_FOWNER",
            "CAP_FSETID",
            "CAP_KILL",
            "CAP_MKNOD",
            "CAP_NET_BIND_SERVICE",
            "CAP_NET_RAW",
            "CAP_SETFCAP",
            "CAP_SETGID",
            "CAP_SETPCAP",
            "CAP_SETUID",
            "CAP_SYS_CHROOT"
        ],
        "BoundingCaps": [
            "CAP_AUDIT_WRITE",
            "CAP_CHOWN",
            "CAP_DAC_OVERRIDE",
            "CAP_FOWNER",
            "CAP_FSETID",
            "CAP_KILL",
            "CAP_MKNOD",
            "CAP_NET_BIND_SERVICE",
            "CAP_NET_RAW",
            "CAP_SETFCAP",
            "CAP_SETGID",
            "CAP_SETPCAP",
            "CAP_SETUID",
            "CAP_SYS_CHROOT"
        ],
        "ExecIDs": [],
        "GraphDriver": {
            "Name": "overlay",
            "Data": {
                "LowerDir": "/home/apinter/.local/share/containers/storage/overlay/9f76177964d37dd2b6b0a0d675d83279c746568e764856d39cbcd9124196d776/diff:/home/apinter/.local/share/containers/storage/overlay/b81706afabe1f2b78cf2592b4f177f245563e5a33819f4e04a9edff9e0ddbbd8/diff",
                "MergedDir": "/home/apinter/.local/share/containers/storage/overlay/4eda62ca4ff798cc5e7361e8ea0f9126aa12e91e676036245d90b3e0d790b632/merged",
                "UpperDir": "/home/apinter/.local/share/containers/storage/overlay/4eda62ca4ff798cc5e7361e8ea0f9126aa12e91e676036245d90b3e0d790b632/diff",
                "WorkDir": "/home/apinter/.local/share/containers/storage/overlay/4eda62ca4ff798cc5e7361e8ea0f9126aa12e91e676036245d90b3e0d790b632/work"
            }
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "c86360b3e3d6b0a0cdc60cf3cb4504a267933d154dae282a7caf760c7b599cdf",
                "Source": "/home/apinter/.local/share/containers/storage/volumes/c86360b3e3d6b0a0cdc60cf3cb4504a267933d154dae282a7caf760c7b599cdf/_data",
                "Destination": "/data",
                "Driver": "local",
                "Mode": "",
                "Options": [
                    "nodev",
                    "exec",
                    "nosuid",
                    "rbind"
                ],
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        "Dependencies": [],
        "NetworkSettings": {
            "EndpointID": "",
            "Gateway": "",
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "MacAddress": "",
            "Bridge": "",
            "SandboxID": "",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "8000/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8000"
                    }
                ]
            },
            "SandboxKey": "/run/user/1000/netns/cni-ca08b508-01ec-9f12-554f-109725a54e4f"
        },
        "ExitCommand": [
            "/usr/bin/podman",
            "--root",
            "/home/apinter/.local/share/containers/storage",
            "--runroot",
            "/run/user/1000/containers",
            "--log-level",
            "error",
            "--cgroup-manager",
            "cgroupfs",
            "--tmpdir",
            "/run/user/1000/libpod/tmp",
            "--runtime",
            "runc",
            "--storage-driver",
            "overlay",
            "--storage-opt",
            "overlay.mount_program=/usr/bin/fuse-overlayfs",
            "--events-backend",
            "journald",
            "container",
            "cleanup",
            "aaef566e500973a2d4e9df22208e4a74ae5ea961083adbbe6bb6e98fbb19f7e7"
        ],
        "Namespace": "",
        "IsInfra": false,
        "Config": {
            "Hostname": "aaef566e5009",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "TERM=xterm",
                "container=podman",
                "HOSTNAME=aaef566e5009",
                "HOME=/root"
            ],
            "Cmd": [
                "/usr/sbin/nginx"
            ],
            "Image": "nginx",
            "Volumes": null,
            "WorkingDir": "/",
            "Entrypoint": "/usr/local/bin/entrypoint.sh",
            "OnBuild": null,
            "Labels": {
                "org.openbuildservice.disturl": "obs://build.opensuse.org/openSUSE:Factory/images/d720a7068b306d15952c962f20a9524c-opensuse-nginx-image",
                "org.opencontainers.image.created": "2021-01-30T17:23:14.138895035Z",
                "org.opencontainers.image.description": "Image containing nginx as web server.",
                "org.opencontainers.image.title": "openSUSE nginx web server container",
                "org.opencontainers.image.url": "https://www.opensuse.org/",
                "org.opencontainers.image.vendor": "openSUSE Project",
                "org.opencontainers.image.version": "1.19.6-8.919",
                "org.opensuse.nginx.created": "2021-01-30T17:23:14.138895035Z",
                "org.opensuse.nginx.description": "Image containing nginx as web server.",
                "org.opensuse.nginx.disturl": "obs://build.opensuse.org/openSUSE:Factory/images/d720a7068b306d15952c962f20a9524c-opensuse-nginx-image",
                "org.opensuse.nginx.reference": "registry.opensuse.org/opensuse/nginx:1.19.6-8.919",
                "org.opensuse.nginx.title": "openSUSE nginx web server container",
                "org.opensuse.nginx.version": "1.19.6-8.919",
                "org.opensuse.reference": "registry.opensuse.org/opensuse/nginx:1.19.6-8.919",
                "org.opensuse.tumbleweed.busybox.created": "2021-01-30T17:00:57.705148070Z",
                "org.opensuse.tumbleweed.busybox.description": "Small image containing busybox.",
                "org.opensuse.tumbleweed.busybox.disturl": "obs://build.opensuse.org/openSUSE:Factory/images/e1deab10410717d6b30d6ff1940e1777-tumbleweed-busybox-image",
                "org.opensuse.tumbleweed.busybox.reference": "registry.opensuse.org/opensuse/busybox:1.32.1-14.1",
                "org.opensuse.tumbleweed.busybox.title": "openSUSE Busybox Container",
                "org.opensuse.tumbleweed.busybox.url": "https://www.opensuse.org/",
                "org.opensuse.tumbleweed.busybox.vendor": "openSUSE Project",
                "org.opensuse.tumbleweed.busybox.version": "1.32.1-14.1"
            },
            "Annotations": {
                "io.container.manager": "libpod",
                "io.kubernetes.cri-o.Created": "2021-02-05T12:52:23.83123341+07:00",
                "io.kubernetes.cri-o.TTY": "false",
                "io.podman.annotations.autoremove": "FALSE",
                "io.podman.annotations.init": "FALSE",
                "io.podman.annotations.privileged": "FALSE",
                "io.podman.annotations.publish-all": "FALSE",
                "org.opencontainers.image.stopSignal": "15"
            },
            "StopSignal": 15,
            "CreateCommand": [
                "podman",
                "run",
                "-d",
                "-p",
                "8000:8000",
                "nginx"
            ],
            "Umask": "0022"
        },
        "HostConfig": {
            "Binds": [
                "c86360b3e3d6b0a0cdc60cf3cb4504a267933d154dae282a7caf760c7b599cdf:/data:rprivate,rw,nodev,exec,nosuid,rbind"
            ],
            "CgroupManager": "cgroupfs",
            "CgroupMode": "host",
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "k8s-file",
                "Config": null
            },
            "NetworkMode": "slirp4netns",
            "PortBindings": {
                "8000/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8000"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": [],
            "CapDrop": [],
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": [],
            "GroupAdd": [],
            "IpcMode": "private",
            "Cgroup": "",
            "Cgroups": "default",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "private",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": [],
            "Tmpfs": {},
            "UTSMode": "private",
            "UsernsMode": "",
            "ShmSize": 65536000,
            "Runtime": "oci",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": 0,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": [],
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "CgroupConf": null
        }
    }
]
```
`Podman inspect` literally covers everything that container is running with. Labels, mounts, volumes, exposed ports etc. Extremely useful for debugging and of course the output can be filtered. For more visit the `podman inspect` man page.

The next useful tool to check what is happening real-time inside a container is `podman logs` which will output the `stdout` of the container. This can cover errors, warnings or general output information from the application inside the container.
```
$ podman logs admiring_franklin
[Thu Feb 04 20:30:48.909589 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module actions_module from /usr/lib64/apache2-prefork/mod_actions.so
[Thu Feb 04 20:30:48.928259 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module alias_module from /usr/lib64/apache2-prefork/mod_alias.so
[Thu Feb 04 20:30:48.937031 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module auth_basic_module from /usr/lib64/apache2-prefork/mod_auth_basic.so
[Thu Feb 04 20:30:48.938069 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module authn_core_module from /usr/lib64/apache2-prefork/mod_authn_core.so
[Thu Feb 04 20:30:48.946344 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module authn_file_module from /usr/lib64/apache2-prefork/mod_authn_file.so
[Thu Feb 04 20:30:48.956344 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module authz_host_module from /usr/lib64/apache2-prefork/mod_authz_host.so
[Thu Feb 04 20:30:48.974215 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module authz_groupfile_module from /usr/lib64/apache2-prefork/mod_authz_groupfile.so
[Thu Feb 04 20:30:48.990336 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module authz_core_module from /usr/lib64/apache2-prefork/mod_authz_core.so
[Thu Feb 04 20:30:48.991396 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module authz_user_module from /usr/lib64/apache2-prefork/mod_authz_user.so
[Thu Feb 04 20:30:48.992640 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module autoindex_module from /usr/lib64/apache2-prefork/mod_autoindex.so
[Thu Feb 04 20:30:48.995014 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module cgi_module from /usr/lib64/apache2-prefork/mod_cgi.so
[Thu Feb 04 20:30:48.996348 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module dir_module from /usr/lib64/apache2-prefork/mod_dir.so
[Thu Feb 04 20:30:49.003103 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module env_module from /usr/lib64/apache2-prefork/mod_env.so
[Thu Feb 04 20:30:49.003674 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module expires_module from /usr/lib64/apache2-prefork/mod_expires.so
[Thu Feb 04 20:30:49.004606 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module include_module from /usr/lib64/apache2-prefork/mod_include.so
[Thu Feb 04 20:30:49.005130 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module log_config_module from /usr/lib64/apache2-prefork/mod_log_config.so
[Thu Feb 04 20:30:49.018137 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module mime_module from /usr/lib64/apache2-prefork/mod_mime.so
[Thu Feb 04 20:30:49.028327 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module negotiation_module from /usr/lib64/apache2-prefork/mod_negotiation.so
[Thu Feb 04 20:30:49.036869 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module setenvif_module from /usr/lib64/apache2-prefork/mod_setenvif.so
[Thu Feb 04 20:30:49.078484 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module ssl_module from /usr/lib64/apache2-prefork/mod_ssl.so
[Thu Feb 04 20:30:49.081221 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module socache_shmcb_module from /usr/lib64/apache2-prefork/mod_socache_shmcb.so
[Thu Feb 04 20:30:49.081547 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module userdir_module from /usr/lib64/apache2-prefork/mod_userdir.so
[Thu Feb 04 20:30:49.082180 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module reqtimeout_module from /usr/lib64/apache2-prefork/mod_reqtimeout.so
[Thu Feb 04 20:30:49.084216 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module rewrite_module from /usr/lib64/apache2-prefork/mod_rewrite.so
[Thu Feb 04 20:30:49.089975 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module headers_module from /usr/lib64/apache2-prefork/mod_headers.so
[Thu Feb 04 20:30:49.135714 2021] [so:debug] [pid 10] mod_so.c(266): AH01575: loaded module php7_module from /usr/lib64/apache2/mod_php7.so
[Thu Feb 04 20:30:49.430178 2021] [core:info] [pid 10] AH00545: MaxRequestsPerChild is deprecated, use MaxConnectionsPerChild instead.
AH00558: httpd-prefork: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
``` 
This can output a lot of information and it is rolling with the container's life cycle. To monitor the logs real-time as they come in - such as one would do with `tail -f` - you can do that with `podman logs -f` followed by the container's name or ID.
 
### Persistent data

Containers are not able to store persistent data. If you need an application to store data between restarts and updates you need to either attach a `volume` or `mount` storage space outside of the container from the host system.

#### Mounts

Are basically can be anything, a file, a folder stored anywhere on the host system. This is not a storage space that is managed by Podman and can be accessed from the host system and from within the container as well. Creating a container with bind mount:

```
$ podman run -it --name tw --mount type=bind,source=./binder,target=/mnt opensuse/tumbleweed /bin/bash                                                              
# 8453197c43b9:/ # df -h
Filesystem           Size  Used Avail Use% Mounted on
fuse-overlayfs       231G   36G  195G  16% /
/dev/mapper/cr_root  231G   36G  195G  16% /mnt
tmpfs                 64M     0   64M   0% /dev
tmpfs                4.0M     0  4.0M   0% /sys/fs/cgroup
tmpfs                782M  374M  409M  48% /etc/hosts
shm                   63M     0   63M   0% /dev/shm
devtmpfs             3.9G     0  3.9G   0% /dev/tty
tmpfs                3.9G     0  3.9G   0% /sys/kernel
tmpfs                3.9G     0  3.9G   0% /proc/acpi
tmpfs                3.9G     0  3.9G   0% /proc/scsi
tmpfs                3.9G     0  3.9G   0% /sys/firmware
tmpfs                3.9G     0  3.9G   0% /sys/dev
# 8453197c43b9:/ # cd /mnt/
# 8453197c43b9:/mnt # ls
# 8453197c43b9:/mnt # touch -a test
# 8453197c43b9:/mnt # ls
test
# 8453197c43b9:/mnt # exit
exit
$ cd binder                                                                                                                                                         
$ ls -a                                                                                                                                                                
total 0
drwxr-xr-x 1 apinter users  8 Feb  5 18:31 .
drwxr-xr-x 1 apinter users 32 Feb  5 17:58 ..
-rw-r--r-- 1 apinter users  0 Feb  5 18:31 test                                                                                                                        
```
Breaking it down:
* `podman run -it --name tw` will start a container in interactive mode and open a terminal (tty) inside the container which is named `tw`,
* `--mount type=bind,source=./binder,target=/mnt` the `--mount` flag will tell Podman that we want to mount something from the host file system, the type is set to `bind` then with `source` set the path to the folder and with `target` set the path within the container where to mount it,
* `opensuse/tumbleweed /bin/bash` defining the image - in this case openSUSE Tumbleweed - and what to run which will be the `bash` shell,
* Following that is a little proof-of-concept that the created file from within the container stay on the host file system after exiting.

#### Volumes

Volumes, unlike bind mounts are managed by Podman and this is the preferred way of having a persistent storage space attached to a container. If running a rootfull session the volume will be stored under `/var/lib/containers`, while rootless volumes will be stored under `~/.local/share/containers/storage`. To be able to attach volumes first need to create them with `podman volume create` and volumes can be listed with `podman volume list`. The volumes can be named and can have labels for meta information. 
```
$ podman volume create testvol                                                                                               
testvol
$ podman volume list                                                                                                        
DRIVER      VOLUME NAME
local       2508530e4332a17683c91aabd18e310607ac57b418da21900e72589c7285ed6f
local       49ad9d0c48db6b8b58a5434de01f2a1b0e2e29fef5ed4b8cf6f85cd11162e985
local       4e79dd89718157538f394e085021800e9f76720f45fed7763793b9cbd2092e4e
local       6c62ee3bc47ad16086cbc772e3dcb358229b2c56b1055167291d5a93bb2f9b4c
local       81d45f6f954fbb85fb4058394b5415c352b05358fc5ed2ae0d05b8ddb885d8d2
local       90d034f570fc7a8b2cb5592cd47a1183c24ff92b2c200e4a8e3a91ab5a70f1ed
local       a297bfa38aecb84775bcfd764dbdc1f885ba0c2d93cdf0ff9bc1e957de6fcef4
local       a6d423add657f0e5ad4ecb5d9c643d2df1ac3708ed433cf857a35b5d2879707a
local       c86360b3e3d6b0a0cdc60cf3cb4504a267933d154dae282a7caf760c7b599cdf
local       f6aec4ec8449c00c5c8f340634571f96befdda2f5c218fb93f2ee4bfb436154e
local       testvol
```
Now mounting the created volume inside a container:
```
$ podman run -it --name tw --mount type=volume,source=testvol,target=/mnt opensuse/tumbleweed /bin/bash                                                             
# 03b1eb2c4b48:/ # df -h
Filesystem           Size  Used Avail Use% Mounted on
fuse-overlayfs       231G   36G  195G  16% /
tmpfs                 64M     0   64M   0% /dev
tmpfs                4.0M     0  4.0M   0% /sys/fs/cgroup
/dev/mapper/cr_root  231G   36G  195G  16% /mnt
tmpfs                782M  374M  408M  48% /etc/hosts
shm                   63M     0   63M   0% /dev/shm
devtmpfs             3.9G     0  3.9G   0% /dev/tty
tmpfs                3.9G     0  3.9G   0% /sys/kernel
tmpfs                3.9G     0  3.9G   0% /proc/acpi
tmpfs                3.9G     0  3.9G   0% /proc/scsi
tmpfs                3.9G     0  3.9G   0% /sys/firmware
tmpfs                3.9G     0  3.9G   0% /sys/dev
# 03b1eb2c4b48:/ # cd /mnt 
# 03b1eb2c4b48:/mnt # ls
# 03b1eb2c4b48:/mnt # echo "a message for the future">> testing
# 03b1eb2c4b48:/mnt # ls
testing
# 03b1eb2c4b48:/mnt # exit
exit
$ podman rm tw                                                                                                                                                      
03b1eb2c4b48d4badc17dfd8f25430afa2e5a32bb10f32e982a4deb86488f6de
$ podman run -it --name tw --mount type=volume,source=testvol,target=/mnt opensuse/tumbleweed /bin/bash                                                             
# 1a203240e9d7:/ # df -h
Filesystem           Size  Used Avail Use% Mounted on
fuse-overlayfs       231G   36G  195G  16% /
tmpfs                 64M     0   64M   0% /dev
tmpfs                4.0M     0  4.0M   0% /sys/fs/cgroup
/dev/mapper/cr_root  231G   36G  195G  16% /mnt
tmpfs                782M  374M  408M  48% /etc/hosts
shm                   63M     0   63M   0% /dev/shm
devtmpfs             3.9G     0  3.9G   0% /dev/tty
tmpfs                3.9G     0  3.9G   0% /sys/kernel
tmpfs                3.9G     0  3.9G   0% /proc/acpi
tmpfs                3.9G     0  3.9G   0% /proc/scsi
tmpfs                3.9G     0  3.9G   0% /sys/firmware
tmpfs                3.9G     0  3.9G   0% /sys/dev
# 1a203240e9d7:/ # cd /mnt/
# 1a203240e9d7:/mnt # ls
testing
# 1a203240e9d7:/mnt # cat testing
a message for the future
# 1a203240e9d7:/mnt # exit
exit
```
Breaking it down (the command is almost identical to the one used in `bind mount` so to keep it short will focus only the differences):
* `--mount type=volume,source=testvol,target=/mnt` just like with `bind mounts` need to use the `--mount` flag, but the `type` will be `volume` and the `source` does not need the whole path just the name/id of the `volume`. Upon the creation of the volume if a name is not set a random one will be generated
* This is being followed by a proof-of-concept in which I created a container, mounted the volume, wrote a file and stored it in the volume then destroyed the container. Recreated it with the same command and `volume` and read out the files previously created inside the already destroyed container.
For more visit the `podman volume` man page.

### Updating containers

Can go about this a few different ways like pulling a container, launching it, mounting the necessary shares and config files and starting it on a different exposed port than the one we're planning to "update". Once this is done, tested and works fine can just change the route to point to this updated container. Me being a lazy DevOps engineer I don't like this option so let's visit `podman auto-update`.

#### Podman auto-update

Before we go any further there are few things to make sure are set correctly on our container:
* there is no `--restart=always` flag present,
* the container is managed by `systemd`,
* the `systemd` unit has been generated by the container's ID not by it's name,
* the `systemd` unit has been generated with the `--new` flag.

Podman's other important feature is the automatic updates of containers without the need of even putting containers in pods or adding a Watchtower service. This is done by adding the `--label io.containers.autoupdate=image` label during container creation.    
This to be followed by generating `systemd` units for the container: `podman generate systemd --new --files [container ID or name]`. Once this is ready and the unit file is under the corresponding `systemd` folder - depending on how podman is used, rootfull or rootless - the container unit to be started. Following this the `podman auto-update` command can update every container with changed images.

## References & credits

I strongly suggest checking out the `man` pages of `podman` which provides a lot of information straight from the terminal. Also the following resources can help a lot on your `podman` journey:

* https://podman.io/
* http://docs.podman.io/en/latest/
* http://docs.podman.io/en/latest/Tutorials.html