## DEPLOY KUBERNETES WITH OPENCONTRAIL CNI ON AWS

### LAUNCH INSTANCES ON AWS (Manual)

    * Ansible Instance  (x1)
    
      This is the instance from which, we will run our playbooks
    
        IMAGE:      Ubuntu 16.04 LTS
        FLAVOR:     t2.micro
        DISK:       20 GB
        SEC GRP:    Allow all traffic from everywhere
    
    * K8s-Master Instance   (x1)
    
      This is the instance which will host Kubernetes & OpenContrail control-plane services (OpenContrail Controller)
    
        IMAGE:      Ubuntu 16.04 LTS
        FLAVOR:     t2.xlarge
        DISK:       250 GB
        SEC GRP:    Allow all traffic from everywhere
    
    * K8s-Node Instance    (x1)
    
      This is the instance which will host Kubernetes & OpenContrail data-plane services (OpenContrail vRouter)
    
        IMAGE:      Ubuntu 16.04 LTS
        FLAVOR:     t2.xlarge
        DISK:       250 GB
        SEC GRP:    Allow all traffic from everywhere

    NOTE: Make sure to launch the instances in the same subnet & remember to select the auto-assign public IP option

### PREPARE THE NODES FOR DEPLOYMENT

    * Run these commands on all nodes. This will enable root access with password
    
            (all-nodes)# sudo su
            (all-nodes)# passwd
            (all-nodes)# sed -i -e 's/PermitRootLogin prohibit-password/PermitRootLogin yes/g' -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
            (all-nodes)# service sshd restart
            
    * Install dependencies
    
            (all-nodes)# apt-get update -y && apt-get install python -y
    
    * Logout & login as root user. Enable passwordless SSH for k8s master & node
   
            (ansible-node)# ssh-keygen -t rsa
            
          Make sure you utilize the internal hostname of the AWS instances while copying the SSH keys
    
          Eg. ssh-copy-id root@ip-172-31-14-25
    
            (ansible-node)# ssh-copy-id root@<k8s-master>
            (ansible-node)# ssh-copy-id root@<k8s-node>
      
            (ansible-node)# cd /root
            (ansible-node)# git clone https://github.com/savithuml/opencontrail
      
    * Populate hosts file with k8s-master & k8s-node info. Remember to use the private IP address of the instance
    
            (ansible-node)# cat /root/opencontrail/kubernetes/hosts
       
                  [masters]
                  10.10.10.1

                  [nodes]
                  10.10.10.2
                  
     * Install ansible
     
            (ansible-node)# apt-get install python-pip -y
            (ansible-node)# pip install ansible
            
        
 ### RUN THE PLAY
 
            (ansible-node)# cd /root/opencontrail/kubernetes
            (ansible-node)# ansible-playbook -i hosts site.yml
