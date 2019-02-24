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
            
      1.  Execute gencertkubelet.sh from the repository. CHange the worker1 and worker2 details  
      
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
        
        d.  You can execute ./generate_kube_config from this repo. Change the worker names
        
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
        
        
##    Configure ETCD cluster

      a.    ETCD is a distributed key value store that provides a reliable way to store data across a cluster of machines
      
      b.    ETCD keeps the data in sync across all machines 
      
      c.    Kubernetes uses ETCD to store all of its data about the cluster state inside etcd distributed datastore
      
      d.    ETCD will be installed on master nodes. 
      
      e.    On all the master nodes perform the below steps 
      
      f.    wget -q --show-progress --https-only --timestamping "https://github.com/coreos/etcd/releases/download/v3.3.5/etcd-v3.3.5-linux-amd64.tar.gz"    ---   Download ETCD binary
      
      g.    tar -xvf etcd-v3.3.5-linux-amd64.tar.gz
      
      h.    sudo mv etcd-v3.3.5-linux-amd64/etcd* /usr/local/bin/
      
      i.     mkdir -p /etc/etcd /var/lib/etcd
      
      j.    cd certs; cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
      
      k.    export few environment variables on all master nodes
            
            export ETCD_NAME=NAME_OF_THE_NODE
            export INTERNAL_IP=IP_ADDRESS_OF_NODE
            export INITIAL_CLUSTER=MASTER1_NAME=https://MASTER1_PRIVATE_IP:2380,MASTER2_NAME=https://MASTER2_PRIVATE_IP,.........
            example : export INITIAL_CLUSTER=controller1=https://10.142.0.4:2380,controller2=https://10.142.0.3:2380
            
      l.    Create a systemd unit file for etcd on all masters 
      
            cat << EOF | sudo tee /etc/systemd/system/etcd.service
            [Unit]
            Description=etcd
            Documentation=https://github.com/coreos

            [Service]
            ExecStart=/usr/local/bin/etcd \\
              --name ${ETCD_NAME} \\
              --cert-file=/etc/etcd/kubernetes.pem \\
              --key-file=/etc/etcd/kubernetes-key.pem \\
              --peer-cert-file=/etc/etcd/kubernetes.pem \\
              --peer-key-file=/etc/etcd/kubernetes-key.pem \\
              --trusted-ca-file=/etc/etcd/ca.pem \\
              --peer-trusted-ca-file=/etc/etcd/ca.pem \\
              --peer-client-cert-auth \\
              --client-cert-auth \\
              --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
              --listen-peer-urls https://${INTERNAL_IP}:2380 \\
              --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
              --advertise-client-urls https://${INTERNAL_IP}:2379 \\
              --initial-cluster-token etcd-cluster-0 \\
              --initial-cluster ${INITIAL_CLUSTER} \\
              --initial-cluster-state new \\
              --data-dir=/var/lib/etcd
            Restart=on-failure
            RestartSec=5

            [Install]
            WantedBy=multi-user.target
            EOF
            
      m.    sudo systemctl daemon-reload
      
      n.    sudo systemctl enable etcd
      
      o.    sudo systemctl start etcd
      
      p.    Verify etcd -
            
            sudo systemctl status etcd
            
            sudo ETCDCTL_API=3 etcdctl member list \
              --endpoints=https://127.0.0.1:2379 \
              --cacert=/etc/etcd/ca.pem \
              --cert=/etc/etcd/kubernetes.pem \
              --key=/etc/etcd/kubernetes-key.pem

##    Download binaries for the Kubernetes Control Plane 

      a.    Kubernetes control plane consists of -
      
            kube-apiserver
            
            etcd
            
            kube-scheduler
            
            kube-controller-manager
            
            cloud-controller-manager (OPTIONAL)
            
      b.    The below steps are to be performed on all MASTER nodes
            
      c.    sudo mkdir -p /etc/kubernetes/config
      
      d.    Download the kubernetes version 1.13 latest binaries from official kubernetes release page - https://kubernetes.io/docs/setup/release/notes/#client-binaries
      
            wget https://dl.k8s.io/v1.13.0/kubernetes-server-linux-amd64.tar.gz
            
      e.    unzip and untar the the file
      
      f.    cd kubernetes/server/bin
      
      g.    cp kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
      
