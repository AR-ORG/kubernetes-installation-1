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

      1.    Install cfssl -
      
      curl -s -L -o /bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
      
      curl -s -L -o /bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
      
      curl -s -L -o /bin/cfssl-certinfo https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
      
      chmod +x /bin/cfssl*
      
          
      2.    Install kubectl -
      
      sudo apt-get update && sudo apt-get install -y apt-transport-https

      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

      echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

      sudo apt-get update
      
      sudo apt-get install -y kubectl
      
      
      
      3.    Provision Certificate Authority
      
      Create 2 files - 
      
      1.    ca-config.json
      
                  {
              "signing": {
                "default": {
                  "expiry": "8760h"
                },
                "profiles": {
                  "kubernetes": {
                    "usages": ["signing", "key encipherment", "server auth", "client auth"],
                    "expiry": "8760h"
                  }
                }
              }
            }
            
      2.    ca-csr.json
      
                  {
              "CN": "Kubernetes",
              "key": {
                "algo": "rsa",
                "size": 2048
              },
              "names": [
                {
                  "C": "US",
                  "L": "Portland",
                  "O": "Kubernetes",
                  "OU": "CA",
                  "ST": "Oregon"
                }
              ]
            }
            
      Execute - cfssl gencert -initca ca-csr.json | cfssljson -bare ca 
      
      The above commands createsthe below files 
      
      ca.pem -- Public Certificate
      
      ca-key.pem  -- private certificate
      
      ca.csr  -- CSR file
      
      
      4.    Generate Client Certificates
      
      
      
      

      
      
