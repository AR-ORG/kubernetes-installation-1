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
      
##    Create Service account key-pair certificate

      a.    Kubernetes uses Service account certificates to sign tokens for each service account created 
      
      b.    Create a file - service-account-csr.json 
      
      c.    
                        {
              "CN": "service-accounts",
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
      
      d.    Execute -- cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes service-account-csr.json | cfssljson -bare service-account
      
##    Copy certificates to respective nodes - 

      a.   Worker Nodes - ca.pem, worker*.pem
       
      b.   Master Nodes - ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem
       
      c.   The remaining certificates will be used in the next section while creating kubeconfig. 
       
##    Generate kubeconfig 

      a.    Kubeconfig stores information about the cluster, users, namespaces and the correspinding authentication mechanism. Multiple K8S clusters can interact with each other using the configuration stored inside kubeconfig.
      
      b.    kubeconfig will be generated for the below
            
            Admin user
            
            kube-scheduler
            
            kube-proxy
            
            kube-controller
            
            kubelet - for each worker

##    Generate kubeconfig for kubelet (each worker)

      a.    export KUBERNETES_ADDRESS=IP_ADDRESS_OF_LOADBALANCER
      
      b.    for instance in worker1 worker2   # {{WORKER NODES}}
            do
              kubectl config set-cluster mycluster \  #{{cluster name}}
                --certificate-authority=ca.pem \   # {{public key for CA}}
                --embed-certs=true \
                --server=https://${KUBERNETES_ADDRESS}:6443 \    # {{Variable from above}}
                --kubeconfig=${instance}.kubeconfig

              kubectl config set-credentials system:node:${instance} \  #{{set the username}}
                --client-certificate=${instance}.pem \    # {{kubelet certificate public}}
                --client-key=${instance}-key.pem \    #{{kubelet private certificate}}
                --embed-certs=true \
                --kubeconfig=${instance}.kubeconfig   #{{name of kubeconfig}}

              kubectl config set-context default \
                --cluster=mycluster \
                --user=system:node:${instance} \
                --kubeconfig=${instance}.kubeconfig

              kubectl config use-context default --kubeconfig=${instance}.kubeconfig
            done
            
        c.  Make sure to remove the # comments before running the above script. 
        
##    Generate kubeconfig for kube-proxy 

      a.    Single kube-proxy config will be used for all nodes. 
      
      b.    kubectl config set-cluster mycluster \
                --certificate-authority=ca.pem \
                --embed-certs=true \
                --server=https://${KUBERNETES_ADDRESS}:6443 \
                --kubeconfig=kube-proxy.kubeconfig

              kubectl config set-credentials system:kube-proxy \
                --client-certificate=kube-proxy.pem \
                --client-key=kube-proxy-key.pem \
                --embed-certs=true \
                --kubeconfig=kube-proxy.kubeconfig

              kubectl config set-context default \
                --cluster=mycluster \
                --user=system:kube-proxy \
                --kubeconfig=kube-proxy.kubeconfig

              kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
              
##    Generate kubeconfig for kube-controller-manager

      a.    kube-controller-manager will not interact with LoadBalancer directly. Hence the server address can be kept as loopback IP.
      
      b.     kubectl config set-cluster mycluster \
                --certificate-authority=ca.pem \
                --embed-certs=true \
                --server=https://127.0.0.1:6443 \
                --kubeconfig=kube-controller-manager.kubeconfig

              kubectl config set-credentials system:kube-controller-manager \
                --client-certificate=kube-controller-manager.pem \
                --client-key=kube-controller-manager-key.pem \
                --embed-certs=true \
                --kubeconfig=kube-controller-manager.kubeconfig

              kubectl config set-context default \
                --cluster=mycluster \
                --user=system:kube-controller-manager \
                --kubeconfig=kube-controller-manager.kubeconfig

              kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig

##    Generate kubeconfig for kube-scheduler
      
      a.    kube-scheduler will not interact with LoadBalancer directly. Hence the server address can be kept as loopback IP.
      
      b.    kubectl config set-cluster mycluster \
                --certificate-authority=ca.pem \
                --embed-certs=true \
                --server=https://127.0.0.1:6443 \
                --kubeconfig=kube-scheduler.kubeconfig

              kubectl config set-credentials system:kube-scheduler \
                --client-certificate=kube-scheduler.pem \
                --client-key=kube-scheduler-key.pem \
                --embed-certs=true \
                --kubeconfig=kube-scheduler.kubeconfig

              kubectl config set-context default \
                --cluster=mycluster \
                --user=system:kube-scheduler \
                --kubeconfig=kube-scheduler.kubeconfig

              kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
              
##    Generate kubeconfig for Admin

      a.      kubectl config set-cluster mycluster \
                --certificate-authority=ca.pem \
                --embed-certs=true \
                --server=https://127.0.0.1:6443 \
                --kubeconfig=admin.kubeconfig

              kubectl config set-credentials admin \
                --client-certificate=admin.pem \
                --client-key=admin-key.pem \
                --embed-certs=true \
                --kubeconfig=admin.kubeconfig

              kubectl config set-context default \
                --cluster=mycluster \
                --user=admin \
                --kubeconfig=admin.kubeconfig

              kubectl config use-context default --kubeconfig=admin.kubeconfig
              
##    Distribute kubeconfig files to master and workers
      
      a.    worker nodes - worker{{X}}.kubeconfig kube-proxy.kubeconfig
      
      b.    master nodes - admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig
      
##    Generate data encryption configuration

      a.    kube-apiserver stores data in etcd. There are cases when you would want to encrypt your data at rest.
      
      b.    In case of secrets, data is encrypted so that its not stored as clear text in etcd. 
      
      c.    In order to achieve this we need to provide kubernetes with an encryption key. 
      
      d.    More details at : https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
      
      e.    ENCRYPTION_KEY=$(head -c 32 /dev/urandom| base64)
      
      f.    Create a file - encryption-config.yaml
      
      kind: EncryptionConfig
      apiVersion: v1
      resources:
        - resources:
            - secrets
          providers:
            - aescbc:
                keys:
                  - name: key1
                    secret: ${ENCRYPTION_KEY}
            - identity: {}
            
        g.  scp encryption-config.yaml to master nodes. 
        

      
      
      
      
      

