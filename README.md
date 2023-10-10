05.Homework Ansible start


***************************************************************************************************
ssh-copy-id username@remote_host     копирование ключа на удаленный сервер
ansible-inventory -i <файл инвенторя>    просмотр инветаря (ansible-inventory -i inv.yaml --graph)
ansible -i inv.yaml -m authorized_key -a "key=\"{{lookup('file','~/.ssh/id_rsa.pub')}}\" user=root" all_workers -u root --ask-pass добавить публичный ключ на удаленные хосты


***************************************************************************************************


#**Homework Assignment 1: Setting Up Ansible**
1. Install Ansible on your local machine or a virtual environment.
2. Create a directory for your Ansible playbooks and configuration files.
3. Write an Ansible playbook that prints "Hello, Ansible!" to the console.
4. Run the playbook using the ansible-playbook command and ensure it executes successfully.
5. Document the installation process and the steps you took to run the playbook.
--------------------------------------------------------------------------------------------
Домашнее задание 1: Настройка Ansible
1. Установите Ansible на свой локальный компьютер или в виртуальную среду.
2. Создайте каталог для ваших сборников Ansible playbooks и файлов конфигурации.
3. Напишите Ansible playbook, который выводит "Привет, Ansible!" на консоль.
4. Запустите playbook с помощью команды ansible-playbook и убедитесь, что она успешно выполнена.
5. Задокументируйте процесс установки и шаги, которые вы предприняли для запуска playbook.
=================================================================================================

1. Install Ansible on your local machine or a virtual environment

mkdir 05.Ansible
cd 05.Ansible
sudo apt update
sudo apt install python3-pip
sudo pip3 install ansible
# Checking the version. Must be at least 2.10  
ansible --version



2. Create a directory for your Ansible playbooks and configuration files.

** Сreating inventory**
# The sample is taken from "demo"  [https://github.com/pluhin/sa.it-academy.by/tree/md-sa2-25-23/demo/05.Ansible]
nano inv.yaml

** Creating a connection via bastion server**

# Copy the ssh key to the remote server
ssh-copy-id -p 32510 jump_sa@178.124.206.53
exit
# Сonnecting to the bastion server
ssh jump_sa@178.124.206.53 -p 32510 
# Setting up connection to Bastion server via config file  [https://github.com/pluhin/sa.it-academy.by/blob/md-sa2-25-23/demo/05.Ansible/config]
nano ~/.ssh/config    
# Checking connect host19 host20
ssh root@192.168.202.19
exit
ssh root@192.168.202.20
exit


** Настройка Ansible**

 * Добавляем ProxyCommand в конфиг инвентаря внутри группы all_workers*
# Закоментируем конфиг для ssh подключения через Bastion server
nano ~/.ssh/config 
mkdir -p group_vars/all_workers
nano group_vars/all_workers/vars.yaml   имя файла задается любое, папка должна иметь название группы ( all_workers)
# Add argument       ansible_ssh_common_args: '-o ProxyCommand="ssh -W %h:%p -q jump_sa@178.124.206.53  -p 32510"'
    
    * Create ansible.cfg *
# In the 05.Ansible directory, we create ansible.cfg  [https://github.com/pluhin/sa.it-academy.by/blob/md-sa2-25-23/demo/05.Ansible/ansible.cfg]
# Where, forks - number of simultaneous connections; 
nano ansible.cfg
 
* Устанавливаем утилиту sshpass для возможности подключения по паролю
sudo apt install sshpass

* Test run ansible *
# We connect to a remote host and check if Python is installed there
ansible -i inv.yaml -m ping all_workers -u root --ask-pass
 
 * Добавляем ssh public key в хранилище пользователя на удаленном хосте
ansible -i inv.yaml -m authorized_key -a "key=\"{{lookup('file','~/.ssh/id_rsa.pub')}}\" user=root" all_workers -u root --ask-pass
# Check 
ansible -i inv.yaml -m ping all_workers -u root


** Ansible. Vault **
ansible-vault create group_vars/all_workers/vault.yaml
# Volt pass 1804
nano inv.yaml
# Copy "ansible_user: root"
ansible-vault edit group_vars/all_workers/vault.yaml
# Paste ansible_user: root
#Check
cat group_vars/all_workers/vault.yaml
#Check
ansible -i inv.yaml -m ping all_workers --ask-vault-pass


** Ansible. Task **

3. Write an Ansible playbook that prints "Hello, Ansible!" to the console.

nano hello_loc.yml
ansible-playbook hello_loc.yml
