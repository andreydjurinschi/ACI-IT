## Лабораторная работа: Автоматизация развертывания многоконтейнерного приложения с Docker Compose с использованием Ansible

### Цель работы

Закрепить знания по Docker и Docker Compose путем автоматизации их установки и развертывания на удаленных виртуальных машинах с помощью Ansible. Студенты научатся объединять инструменты конфигурационного управления (Ansible) с контейнеризацией (Docker), создавая reproducible инфраструктуру. Это позволит понять, как в реальных сценариях DevOps Ansible используется для оркестрации контейнеров на нескольких хостах.

### Задания

![](/lab_04/Screenshot_1.png)

__________________

- создаю директорию для лабы 

```bash
aiai@PC:~$ mkdir ansible_lab04
aiai@PC:~$ cd ansible_lab04
aiai@PC:~/ansible_lab04$ 
```

- первый плейбук (установка Docker)

```bash
- name: install docker # заголовок
  hosts: docker_hosts # запуск на всех хостах группы 
  become: yes # комманды от root 
    # таски
  tasks:
    - name: install packages 
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        state: present
        update_cache: yes
    # скачиваем ключик
    - name: docker key
      ansible.builtin.shell: |
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
      args:
        warn: false
    # подключаем репозиторий
    - name: Add Docker repository
      copy:
        content: |
          deb [arch=amd64 signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable
        dest: /etc/apt/sources.list.d/docker.list
    
    # обновление пакетов
    - name: Update apt cache
      apt:
        update_cache: yes

    # установка докер ядра
    - name: Install Docker Engine
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-buildx-plugin
          - docker-compose-plugin
        state: latest

    # добавление пользователя в группу
    - name: Add current user to docker group
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes

    # запуск и вклюбчение сервиса
    - name: Enable & start Docker
      systemd:
        name: docker
        enabled: yes
        state: started
```

- добавляю группу 

```bash
[web]
localhost ansible_connection=local
[docker_hosts]
localhost ansible_connection=local
```

```bash
aiai@PC:~/ansible_lab04$ ansible-playbook -i ~/ansible-lab03/hosts.ini docker_install.yml -K
---
PLAY RECAP ***************************************************************************************************************************************************************************
localhost                  : ok=11   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- docker-compose.yml

```yml

services:
  db:
    image: mysql:8.0
    container_name: wp_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: wppass
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    container_name: wp_app
    depends_on:
      - db
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    restart: always

volumes:
  db_data:

```

- плейбук на деплой

```yml
- name: Deploy WordPress + MySQL stack
  hosts: docker_hosts
  become: yes

  tasks:
    - name: Copy docker-compose.yml to remote host
      copy:
        src: docker-compose.yml
        dest: /home/aiai/ansible_lab04/docker-compose.yml
        owner: aiai
        group: aiai
        mode: '0644'

    - name: Start stack
      command: docker compose -f /home/aiai/ansible_lab04/docker-compose.yml up -d
      args:
        chdir: /home/aiai

    - name: Check running containers
      command: docker ps
      register: docker_ps

    - name: Show docker ps output
      debug:
        var: docker_ps.stdout_lines
```

```bash
aiai@PC:~/ansible_lab04$ ansible-playbook -i ~/ansible-lab03/hosts.ini deploy.yml -K
BECOME password: 

PLAY [Deploy WordPress + MySQL stack] ************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************
ok: [localhost]

TASK [Copy docker-compose.yml to remote host] ****************************************************************************************************************************************
changed: [localhost]

TASK [Start stack] *******************************************************************************************************************************************************************
changed: [localhost]

TASK [Check running containers] ******************************************************************************************************************************************************
changed: [localhost]

TASK [Show docker ps output] *********************************************************************************************************************************************************
ok: [localhost] => {
    "docker_ps.stdout_lines": [
        "CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS                  PORTS                                     NAMES",
        "fca9509ce712   wordpress:latest   \"docker-entrypoint.s…\"   1 second ago    Up Less than a second   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   wp_app",
        "5c34926d5048   mysql:8.0          \"docker-entrypoint.s…\"   2 seconds ago   Up 1 second             3306/tcp, 33060/tcp                       wp_db"
    ]
}

PLAY RECAP ***************************************************************************************************************************************************************************
localhost                  : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

![](/lab_04/Screenshot_2.png)