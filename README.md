# kubernetes-installation
Install Kubernetes

Requirement -6 VM - ubuntu 

1.  controller1 - 2CPUx4GB
2.  controller2 - 2CPUx4GB
3.  worker1 - 2CPUx4GB
4.  worker2 - 2CPUx4GB
5.  loadbalancer - 1CPUx2GB
6.  clientnode  - 1CPUx2GB

##  Client Node (clientnode) steps

##    Install cfssl -
      
      curl -s -L -o /bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
      
      curl -s -L -o /bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
      
      curl -s -L -o /bin/cfssl-certinfo https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
      
      chmod +x /bin/cfssl*
      
          
##    Install kubectl -
      
      sudo apt-get update && sudo apt-get install -y apt-transport-https

      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

      echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

      sudo apt-get update
      
      sudo apt-get install -y kubectl 
      
##    Provision Certificate Authority
      
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
      
      
##    Generate Admin Client certificate
          
      Create a file : admin-csr.json
            
                              {
                    "CN": "admin",
                    "key": {
                      "algo": "rsa",
                      "size": 2048
                    },
                    "names": [
                      {
                        "C": "US",
                        "L": "Portland",
                        "O": "system:masters",
                        "OU": "Kubernetes The Hard Way",
                        "ST": "Oregon"
                      }
                    ]
                  }
                  
      Execute -- cfssl gencert  -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin  
            
 ##   Generate client certificate for kubelet 
            
      1.  Create a file nodedetails.txt with details of hostname and private IP address separated by colon
      2.  Execute the script create_worker_kubelet_certificate.sh against nodedetails.txt to generate certificates for worker nodes. 
      
##    Generate client certificate for controller Manager 
            
      Create a file - kube-controller-manager.json 
                                          {
                          "CN": "system:kube-controller-manager",
                          "key": {
                            "algo": "rsa",
                            "size": 2048
                          },
                          "names": [
                            {
                              "C": "US",
                              "L": "Portland",
                              "O": "system:kube-controller-manager",
                              "OU": "Kubernetes The Hard Way",
                              "ST": "Oregon"
                            }
                          ]
                        }
                  
      Execute - cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager.json | cfssljson -bare kube-controller-manager
                  
##    Generate client certificate for kube-proxy
            
      Create a file - kube-proxy-csr.json
                  
                                          {
                          "CN": "system:kube-proxy",
                          "key": {
                            "algo": "rsa",
                            "size": 2048
                          },
                          "names": [
                            {
                              "C": "US",
                              "L": "Portland",
                              "O": "system:node-proxier",
                              "OU": "Kubernetes The Hard Way",
                              "ST": "Oregon"
                            }
                          ]
                        }
                  
      cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
                  
##    Create client certificate for kube-scheduler
            
      Create a file - kube-scheduler-csr.json
                  
                         {
                          "CN": "system:kube-scheduler",
                          "key": {
                            "algo": "rsa",
                            "size": 2048
                          },
                          "names": [
                            {
                              "C": "US",
                              "L": "Portland",
                              "O": "system:kube-scheduler",
                              "OU": "Kubernetes The Hard Way",
                              "ST": "Oregon"
                            }
                          ]
                        }
                        
      cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler

##    Generate kubernetes apiserver Server Certificate
      
      a.    export CERT_HOSTNAME=10.32.0.1,IP_ADDRESSES_OF_MASTER,HOSTNAMES_OF_MASTERS,IP_ADDRESS_OF_LB,HOSTNAME_OF_LB,127.0.0.1,localhost,kubernetes.default
      b.    Create a file - kubernetes-csr.json
            
                                          {
                          "CN": "kubernetes",
                          "key": {
                            "algo": "rsa",
                            "size": 2048
                          },
                          "names": [
                            {
                              "C": "US",
                              "L": "Portland",
                              "O": "Kubernetes",
                              "OU": "Kubernetes The Hard Way",
                              "ST": "Oregon"
                            }
                          ]
                        }
              
      c.    Execute - cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -hostname=${CERT_HOSTNAME} -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
        

      
      
