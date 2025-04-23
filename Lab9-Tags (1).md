## Tags
Tags can be used for selective execution, task grouping, and debugging in a playbook.

Create the Playbook. Each task will be tagged so that you can see how tags allow for selective task execution.
```
cd ~/ansible-labs
```
```
vi tags.yaml
```
```
- name: Tags Demonstration 
  hosts: localhost
  become: yes
  tasks:
    - name: Check if file '/tmp/file1.txt' exists
      stat:
        path: /tmp/file1.txt
      register: file1_status
      tags: check_file1

    - name: Check if file '/tmp/file2.txt' exists
      stat:
        path: /tmp/file2.txt
      register: file2_status
      tags: check_file2

    - name: Check if directory '/tmp/mydir' exists
      stat:
        path: /tmp/mydir
      register: dir_status
      tags: check_directory

    - name: Check if user 'testuser' exists
      command: id -u testuser
      register: user_status
      failed_when: false
      changed_when: false
      tags: check_user

```
Running Specific Tasks Using Tags
```
ansible-playbook tags.yaml --tags "check_file1"
```
Running Multiple Tags Together
```
ansible-playbook tags.yaml --tags "check_file1,check_file2"
```
Skipping Specific Tasks Using --skip-tags
```
ansible-playbook tags.yaml --skip-tags "check_user"
```
Skipping Multiple Tasks Using --skip-tags
```
ansible-playbook tags.yaml --skip-tags "check_file1,check_directory"
```
Starting from a Specific Task Using --start-at-task
```
ansible-playbook tags.yaml --start-at-task "Check if directory '/tmp/mydir' exists"
```


