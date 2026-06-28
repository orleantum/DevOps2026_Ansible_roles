#### Создание структуры

```Bash
mkdir -p {group_vars,roles} && touch ansible.cfg group_vars/all.yml inventory.yml playbook.yml
cd roles/
ansible-galaxy init Nginx
touch roles/Nginx/templates/index.html.j2
```

<img width="289" height="518" alt="image" src="https://github.com/user-attachments/assets/e5e96d5e-670d-4838-aa78-9d83fdd1dc12" />



#### Файл `playbook.yml`:

```YAML
---
- name: Deploy Nginx
  hosts: myhosts
  remote_user: iadiyanov
  become: yes
  become_method: sudo

  vars_files:
    - vault.yml

  roles:
    - Nginx
```


#### Файл `inventory.yml`:

```YAML
---
myhosts:
  hosts:
    10.0.2.3:
      ansible_ssh_private_key_file: "{{ vault_ssh_key_path }}"
      ansible_become_password: "{{ vault_sudo_password }}"
```


#### Файл `vault.yml`

Используется для шифрования:
- `vault_ssh_key_path`
- `vault_sudo_password`

Чтобы изменить редактор по умолчанию с `vim` на `nano` нужно добавить переменную окружения, добавим её в конфигурационный файл оболочки:

```Bash
cd ..
nano .bashrc
```

В конце файла дописать:

```Bash
export EDITOR=nano
```

Применить изменения:

```Bash
source ~/.bashrc
```

Создать `vault.yml`:

```Bash
ansible-vault create vault.yml
```

Vault password: Qwerty12345!

Чтобы просмотреть:
```Bash
ansible-vault view vault.yml
```

<img width="522" height="62" alt="image" src="https://github.com/user-attachments/assets/391ea7d3-b08b-41f7-9eca-825ee6eb17c1" />


#### Файл `ansible.cfg`

Используется для просмотра `vault.yml` без необходимости ввода пароля

Файл глобальных настроек называется `ansible.cfg` (смысловое имя), но парсится как INI (`.ini`).

```INI
[defaults]
vault_password_file = ~/.ansible_vault_pass
```


#### Файл `roles/Nginx/tasks/main.yml`:

```YAML
---
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: yes

- name: Remove default Nginx virtual host (Ubuntu specific)
  ansible.builtin.file:
    path: /etc/nginx/sites-enabled/default
    state: absent
  notify: restart nginx

- name: Create DocumentRoot directories for vhosts
  ansible.builtin.file:
    path: "/var/www/{{ item }}"
    state: directory
    owner: iadiyanov
    group: iadiyanov
    mode: '0755'
  loop: "{{ sites }}"

- name: Create index.html from template
  ansible.builtin.template:
    src: 'index.html.j2'
    dest: "/var/www/{{ item }}/index.html"
    owner: iadiyanov
    group: iadiyanov
    mode: '0644'
  loop: "{{ sites }}"

- name: Copy nginx config from template
  ansible.builtin.template:
    src: 'site.conf.j2'
    dest: '/etc/nginx/conf.d/site-{{ item }}.conf'
    owner: root
    group: root
    mode: '0644'
  loop: '{{ sites }}'
  notify: restart nginx

- name: Ensure Nginx is started and enabled on boot
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: yes
```


#### Файл `roles/Nginx/handlers/main.yml`:

```YAML
---
- name: restart nginx
  ansible.builtin.services:
    name: nginx
    state: restarted
```


#### Файл `roles/Nginx/templates/site.conf.j2`:

```conf
server {
    listen 80;
    server_name {{ item }} www.{{ item }};

    root /var/www/{{ item }};
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```


#### Файл `roles/Nginx/templates/index.html.j2`:

```HTML
<HTML>
  <BODY>
    Hello! Site: {{ item }}
  </BODY>
</HTML>
```


#### Файл `group_vars/all.yml`:

```YAML
---
sites:
  - site1.ru
  - site2.com
  - site3.gov
```


#### Итоговая структура:

<img width="289" height="574" alt="image" src="https://github.com/user-attachments/assets/d378c664-c53d-4913-8982-d12c9bcaacbd" />


#### Тестовый запуск:

<img width="1092" height="748" alt="image" src="https://github.com/user-attachments/assets/9210a2f1-3628-434a-9018-e5c43d297dc6" />



#### Настоящий запуск:

<img width="1094" height="751" alt="image" src="https://github.com/user-attachments/assets/48772385-27fa-4782-bf70-3bffab549106" />


В `C:\Windows\System32\drivers\etc` необходимо прописать:

```
127.0.0.1 site1.ru site2.com site3.gov
```


#### Проверка доступности:

<img width="580" height="346" alt="image" src="https://github.com/user-attachments/assets/0c875ce4-9d55-4e47-b388-6699e820b58a" />

<img width="970" height="466" alt="image" src="https://github.com/user-attachments/assets/98ec1a44-3d81-4be2-8e98-de73e30787c9" />

<img width="1012" height="713" alt="image" src="https://github.com/user-attachments/assets/2be7f2a3-9722-41ff-a3ed-76c32e2e5ea2" />

<img width="972" height="524" alt="image" src="https://github.com/user-attachments/assets/1ee31cc5-dd33-46d9-a517-2bebcf7ab83c" />
