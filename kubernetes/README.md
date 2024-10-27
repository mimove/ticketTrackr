# Steps to run initial nginx deployment

As a first step towards learning how to use kubernetes. We have created an [example-nginx](./example-nginx/nginx-deployment.yaml)  to test the configuration of microk8s in ubuntu server (it can be in a virtual machine or in a Rapsberry Pi with at least 2 GB of RAM)

## Requirements

First, you need to install microk8s running

```sh
sudo snap install microk8s --classic
```

*If you want to remove a previous installation, you can run

```sh
sudo snap remove microk8s --purge
```


## Commands to deploy the service

1. Start microk8s
   
```sh
sudo microk8s start
```

2. Add your user to the microk8s group to avoid needing sudo for MicroK8s commands:

```sh
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
newgrp microk8s
```

3. Enable necessary services
```sh
microk8s enable dns
microk8s enable rbac
microk8s enable storage
```
* You can also verify that the API server is accesible by running
  ```sh
  microk8s kubectl get nodes
  ```


4. Create a namespace
```sh 
microk8s kubectl create namespace <your-namespace>
```

5. Apply using kubectl
```sh
microk8s kubectl apply -f nginx-deployment.yaml -n <your-namespace>
```

6. Verify that the deployment is running
```sh
microk8s kubectl get deployments -n <your-namespace>
microk8s kubectl get pods -n <your-namespace>
```


## Commands to access the service from host machine

After you've deployed the service you can follow the next steps to access it from your local machine

1. Expose the service so you can access it from the host machine

```sh
microk8s kubectl expose deployment nginx-deployment --type=NodePort --port=80 -n <your-namespace>
```

2. Find the assign port

```sh
microk8s kubectl get service nginx-deployment -n <your-namespace>
```

This will give you an output similar to:
```ruby
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-deployment  NodePort   10.152.183.100  <none>        80:XXXXX/TCP   1m
```

Note the XXXXX in the PORT(S) column; this is the NodePort.


3. Ensure the port is open and accessible
```sh
sudo ufw allow XXXXX/tcp
sudo ufw reload
```

4. Access the service from the host machine
```sh
http://<vm-ip>:XXXXX
```


<br>
<br>

# Steps to access the cluster using [k9s](https://k9scli.io/) from your host machine

Now that you have the initial deployment running, you can configure a service (such as [k9s](https://k9scli.io/)) to access your cluster from the host machine.

This guide will show you the steps you need to achieve just that.

1. Install k9s in your host machine (MacBook steps)

```sh
brew install derailed/k9s/k9s
```

2. Edit your MicroK8s configuration to allow for external access adding this line ```--bind-address=0.0.0.0``` to the following file:

```sh
sudo vi /var/snap/microk8s/current/args/kube-apiserver
```

3. Stop and start the microK8s service

```sh
microk8s stop
microk8s start
```

4. Check the status:

```sh
microk8s status
```


5. Retrieve the config file

```sh
microk8s config > ~/.kube/config
```

6. Open firewall port in your vm
```sh
sudo ufw allow 16443/tcp
sudo ufw reload
```

7. Copy it to your host machine
   
```sh
scp <username>@<vm-ip>:~/.kube/config ~/.kube/
```

8. Run k9s




<br>
<br>

# Run CLI in the cluster

To run the CLI for ticketTrackr, first you need to create the secrets of some variables inside the cluster. 

To do that, you have to run the following command (after you've defined your variables in the .env file)

```sh
microk8s kubectl create secret generic cli-env --from-env-file=.env -n <your-namespace>
```

* If you need to remove a secret to update it you can run
  ```sh
  kubectl delete secret cli-env -n <your-namespace>
  ```

After that, you can go to the folder with the [manifest](./manifest/extract_load.yml) and load it using this command:

```sh
microk8s kubectl apply -f extract_load.yml
```