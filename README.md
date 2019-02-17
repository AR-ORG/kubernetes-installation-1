# kubernetes-installation
Install Kubernetes

Requirement -6 VM - ubuntu 

1.  controller1 - 2CPUx4GB
2.  controller2 - 2CPUx4GB
3.  worker1 - 2CPUx4GB
4.  worker2 - 2CPUx4GB
5.  loadbalancer - 1CPUx2GB
6.  clientnode  - 1CPUx2GB

##  Install client tools on clientnode

      Install cfssl -
      
      curl -s -L -o /bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
      
      curl -s -L -o /bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
      
      curl -s -L -o /bin/cfssl-certinfo https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
      
      chmod +x /bin/cfssl*
      
          
      Install kubectl -
      
      sudo apt-get update && sudo apt-get install -y apt-transport-https

      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

      echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

      sudo apt-get update
      
      sudo apt-get install -y kubectl