##    Configure the kube-apiserver

      a.    The systemd unit file for kube-apiserver needs to be installed on all MASTERS
      
      b.    mkdir -p /var/lib/kubernetes/
      
      c.    Copy certificates to /var/lib/kubernetes
            
            cp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem  service-account-key.pem service-account.pem encryption-config.yaml /var/lib/kubernetes/
            
      d.    export the below variables on all MASTER hosts
            
            export INTERNAL_IP=INTERNAL_IP_ADDRESS_OF_THE_MASTER
            
            export CONTROLLER0_IP=IP_ADDRESS_OF_MASTER0
            
            export CONTROLLER1_IP=IP_ADDRESS_OF_MASTER1
            
            and so on export CONTROLLER2_IP, CONTROLLER3_IP.....
            
      e.    Create the systemd unit file for kube-apiserver that takes the variables from above - 
      
      cat << EOF | sudo tee /etc/systemd/system/kube-apiserver.service
      [Unit]
      Description=Kubernetes API Server
      Documentation=https://github.com/kubernetes/kubernetes

      [Service]
      ExecStart=/usr/local/bin/kube-apiserver \\
        --advertise-address=${INTERNAL_IP} \\
        --allow-privileged=true \\
        --apiserver-count=3 \\
        --audit-log-maxage=30 \\
        --audit-log-maxbackup=3 \\
        --audit-log-maxsize=100 \\
        --audit-log-path=/var/log/audit.log \\
        --authorization-mode=Node,RBAC \\
        --bind-address=0.0.0.0 \\
        --client-ca-file=/var/lib/kubernetes/ca.pem \\
        --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
        --enable-swagger-ui=true \\
        --etcd-cafile=/var/lib/kubernetes/ca.pem \\
        --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
        --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
        --etcd-servers=https://$CONTROLLER0_IP:2379,https://$CONTROLLER1_IP:2379 \\
        --event-ttl=1h \\
        --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
        --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
        --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
        --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
        --kubelet-https=true \\
        --runtime-config=api/all \\
        --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
        --service-cluster-ip-range=10.32.0.0/24 \\
        --service-node-port-range=30000-32767 \\
        --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
        --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
        --v=2 \\
        --kubelet-preferred-address-types=InternalIP,InternalDNS,Hostname,ExternalIP,ExternalDNS
      Restart=on-failure
      RestartSec=5

      [Install]
      WantedBy=multi-user.target
      EOF
      
##    Set Up kube-controller-manager

      a.    The below steps are to be performed on all master servers
      
      b.     cp kube-controller-manager.kubeconfig /var/lib/kubernetes/
      
      c.    Generate the systemd unit file for kube-scheduler
      
      cat << EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
      [Unit]
      Description=Kubernetes Controller Manager
      Documentation=https://github.com/kubernetes/kubernetes

      [Service]
      ExecStart=/usr/local/bin/kube-controller-manager \\
        --address=0.0.0.0 \\
        --cluster-cidr=10.200.0.0/16 \\
        --cluster-name=kubernetes \\
        --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
        --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
        --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
        --allocate-node-cidrs=true \
        --leader-elect=true \\
        --root-ca-file=/var/lib/kubernetes/ca.pem \\
        --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
        --service-cluster-ip-range=10.32.0.0/24 \\
        --use-service-account-credentials=true \\
        --v=2
      Restart=on-failure
      RestartSec=5

      [Install]
      WantedBy=multi-user.target
      EOF
      
      d.    Explanation of few flags -
            
            --kubeconfig - location of the kubeconfig file
            
            --cluster-cidr - This is the cidr for the cluster
            
            --service-cluster-ip-range - This ip address is used by kubernetes internally for initializing ClusterIp service for kubernetes cluster
            
 ##   Configure kube-scheduler
 
      a.   The below steps are to be performed on all master servers
      
      b.    cp kube-scheduler.kubeconfig /var/lib/kubernetes/
      
      c.    kube-scheduler requires an additional YAML files to store kube-scheduler configuration
      
      e.    Create the kube-scheduler.yaml file to store the kube-scheduler configs
      
      cat << EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
      apiVersion: kubescheduler.config.k8s.io/v1alpha1
      kind: KubeSchedulerConfiguration
      clientConnection:
        kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
      leaderElection:
        leaderElect: true
      EOF
      
      f.    Create the systemd unit file for kube-scheduler
      
      cat << EOF | sudo tee /etc/systemd/system/kube-scheduler.service
      [Unit]
      Description=Kubernetes Scheduler
      Documentation=https://github.com/kubernetes/kubernetes

      [Service]
      ExecStart=/usr/local/bin/kube-scheduler \\
        --config=/etc/kubernetes/config/kube-scheduler.yaml \\
        --v=2
      Restart=on-failure
      RestartSec=5

      [Install]
      WantedBy=multi-user.target
      EOF
      
