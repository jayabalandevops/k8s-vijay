# Sri Ramajeyam..! [sri ram](http://google.com)
# Kubernetes Installation

  [Kubectl Installation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl) and [Docker](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker). 
These are the official sites, to be followed for installations.
  
  Below are the steps required to get this working on a base linux system.
  
  - Provision a Virtual machine
  - Install kubernetes 
  - Install docker
  - Install and Configure ```kubeadm``` in the kubernetes ```master```
  - Join other ```nodes``` with the ```master```
  - Verify the cluster
   
## 1. Provision the Virtual machines
  
  Create 4 machines of same configuration.
  - Ram 4GB, #of CPUs 2, Ubuntu 18.04 LTS, 10GB Hard Disk.
  - master, node1, node2, node3
  - Note: all the Virtual machines should be in the same network
  - Its applicable to any cloud providers
  - [GCE](https://console.cloud.google.com) and [AWS](https://console.aws.amazon.com). 
   
## 2. Install kubernetes
    
 Install kubectl,kubeadm and kubelet
  - Follow this link for 
  [Kubectl Installation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl).
  - Note: tested using ubuntu 18.04 LTS
  ```shell script

```
## 3. Start Database Service
  - Start the database service
    
        service mysql start

  - Create database and database users
        
        # mysql -u <username> -p
        
        mysql> CREATE DATABASE employee_db;
        mysql> GRANT ALL ON *.* to db_user@'%' IDENTIFIED BY 'Passw0rd';
        mysql> USE employee_db;
        mysql> CREATE TABLE employees (name VARCHAR(20));
        
  - Insert some test data
        
        mysql> INSERT INTO employees VALUES ('JOHN');
    
## 4. Install and Configure Web Server

Install Python Flask dependency

    pip install flask
    pip install flask-mysql

- Copy app.py or download it from source repository
- Configure database credentials and parameters 

## 5. Start Web Server

Start web server

    FLASK_APP=app.py flask run --host=0.0.0.0
    
## 6. Test

Open a browser and go to URL

    http://<IP>:5000                            => Welcome
    http://<IP>:5000/how%20are%20you            => I am good, how about you?
    http://<IP>:5000/read%20from%20database     => JOHN
