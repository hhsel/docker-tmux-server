# tmux server Deployment manifest for Kubernetes
This is initially aimed at running a tmux server for arm64-based Kubernetes clusters, so if you plan to use this image you'll need some modification (though that's quite easy).  
I'll extend this repository to allow for users to deploy in more architecture-agnostic way if needed.
Currently single user login (tmux user) is supported.

Confirmed to work on ROCK64 and ROCKPRO64 based kubernetes clusters, but it will work for arm64 platforms without no modification.  
If you're working on x86-based clusters, see below instruction.

## How to use (for arm64-based platforms)
 * Move to kubernetes directory.
 * Add public key to files/authorized_keys file.
 * run ```./create_configmap``` to deploy ConfigMap.
 * run kubectl apply -f, to deploy deploy.yml and svc.yml.
 * Now you can ssh into the pod with jumpserver-internal.management.svc.k8s.local (k8s.local part depends on your setup) with the private key you created with the public key that was added to files/authorized_keys. run
``ssh -i <my private key> tmux@jumpserver-internal.management.svc.k8s.local``
 * Currently logging in as tmux user is supported, and password authentication is prohibited (the password is on Dockerfile!).
 * After login, run 'tmux new' to start tmux. You can further ssh into somewhere else.

### if you need modification...
 * Move to docker directory and modify Dockerfile, then run ./build, then 'hhsel/tmux-server-arm64:2.8' will be built.
 * Delete intermediate (dangling) images to save disk space.
 * Upload the image to your local registry

## for x86-based platforms
 * Remove arm64v8 parts from Dockerfile and build yourself
 * Remove ```beta.kubernetes.io/arch: arm64``` nodeSelector from deploy.yml.
 * Remove arm64 from image name if you prefer.
 
## Container Options
Mount below files to override default config.

 * sshd config file: /opt/openssh/etc/sshd_config
 * authorized_keys: /home/tmux/.ssh/authorized_keys
 * tmux config file: /home/tmux/.tmux.conf
 
## Container specification
 * Supervisord launches sshd and rsyslogd (for obtaining sshd-related logs).
 * **The container prints sshd logs to stdout**.
 * tmux version is 2.8, latest as of Jan 2019.