##    Start the Master Control Plane

      a.    sudo systemctl daemon-reload
      
      b.    sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
      
      c.    sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
      
      d.    kubectl get componentstatuses --kubeconfig admin.kubeconfig
      
      e.    sudo systemctl status kube-apiserver kube-controller-manager kube-scheduler
      
##    Set Up health-check on /healthz

      a.    In our current scenario - we will use nginx to set up a health check for kubernetes cluster at https://localhost:6443/healthz. This needs to be setup on all Master nodes
      
      b.    You can use external LB on AWS/GCP to perform the same action and setup a health check for the same
      
      c.    apt-get install -y nginx 
      
      d.    Create a nginx.config.cluster file with the below proxypass content - 
      
            server {
              listen      80;
              server_name kubernetes.default.svc.cluster.local;

              location /healthz {
                 proxy_pass                    https://127.0.0.1:6443/healthz;
                 proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
              }
            }
            
      e.    cp nginx.config.cluster /etc/nginx/sites-available/nginx.config.cluster
      
      f.    ln -s /etc/nginx/sites-available/nginx.config.cluster /etc/nginx/sites-enabled/
      
      g.    sudo systemctl restart nginx

      h.    sudo systemctl enable nginx
      
      i.    Test the health-check using - curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
      
##    Set Access controls for Kubelet authorization

      a.    Kubernetes API communicates with Kubelet API for various tasks provided to it either by user or by other master components. 
            API-server may also access kubelet to gather metrics or log information
      
      b.    Kubetnetes API needs to have corresponding authorization to access kubelet API on all nodes.
      
      c.    In kubernetes access control is governed using RBAC (Role based access control)
      
      d.    Since API server is up and running, the RBAC can be created on any one master. 
      
      e.    Create a file kubelet-rbac-clusterrole.yaml
      
      apiVersion: rbac.authorization.k8s.io/v1beta1
      kind: ClusterRole
      metadata:
        annotations:
          rbac.authorization.kubernetes.io/autoupdate: "true"
        labels:
          kubernetes.io/bootstrapping: rbac-defaults
        name: system:kube-apiserver-to-kubelet
      rules:
        - apiGroups:
            - ""
          resources:
            - nodes/proxy
            - nodes/stats
            - nodes/log
            - nodes/spec
            - nodes/metrics
          verbs:
            - "*"
      
      f.    create another file - kubelet-rbac-clusterrolebinding.yaml
      
      apiVersion: rbac.authorization.k8s.io/v1beta1
      kind: ClusterRoleBinding
      metadata:
        name: system:kube-apiserver
        namespace: ""
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: system:kube-apiserver-to-kubelet
      subjects:
        - apiGroup: rbac.authorization.k8s.io
          kind: User
          name: kubernetes
          
      g.    kubectl apply -f kubelet-rbac-clusterrole.yaml --kubeconfig admin.kubeconfig
      
      h.    kubectl apply -f kubelet-rbac-clusterrolebinding.yaml --kubeconfig admin.kubeconfig
      
