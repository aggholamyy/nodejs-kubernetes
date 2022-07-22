# How to Deploy a node.js application to kuberenetes(k8s).

## Question 1
`Provide steps to set up Nginx reverse proxy, as well as reasons for it.`

```bash
server {
listen      80;
server_name main-domain.com www.main-domain.com;
index       index.php;
root        /var/www/main-domain.com/public    # fallback for index.php
location / {
try_files $uri $uri/ /index.php?$query_string;
}location /blog {
proxy_pass http://blog.domain.com;proxy_http_version                 1.1;
proxy_cache_bypass                 $http_upgrade;

# Proxy headers
proxy_set_header Upgrade           $http_upgrade;
proxy_set_header Connection        "upgrade";
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header Host              $host;
proxy_set_header X-Forwarded-Host  $host;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Port  $server_port;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;


# Proxy timeouts
proxy_send_timeout                 60s;
proxy_connect_timeout              60s;
proxy_read_timeout                 60s;
}
```
## Question 2
`Assuming we have an EC2 t3a.small instance running that is managed with Terraform, the local configuration file is edited and the instance type is changed to t3a.large , if the terraform apply command is run, what would happen?`

In this scenario it will stop the instance, change the instance type and then start it so this will cause some downtime.

## Question 3
`Imagine you have a DB backup dump/file which is compressed to tar.gz format and its size is 2TB. We always need to share this file between different EC2 instances, can you provide steps?`

In terms of using Amazon services for fulfill the question, I choose  `Amazon Elastic File System (Amazon EFS)` or `EBS Multi-Attach enabled EBS volumes` which allows to attache EC2s up to 16 instances.

But it could be possible to share these files over GlusterFs shared storage, as Glusterfs has good performance on sharing large files which do not face with micro-changes over time or immutable.
In this senario we install gluster server on 2 instances (if there is not any other instance, if there are other instances it is far better(for protecting data from loss) to install gluster server on those and just mount them  on these two EC2 which need to share file by glusterfs client).

## Question 4
`You have ssh access to an Ubuntu or Debian Linux server, the disk is full and you need to clean it up, what would you do and which specific commands should you run?`

```bash
sudo journalctl --vacuum-size 5M
sudo apt-get autoclean
sudo apt-get autoremove --purge
sudo du -sk /var/log/* | sort -nr | head -10
rm -f <biggest log files if is it possible>
```
## Question 5
`The following NodeJs app has been developed, and the team is requesting a complete deployment approach. You have a pair of ACCESS_KEY and SECRET_KEY of an AWS user and also you can tell the customer what they need to do from the AWS console if you need.`

All the files in this gist are for the Question5, and I made the deployment based on kubernetes cluster on my labtop which created based on k3d project by rancher.

1. 
   ```bash
   docker build . -t shardlabs_node_app
   ```
1. 
   ```bash
   k3d image import -c <k8s cluster name> shardlabs_node_app
   ```
1. 
   ```bash
   k apply -f deployment.yaml
   ```
1. ```bash
   k apply -f ingress.yaml
   ```
1. Based on the port you choose for loadbalancer of k8s cluster of k3d now you can access the page by browser, in my case the lb maped on port 8080, therefor I can access the page by this address `http://localhost:8080/`
