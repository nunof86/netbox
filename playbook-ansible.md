# Playbook - Ansible

## Ansible Playbook

1. Replace the <mark style="color:red;">`hosts:`</mark> option to <mark style="color:red;">**localhost**</mark> or other <mark style="color:red;">**host**</mark> of your choice and the credentials.
2. Replace the <mark style="color:red;">`admin`</mark> user and <mark style="color:red;">`admin password`</mark> in the <mark style="color:red;">`Create Superuser if not Exists`</mark> task.
3. Change the <mark style="color:red;">`server_name</mark> and your_ip_address</mark> to match your needs.

```yaml
---

- name: Provision PostgreSQL, Redis, and NetBox with Nginx
  hosts: localhost
  become: yes

  tasks:
    - name: System Update
      apt:
        update_cache: yes
      changed_when: false

    - name: Dist Upgrade
      apt:
        upgrade: dist
      register: upgrade_output

    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present

    - name: Create NetBox Database and User
      shell: |
        sudo -u postgres psql -c "CREATE DATABASE netbox;"
        sudo -u postgres psql -c "CREATE USER netbox WITH PASSWORD 'password';"
        sudo -u postgres psql -c "ALTER DATABASE netbox OWNER TO netbox;"

    - name: Install Redis
      apt:
        name: redis-server
        state: present

    - name: Install Python Packages and Dependencies
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - python3
        - python3-pip
        - python3-venv
        - python3-dev
        - build-essential
        - libxml2-dev
        - libxslt1-dev
        - libffi-dev
        - libpq-dev
        - libssl-dev
        - zlib1g-dev

    - name: Install Git
      apt:
        name: git
        state: present

    - name: Create NetBox Directory
      file:
        path: /opt/netbox/
        state: directory

    - name: Clone NetBox Repository
      git:
        repo: https://github.com/netbox-community/netbox.git
        dest: /opt/netbox/
        version: master
        depth: 1

    - name: Create NetBox System User
      become_user: root
      command: adduser --system --group netbox

    - name: Set Ownership for NetBox Directories
      become: yes
      file:
        path: "{{ item }}"
        state: directory
        owner: netbox
        group: netbox
      loop:
        - /opt/netbox/netbox/media/
        - /opt/netbox/netbox/reports/
        - /opt/netbox/netbox/scripts/

    - name: Copy NetBox Configuration
      command: cp /opt/netbox/netbox/netbox/configuration_example.py /opt/netbox/netbox/netbox/configuration.py

    - name: Modify ALLOWED_HOSTS Option in configuration.py
      lineinfile:
        path: /opt/netbox/netbox/netbox/configuration.py
        regexp: "^ALLOWED_HOSTS = .*"
        line: "ALLOWED_HOSTS = ['*']"
      become: yes

    - name: Replace DATABASE Block in NetBox Configuration File
      replace:
        path: /opt/netbox/netbox/netbox/configuration.py
        regexp: "(?s)(DATABASE = \\{).*?(\\})"
        replace: |
          DATABASE = {
              'ENGINE': 'django.db.backends.postgresql',
              'NAME': 'netbox',
              'USER': 'netbox',
              'PASSWORD': 'password',
              'HOST': 'localhost',
              'PORT': '',
              'CONN_MAX_AGE': 300,
          }
      become: yes

    - name: Generate Secret Key
      command: python3 ../generate_secret_key.py
      args:
        chdir: /opt/netbox/netbox/netbox/
      register: secret_key_output
      when: secret_key_output is not defined

    - name: Update SECRET_KEY in NetBox Configuration File
      lineinfile:
        path: /opt/netbox/netbox/netbox/configuration.py
        regexp: '^SECRET_KEY = .*'
        line: "SECRET_KEY = '{{ secret_key_output.stdout }}'"
      become: yes

    - name: Add django-storages to local_requirements.txt
      shell: echo 'django-storages' >> /opt/netbox/local_requirements.txt
      become: yes

    - name: Run upgrade script
      shell: /opt/netbox/upgrade.sh
      become: yes

    - name: Check if Superuser Exists
      command: >
        bash -c "source /opt/netbox/venv/bin/activate &&
                cd /opt/netbox/netbox &&
                echo -e 'from django.contrib.auth import get_user_model; User = get_user_model(); print(User.objects.filter(username=\"admin\").exists())' | python3 manage.py shell"
      register: superuser_exists
      changed_when: false
      failed_when: false
      become: yes
      become_user: netbox
      environment:
        DJANGO_SETTINGS_MODULE: "netbox.settings"

    - name: Create Superuser if not Exists
      command: >
        bash -c "source /opt/netbox/venv/bin/activate &&
                cd /opt/netbox/netbox &&
                echo -e 'from django.contrib.auth import get_user_model; User = get_user_model(); User.objects.create_superuser(\"admin\", \"admin@example.com\", \"adminpassword\")' | python3 manage.py shell"
      when: not superuser_exists.stdout | bool
      become: yes
      become_user: netbox
      environment:
        DJANGO_SETTINGS_MODULE: "netbox.settings"

    - name: Deactivate Virtual Environment
      command: deactivate
      args:
        executable: /bin/bash
      when: "'VIRTUAL_ENV' in ansible_env"

    - name: Create Symlink for netbox-housekeeping Cron Job
      become: yes
      become_user: root
      file:
        src: /opt/netbox/contrib/netbox-housekeeping.sh
        dest: /etc/cron.daily/netbox-housekeeping
        state: link

    - name: Deactivate Virtual Environment
      command: deactivate
      args:
        executable: /bin/bash
      when: "'VIRTUAL_ENV' in ansible_env"

    - name: Copy gunicorn.py
      command: cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
      become: yes

    - name: Copy Service Files
      copy:
        src: "/opt/netbox/contrib/{{ item }}"
        dest: "/etc/systemd/system/"
        mode: "0644"
      loop: "{{ ['netbox.service', 'netbox-rq.service'] }}"
      become: yes

    - name: Reload Systemd Daemon
      systemd:
        daemon_reload: yes
      become: yes

    - name: Start and Enable NetBox Services
      systemd:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop:
        - netbox
        - netbox-rq
      become: yes

    - name: Generate SSL Certificates
      become: yes
      command: openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
              -keyout /etc/ssl/private/netbox.key \
              -out /etc/ssl/certs/netbox.crt \
              -subj "/C=US/ST=State/L=City/O=Organization/OU=Unit/CN=netbox.example.com"

    - name: Install Nginx
      become: yes
      apt:
        name: nginx
        state: present

    - name: Copy Nginx Configuration
      become: yes
      copy:
        src: "/opt/netbox/contrib/nginx.conf"
        dest: "/etc/nginx/sites-available/netbox"
        mode: "0644"

    - name: Remove Default Nginx Site
      become: yes
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent

    - name: Create Symbolic Link for Netbox Nginx Site
      become: yes
      file:
        src: /etc/nginx/sites-available/netbox
        dest: /etc/nginx/sites-enabled/netbox
        state: link

    - name: Update server_name in Nginx Configuration
      become: yes
      replace:
        path: /etc/nginx/sites-enabled/netbox
        regexp: 'server_name .*;'
        replace: 'server_name your_ip_address;'

    - name: Restart Nginx Service
      become: yes
      service:
        name: nginx
        state: restarted
