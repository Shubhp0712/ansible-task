#### Step 1: Using AWS first provision the EC2 instance .pem key and ssh, http and https inbound rules.

#### Step2: Place the downloaded .pem key inside current working directory and give the 400 permission to it.
``` 
chmod 400 <key-name> 
```

#### Step3: In inventory file add the details such as ec2 public_ip as ansible_host, ec2_username as ansible_user and .pem key file name as ansible_ssh_private_key_file 

#### Step4: Run the playbook with given command.
```
ansible-playbook -i <inventory-file-name> <playbook-name>
```