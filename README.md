# ansible_playbook2
Modify the winconfig.yml playbook to install and start the openssh server on windows in addition to the other configurations. Test by ssh'ing to the windows machine from your linux machine.

```

#configMgmt4
#20232405
#create local user, group, directoy and share the c drive shared AND SSH CONNECTION



- name: install sshpass or linux
  hosts: linux1
  tasks:
      - name: install sshpass
        apt:
          name: sshpass
          state: present


#create a play book
- name: create playbook
  hosts: windows1
  vars:
    ansible_connection: ssh
    #ansible_user: User1
    #ansible_password: B0bP4ssw0rd
    ansible_connection: winrm

  tasks:

#add group
    - name: create group
      ansible.windows.win_group:
        name: Group1
        description: deploy Group1
        state: present

# add user
    - name: create user 
      ansible.windows.win_user:
        name: User1
        password: B0bP4ssw0rd
        state: present
        groups:
         - Group1 # create
        home_directory: c:\Shared  # create


#Create Path
    - name: Touch file
      ansible.windows.win_file:
        path: C:\Shared        # gruop1 needs access
        state: directory
        owner: User1
        group: Group1

# install open ssh server on windows

    - name: install open ssh
      win_chocolatey:
        name: openssh
        package_params: /SSHServerFeature
        state: present

    - name: set the default shell to PowerShell
      win_regedit:
        path: HKLM:\SOFTWARE\OpenSSH
        name: DefaultShell
        data: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
        type: string
        state: present

# THIS LETS ME LOG IN BUT DOESNT SHOW ME AN INTERACTIVE SHELL
- name: log in with ssh
  hosts: linux1
  tasks:
    - name: Execute SSH command from Linux to Windows
      ansible.builtin.raw: ssh -t User1@x.x.x.x 'echo Hello from Windows'
      delegate_to: localhost

# THIS ONE LET ME LOG IN ERRORS OUT      
# - name: SSH into Windows and run command
#   hosts: linux1
#   tasks:
#     - name: Run SSH command
#       ansible.builtin.shell: ssh -t User1@x.x.x.x
#       delegate_to: localhost


```
