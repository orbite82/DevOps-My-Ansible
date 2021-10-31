# Ansible #

This README would normally document whatever steps are necessary to get your application up and running.

## Pre-req

* Máquina t2.medium
* ubuntu 18 ou 20


* __Install Ansible 1.0:__
  * Link de referência
  * [Link](https://www.youtube.com/watch?v=3KPhg4KXz3I&t=633s)

  ```
  sudo -i

  sudo apt update && sudo apt upgrade -y

  sudo apt install software-properties-common -y

  sudo add-apt-repository --yes --update ppa:ansible/ansible

  sudo apt update && sudo apt upgrade -y

  sudo apt install ansible -y

  ansible --version

    ansible 2.9.6
    config file = /etc/ansible/ansible.cfg
    configured module search path = ['/var/lib/rundeck/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
    ansible python module location = /usr/lib/python3/dist-packages/ansible
    executable location = /usr/bin/ansible
    python version = 3.8.5 (default, May 27 2021, 13:30:53) [GCC 9.3.0]

    ssh-keygen

    ou

    ssh-keygen -q -t rsa -f /root/.ssh/key -N ''

    cat /root/.ssh/id_rsa.pub
  ```
* __Acessar máquina de destino:__

  *  Abrir com vim o caminho abaixo para adicionar a chave do ansible no host de destino
  ```
  root@ip-172-30-1-94:~$ sudo -i

  root@ip-172-30-1-94:~$ ls -la /root

  root@ip-172-30-1-94:~$ ls -la /root/.ssh/

  vim /root/.ssh/authorized_keys
  ```

* __Acessar máquina Ansible e jogar a chave na máquina de destino:__

  * Segue exemplo:

  ```
  root@ip-172-:~$ vim /root/.ssh/authorized_keys

    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCoRWQmx3ajThe3nASiVI6YJ4IUDKP0M795ABlvSAhXehhKA76ZeUObYhp35bIGOx0gDFxc2RVmKrBDWsd4cYQWGEWIaGQL63atTF2h2GiMyFz9lSNWjmTYq5GKWTmkFAOJJxGbuvtLFY5OnIqFZRCrkDl2G5pxqc7SurqKd0ZrDK0WvLJI3wVH0/zbLdrqr8VZ83G7J3nLaeB3TK3H78544jlso+u/KtGAiJ52nonNIRpe1Mnf09Ub59Q5Bwou8KR7BMCxgJl2z/6aFU9bn7FI6iAbULf8hqRi8YObx27pf7ecZgVY80RvVF4iy022c86nCcOHS5Q8TAe8u5/2hpLnQEQb8jjN9N2tuiXFRss/8+umE7zAcWxsIA8HodOeUMQgqxjEbQkYuNgZY3JxpsHH+XN1MZQypUFeUntG4Nqnx+jRbmkzXxYrv/kOA1aBYEOa2NWYaGZsPnTFmpSIIznnbSMeLsZq5dFyvmKKPMpfDvimc4myq7aOFV2Dylvsew0= root@ip-172-
  ```
* __Dentro da máquina Ansible fechar ssh para a máquina de destino:__

  ```
  root@ip-172-:~$ ssh 172.30.1.0

  root@ip-172-:~$ yes

  root@ip-172-:~$ logout

  sudo -i

  ```
* __Inventario de Hosts:__

  * Fica neste caminho

  ```
  vim /etc/ansible/hosts

  ``` 
   * Corrigir erro ao adicionar host ex:

  ```
   #Forma Padrão de add:
   [devops]
   #[devops]
   172.30.1.0

  ```
  # Forma para corrigir e seta a versão do python

  ```
   [devops_nexus]
   #devops-nexus
   172.30.1.54 ansible_python_interpreter=/usr/bin/python3

  ```
  * Não são todos os casos que isso ocorre, segue exemplo do erro encontrado abaixo

  * [Link](https://www.digitalocean.com/community/questions/ansible-problem-shared-connection-to-server-closed)

   ```
    172.30.1.251 | FAILED! => {
        "changed": false,
        "module_stderr": "Shared connection to 172.30.1.251 closed.\r\n",
        "module_stdout": "/bin/sh: 1: /usr/bin/python: not found\r\n",
        "msg": "The module failed to execute correctly, you probably need to set the interpreter.\nSee stdout/stderr for the exact error",
        "rc": 127
      }

   ```
* __Inventario de Hosts:__

  * Comandos uteis

  * [Link](https://www.digitalocean.com/community/tutorials/como-instalar-e-configurar-o-ansible-no-ubuntu-18-04-pt)

  * [Link](https://www.tadeubernacchi.com.br/ansible-primeiros-passos-e-exemplos/) 

   ``` 
    ansible devops -a "uptime" -u root

    ansible devops  -a "df -h" -u root

    ansible all -m ping -u root

   ```  

* __Executar um comando direto em um Host especifico:__

  * Verificar como esta no inventario de host para executar o comando 

  * Exemplo:

  ```
  [devops]
  #[devops]
  172.30.1.0

  [prd_]
  #[prd-]
  172.30.1.0
  
  [devops_nexus]
  #devops-nexus
  172.30.1.54 ansible_python_interpreter=/usr/bin/python3
  
  ```
  * Executando comando na máquina devops, como esta descrito na primeira linha do arquivo hosts

  * vim /etc/ansible/hosts

  ```
   ansible devops-geral_dev_a "uptime" -u root

  ``` 
* __Executar arquivos de playbook:__

  * Cuidado sempre quando for executar verificar dentro do arquivo .yml o host de destino e se esta correto o apotamento
  * Um erro no host de destino pode ser fatal
  * Segue estrutura que ainda passa por melhorias

  ```
  root@ip-172-:~# tree /etc/ansible/
    /etc/ansible/
    ├── ansible.cfg
    ├── hosts
    └── playbooks
        ├── apps.yml
        ├── docker.yml
        └── teste.yml
  ```
 
  * Em hosts é o local que vc vai add e salvar o arquivo da máquina que quer efetuar a automação

  ```  
    root@ip-172-:~# cat /etc/ansible/playbooks/apps.yml 
    ---
    - name: Preparing Workstation
    hosts: 
    tasks:

        - name: Installing Linux Apps
        become: true
        apt:
            name: '{{ item }}'
            install_recommends: yes
            state: present
        loop:
            - vim
            - htop
            - curl
            - wget
            - tree
            - apt-transport-https
            - python3-pip
            - git
            - bash-completion
  ```

* __Executar um playbook após setar um Host no yml :__

  * Existem duas forma dentro ou fora do diretório

  ```
  ansible-playbook -i /etc/ansible/playbooks/apps.yml
  ansible-playbook apps.yml
  
  ```
