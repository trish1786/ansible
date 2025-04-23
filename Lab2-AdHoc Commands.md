## Ad-Hoc Commands

```
sudo vi /etc/ansible/hosts
```

Add the given line, by pressing "INSERT" 
add localhost and add the connection as local so that it wont try to use ssh
```
localhost ansible_connection=local
```
**save the file using** `ESCAPE + :wq!`

In real life situations, one of the managed node may be used as the ansible control node. In such cases, we can make it a managed node, by adding localhost in hosts inventory file.

### Get memory details of the hosts using the below ad-hoc command
```
ansible all -m command -a "free -h"
```
![image](https://github.com/user-attachments/assets/8f1fd295-a9df-4356-9b4e-99ee1e68b97a)


### Implicit Module: When the module is not specified, Ansible uses the command module by default.
```
ansible all -a "free -h"
```
![image](https://github.com/user-attachments/assets/a892b7b1-218a-4dc9-b029-6f62d5bc982b)



### Create a user ansible-test in the the control node. 
```
ansible all -m user -a "name=ansible-test" --become
```
![image](https://github.com/user-attachments/assets/db7a27a6-9372-416f-93eb-7b8894e1b87a)



### Lists all users in the machine. Check if ansible-test is present. 
```
ansible localhost -a "cat /etc/passwd"
```
![image](https://github.com/user-attachments/assets/a075beb0-8202-4eba-9a08-54a58e38eafe)



### List all directories in /home. Ensure that directory 'ansible-test' is present in /home. 
```
ansible managed-node2 -a "ls /home"
```
![image](https://github.com/user-attachments/assets/ff6ee057-d9c4-4eaa-89f7-65d306951d0f)



### Change the permission mode from '700' to '755' for the new home directory created for ansible-test
```
ansible managed-node2 -m file -a "dest=/home/ansible-test mode=755" --become
```
![image](https://github.com/user-attachments/assets/19a68b54-0c69-44a7-a2f2-4af1daa868ff)




### Check if the permissions got changed
```
ansible managed-node2 -a "sudo ls -l /home"
```
![image](https://github.com/user-attachments/assets/0a07f0ff-9ea7-4f65-9b15-6ce86fb21222)




### Create a new file in the new dir in node 1
```
ansible managed-node1 -m file -a "dest=/home/ansible-test/demo.txt mode=600 state=touch" --become
```
![image](https://github.com/user-attachments/assets/4ac3f703-0578-4a3c-a7e9-0583ce82cf81)




### Check if the permissions got changed
```
ansible managed-node1 -a "sudo ls -l /home/ansible-test/"
```
![image](https://github.com/user-attachments/assets/f2a6cae8-d66d-4afa-863d-1a3715aa186f)





### Add content into the file
```
ansible managed-node1 -b -m lineinfile -a 'dest=/home/ansible-test/demo.txt line="This server is managed by Ansible"'
```
![image](https://github.com/user-attachments/assets/67ab0482-a044-4248-8965-b3b75eaf503e)



### Check if the lines are added in demo.txt
```
ansible managed-node1 -a "sudo cat /home/ansible-test/demo.txt"
```
![image](https://github.com/user-attachments/assets/bb29e580-b492-432e-850d-bfb2d725dc0f)




### You can remove the line using the parameter state=absent
```
ansible managed-node1 -b -m lineinfile -a 'dest=/home/ansible-test/demo.txt line="This server is managed by Ansible" state=absent'
```
![image](https://github.com/user-attachments/assets/960a14cf-07e6-4917-bcb8-44afe434d6fe)




### Check if the lines are removed from demo.txt
```
ansible managed-node1 -b -a "sudo cat /home/ansible-test/demo.txt"
```
![image](https://github.com/user-attachments/assets/69ecf6d3-2553-43df-8c7a-252b8b1e8b09)




### Copy a file from ansible-control node to host managed-node1
```
touch test.txt
```
```
echo "This file will be copied to managed node using copy module" >> test.txt
```
Now Copy. --become can be replaced by -b
```
ansible managed-node1 -m copy -a "src=test.txt dest=/home/ansible-test/test" -b 
```
![image](https://github.com/user-attachments/assets/16cb9bba-68ee-4af7-b36a-85c68b51d942)



### Check if the file got copied to managed node.
```
ansible managed-node1 -b -a "sudo ls -l /home/ansible-test/test"
```
![image](https://github.com/user-attachments/assets/9bbae2a7-1987-4c14-b815-b585cc4fad1d)


### clean up
Delete the users
```
ansible all -m user -a "name=ansible-test state=absent" --become
```
![image](https://github.com/user-attachments/assets/63cbf363-7cbb-4ab2-9cc0-032545c3dd11)

Delete the directories
```
ansible all -a "rm -rf /home/ansible-test" --become
```
![image](https://github.com/user-attachments/assets/50171768-1b7c-4c4a-a749-fc5c9a6b927a)

Cross Verify
```
ansible all -a "id ansible-test"
```
![image](https://github.com/user-attachments/assets/95a04f0e-e6bd-4b1f-bb44-2fc6557104de)

```
ansible all -a "ls /home"
```
![image](https://github.com/user-attachments/assets/2785dbb6-e8c5-4bfc-b5b8-d89a6ba5b181)


Now remove the localhost from the host file
```
sudo vi /etc/ansible/hosts
```
Remove the below line from hosts inventory file. 

`localhost ansible_connection=local`

![image](https://github.com/user-attachments/assets/710199b0-d722-4086-87a1-fa2e88071c72)


**save the file using** `ESCAPE + :wq!`
