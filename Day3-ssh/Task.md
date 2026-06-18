
Task login into servers and disable ssh root login




server : ssh   tony@stapp01
Password : Ir0nM@n

server sh   steve@stapp02 
Password Am3ric@


login  -- vi etc/ssh/sshd_config
   change PermitRootLogin no
   restart systemctl restart sshd


---
Let see how to do it with a `Make file` 
#### production dont use make for such task , use ansible , this is only for learning purpose
- Prerequisites: Before running the Makefile, ensure sshpass is installed on your controlling machine
-  sudo apt-get install sshpass -y

  ```Make
# Define Target Servers and Credentials
SERVER1 = tony@stapp01
PASS1   = Ir0nM@n

SERVER2 = steve@stapp02
PASS2   = Am3ric@

# Command to disable root login and restart SSH daemon
# The 'echo $PASS | sudo -S' injects the password for the sudo prompt on the server
CMD     = "echo '$(1)' | sudo -S sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config && echo '$(1)' | sudo -S systemctl restart sshd"

.PHONY: all server1 server2

all: server1 server2

server1:
	@echo "Configuring $(SERVER1)..."
	@sshpass -p "$(PASS1)" ssh -o StrictHostKeyChecking=no $(SERVER1) $(call CMD,$(PASS1))

server2:
	@echo "Configuring $(SERVER2)..."
	@sshpass -p "$(PASS2)" ssh -o StrictHostKeyChecking=no $(SERVER2) $(call CMD,$(PASS2))

```

 Run It: ` make all `

 ---

 ### Ansible Playbook

 ##### 1. The Inventory File (hosts.ini)
 ```ini
[webservers]
stapp01 ansible_user=tony
stapp02 ansible_user=steve
```

##### 2. The Task Automation File (harden_ssh.yml)
```yaml
---
- name: Secure Production SSH Configurations
  hosts: webservers
  become: yes  # Automatically elevates privileges using sudo safely

  tasks:
    - name: Ensure root login via SSH is disabled
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'
        line: 'PermitRootLogin no'
        state: present
      notify: Restart SSH Service

  handlers:
    - name: Restart SSH Service
      ansible.builtin.service:
        name: sshd
        state: restarted
```
##### 3. Run the Playbook
```bash
ansible-playbook -i hosts.ini harden_ssh.yml -k -K
```

Note : there will be password prompt , and error's which we will pick in ansible section
