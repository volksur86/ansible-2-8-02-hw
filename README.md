Домашнее задание к занятию «Ansible.Часть 2» Волкивский Андрей

Задание 1

Выполните действия, приложите файлы с плейбуками и вывод выполнения.

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.

Плейбуки должны:

1.1. Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать официальный сайт и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.

1.2. Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.

1.3. Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.

Решение 1

1.1
```yaml
---
- name: Download and extract Apache Kafka
  hosts: localhost
  connection: local
  tasks:
    - name: Create directory for Kafka
      become: yes
      file:
        path: /opt/kafka
        state: directory
        mode: '0755'

    - name: Download Kafka
      get_url:
        url: "https://downloads.apache.org/kafka/4.1.0/kafka_2.13-4.1.0.tgz"
        dest: /tmp/kafka.tgz
        mode: '0644'

    - name: Extract Kafka
      become: yes
      unarchive:
        src: /tmp/kafka.tgz
        dest: /opt/kafka
        remote_src: yes
        extra_opts: [--strip-components=1]
        creates: /opt/kafka/bin
```

<img width="1138" height="956" alt="image" src="https://github.com/user-attachments/assets/63e8864d-b209-4b5d-8739-591c96ff981c" />

1.2
```yaml
---
- name: Install and configure Tuned
  hosts: localhost
  connection: local
  tasks:
    - name: Install Tuned
      become: yes
      apt:
        name: tuned
        state: present
        update_cache: yes

    - name: Start and enable Tuned service
      become: yes
      systemd:
        name: tuned
        state: started
        enabled: yes
```

<img width="1144" height="957" alt="image" src="https://github.com/user-attachments/assets/3390d743-5ca1-4ca2-860d-993e00511e2d" />

1.3
```yaml
---
- name: Change MOTD
  hosts: localhost
  connection: local
  vars:
    custom_motd: "Добро пожаловать на урок Ansible"
  tasks:
    - name: Update MOTD
      become: yes
      copy:
        content: "{{ custom_motd }}\n"
        dest: /etc/motd
        mode: '0644'
```

<img width="1182" height="726" alt="image" src="https://github.com/user-attachments/assets/1b572688-659e-48d3-a7be-2764fe0f5d30" />

Задание 2
Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору.

Решение 2
```yaml
---
- name: Hello IP, HOSTNAME and wishing good day
  hosts: localhost
  become: yes

  tasks:
    - name: Info host
      setup:
        gather_subset:
          - network
          - hardware

    - name: welcome file /etc/motd
      copy:
        content: |
          Добро пожаловать на урок Ansible {{ ansible_hostname }}!
          IP-адрес: {{ ansible_default_ipv4.address }}
          Системный администратор, желаю хорошего дня!
        dest: /etc/motd
        owner: root
        group: root
        mode: '0644'
```

<img width="1146" height="955" alt="image" src="https://github.com/user-attachments/assets/759e69b6-e021-4c22-8c31-6dd49cbca54a" />

Задание 3
Выполните действия, приложите архив с ролью и вывод выполнения.

Ознакомьтесь со статьёй «Ansible - это вам не bash», сделайте соответствующие выводы и не используйте модули shell или command при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

Установить веб-сервер Apache на управляемые хосты.
Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес. Используйте Ansible facts и jinja2-template. Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
Сделать проверку доступности веб-сайта (ответ 200, модуль uri).
В качестве решения:

предоставьте плейбук, использующий роль;
разместите архив созданной роли у себя на Google диске и приложите ссылку на роль в своём решении;
предоставьте скриншоты выполнения плейбука;
предоставьте скриншот браузера, отображающего сконфигурированный index.html в качестве сайта.

Решение 3
```yaml
---
# Установка Apache
- name: Установить Apache2
  apt:
    name: apache2
    state: present
    update_cache: yes
  become: yes

# Запуск и автозагрузка
- name: Запустить Apache2 и добавить в автозагрузку
  systemd:
    name: apache2
    state: started
    enabled: yes
  become: yes

# Создание index.html из шаблона
- name: Создать index.html с информацией о системе
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: www-data
    group: www-data
    mode: '0644'
  notify: restart apache
  become: yes

# Открытие порта 80
- name: Открыть порт 80 в UFW
  ufw:
    rule: allow
    port: '80'
    proto: tcp
  become: yes
  ignore_errors: yes

# Проверка доступности сайта
- name: Проверим доступность веб-сайта (ожидаем ответ HTTP 200)
  uri:
    url: "http://{{ ansible_default_ipv4.address }}/"
    status_code: 200
  register: website_check
  retries: 3
  delay: 2
```