##    Setup Loadbalancer for api-server

      a.    The loadbalancer will balance load between the apiserver on all the masters.
            You can utilize the tcp stream functionality of nginx to create your loadbalancer. Alternatively you may also use
            HAPROXY or any external LB of your choice.
      
      b.    The below commands are to be run on loadbalancer node
      
      c.    apt-get install -y nginx
      
      d.    systemctl enable nginx
      
      e.    mkdir -p /etc/nginx/tcpconf.d
      
      f.    Edit /etc/nginx/nginx.conf and add the following line at the end - include /etc/nginx/tcpconf.d/*; 
      
      g.    Create a file - /etc/nginx/tcpconf.d/kubernetes.conf
      
      stream {
          upstream kubernetes {
              server $CONTROLLER0_IP:6443;
              server $CONTROLLER1_IP:6443;
          }

          server {
              listen 6443;
              listen 443;
              proxy_pass kubernetes;
          }
      }
      
      h.    nginx -s reload 
      
      i.    Test the configuration - curl -k https://localhost:6443/version  --- Output should be a json response with version 1.13
      
##    Setting up Kubernetes worker nodes 

      a.    Worker nodes consists of 
      
            kubelet
            
            kube-proxy
            
            Container runtime - docker / containerd / rkt etc.
            
      b.    The below steps are to be run on all worker nodes 
      
      c.    apt-get -y install socat conntrack ipset
      
            conntrack - dependency required for kube-proxy
            
            socat - Used to interface with containers while executing kubectl exec or kubectl proxy etc. 
            
            ipset - used by IPVS for loadbalancing with client session based affinity
            
##    Install crictl on worker nodes

      VERSION="v1.13.0"
      
      wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
      
      sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
      
      rm -f crictl-$VERSION-linux-amd64.tar.gz

##    Install runsc on worker nodes
      gVisor is a user-space kernel, written in Go, that implements a substantial portion of the Linux system surface. It includes an Open Container Initiative (OCI) runtime called runsc that provides an isolation boundary between the application and the host kernel. The runsc runtime integrates with Docker and Kubernetes, making it simple to run sandboxed containers.

      wget https://storage.googleapis.com/gvisor/releases/nightly/latest/runsc
      
      wget https://storage.googleapis.com/gvisor/releases/nightly/latest/runsc.sha512
      
      sha512sum -c runsc.sha512
      
      chmod a+x runsc
      
      sudo mv runsc /usr/local/bin
      
##    Install container runtime (docker)

      Installation steps as per : https://docs.docker.com/install/linux/docker-ce/ubuntu/
      
##    Download worker node components 

      wget https://dl.k8s.io/v1.13.0/kubernetes-node-linux-amd64.tar.gz
      
      gzip -d kubernetes-node-linux-amd64.tar.gz
      
      tar xvf kubernetes-node-linux-amd64.tar
      
      mkdir -p /var/lib/kubelet /var/lib/kube-proxy    /var/lib/kubernetes /var/run/kubernetes
      
      cd kubernetes/node/bin
      
      chmod +x *
      
      cp * /usr/local/bin/
      
##    Install CNI plugin archive

      wget https://github.com/containernetworking/plugins/releases/download/v0.7.4/cni-plugins-amd64-v0.7.4.tgz
      
      mkdir -p /etc/cni/net.d  /opt/cni/bin 
      
      tar -xvf cni-plugins-amd64-v0.7.4.tgz -C /opt/cni/bin/
      
##    Configure kubelet

      a.    The below steps are to be performed on all worker nodes
      
      b.    export HOSTNAME=`hostname`
      
      c.    mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
      
      d.    mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
      
      e.    mv ca.pem /var/lib/kubernetes/
      
      f.    Create configfile for kubelet - /var/lib/kubelet/kubelet-config.yaml
      
      cat << EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
      kind: KubeletConfiguration
      apiVersion: kubelet.config.k8s.io/v1beta1
      authentication:
        anonymous:
          enabled: false
        webhook:
          enabled: true
        x509:
          clientCAFile: "/var/lib/kubernetes/ca.pem"
      authorization:
        mode: Webhook
      clusterDomain: "cluster.local"
      clusterDNS: 
        - "10.32.0.10"
      runtimeRequestTimeout: "15m"
      tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
      tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
      EOF
      
      g.    Create systemd unit file for kubelet - /etc/systemd/system/kubelet.service
      
      cat << EOF | sudo tee /etc/systemd/system/kubelet.service
      [Unit]
      Description=Kubernetes Kubelet
      Documentation=https://github.com/kubernetes/kubernetes
      After=docker.service
      Requires=docker.service

      [Service]
      ExecStart=/usr/local/bin/kubelet \\
        --config=/var/lib/kubelet/kubelet-config.yaml \\
        --image-pull-progress-deadline=2m \\
        --kubeconfig=/var/lib/kubelet/kubeconfig \\
        --network-plugin=cni \\
        --register-node=true \\
        --v=2 \\
        --hostname-override=${HOSTNAME} \\
        --allow-privileged=true
      Restart=on-failure
      RestartSec=5

      [Install]
      WantedBy=multi-user.target
      EOF
      
##    Configure kube-proxy

      a.    mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
      
      b.    Create Kubeproxy config yaml file - 
      
      cat << EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
      kind: KubeProxyConfiguration
      apiVersion: kubeproxy.config.k8s.io/v1alpha1
      clientConnection:
        kubeconfig: "/var/lib/kube-proxy/kubeconfig"
      mode: "iptables"
      clusterCIDR: "10.200.0.0/16"
      EOF
      
      c.    Create systemd unit file -
      
      cat << EOF | sudo tee /etc/systemd/system/kube-proxy.service
      [Unit]
      Description=Kubernetes Kube Proxy
      Documentation=https://github.com/kubernetes/kubernetes

      [Service]
      ExecStart=/usr/local/bin/kube-proxy \\
        --config=/var/lib/kube-proxy/kube-proxy-config.yaml
      Restart=on-failure
      RestartSec=5

      [Install]
      WantedBy=multi-user.target
      EOF

##    Start worker node 

      a.    systemctl daemon-reload
      
      b.    systemctl restart kubelet kube-proxy
      
      c.    systemctl status kubelet kube-proxy
      
      d.    log in to any controller node and run - kubectl get nodes to get all nodes 
      

##    Manage remote kubectl for remote cluster access

      a.    The following steps will be done on clientnode machine 
      
      b.    kubectl config set-cluster mycluster  --certificate-authority=ca.pem --embed-certs=true  --server=https://loadbalancer:6443
      
      c.    kubectl config set-credentials admin --client-certificate=admin.pem --client-key=admin-key.pem
      
      d.    kubectl config set-context mycluster --cluster=mycluster --user=admin
      
      e.    kubectl config use-context mycluster
      
      f.    Verify using kubectl get nodes / kubectl get pods 
      
##    Configure Kubernetes networking (CALICO)

      a.    https://kubernetes.io/docs/concepts/cluster-administration/networking/
      
      b.    Cluster network architecture
            
            cluster-cidr - IP range used to assign IPs to pods. We have configured this as 10.200.0.0/16 on systemd unit file of controller manager
            
            service-cluster-ip-range - IP range used to assign IPs to services. We have configured this as 10.32.0.0/16
            
            pod_cidr - IP range of pods within a specific node. This should fall within the range of cluster-cidr and is managed by CNI
            
      c.    Download calico.yaml 
            
            curl https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml -O
            
      d.    Find CALICO_IPV4POOL_CIDR in calico.yaml and change the range from 192.168.0.0/16 to the value set for cluster-cidr (10.200.0.0/16)
      
      e.    kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
      
      f.    kubectl apply -f calico.yaml
      
##    Configure DNS service

      a.    kubectl create -f https://storage.googleapis.com/kubernetes-the-hard-way/kube-dns.yam
      
      b.    git clone https://github.com/coredns/deployment.git
      
      c.    cd coredns/kubernetes
      
      d.    apt-get install jq 
      
      e.    ./deploy.sh > coredns.yaml
      
      f.    kubectl apply -f coredns.yaml
      
      g.    kubectl delete --namespace=kube-system deployment kube-dns
      
      


      
      
      
      
      
      

      
      







        

      
      
      
      
      

