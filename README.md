 Домашнее задание к занятию "`Практическое задание с самопроверкой «Ansible. Часть 2»
" - `Галицин Николай`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. В личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

`Задание 1
Выполните действия, приложите файлы с плейбуками и вывод выполнения.

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.
Плейбуки должны:

Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать официальный сайт и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.
Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.
Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.


Поле для вставки кода
Плейбук 1: Скачивание и распаковка архива Kafka....
....---
- name: Скачать и распаковать Apache Kafka
  hosts: localhost
  connection: local
  vars:
    kafka_version: "3.6.0"
    kafka_url: "https://downloads.apache.org/kafka/{{ kafka_version }}/kafka_2.13-{{ kafka_version }}.tgz"
    download_dir: "/tmp/kafka_download"
    extract_dir: "/opt/kafka"
  
  tasks:
    - name: Создать директорию для скачивания
      file:
        path: "{{ download_dir }}"
        state: directory
        mode: '0755'
    
    - name: Скачать архив Kafka
      get_url:
        url: "{{ kafka_url }}"
        dest: "{{ download_dir }}/kafka_{{ kafka_version }}.tgz"
        mode: '0644'
    
    - name: Создать директорию для распаковки
      file:
        path: "{{ extract_dir }}"
        state: directory
        mode: '0755'
      become: yes
    
    - name: Распаковать архив Kafka
      unarchive:
        src: "{{ download_dir }}/kafka_{{ kafka_version }}.tgz"
        dest: "{{ extract_dir }}"
        remote_src: yes
        extra_opts: [--strip-components=1]
      become: yes
    
    - name: Проверить содержимое распакованной директории
      find:
        paths: "{{ extract_dir }}"
        file_type: any
      register: kafka_files
    
    - name: Показать распакованные файлы
      debug:
        msg: "Kafka успешно распакована. Найдено {{ kafka_files.files | length }} файлов"
    
    - name: Очистить временные файлы
      file:
        path: "{{ download_dir }}"
        state: absent
Плейбук 2: Установка и запуск tuned
---
- name: Установка и настройка tuned
  hosts: localhost
  connection: local
  become: yes
  
  tasks:
    - name: Обновить кэш пакетов (для apt-based систем)
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"
    
    - name: Установить tuned (RedHat/CentOS)
      yum:
        name: tuned
        state: present
      when: ansible_os_family == "RedHat"
    
    - name: Установить tuned (Debian/Ubuntu)
      apt:
        name: tuned
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Запустить сервис tuned
      systemd:
        name: tuned
        state: started
        daemon_reload: yes
    
    - name: Добавить tuned в автозагрузку
      systemd:
        name: tuned
        enabled: yes
    
    - name: Проверить статус сервиса tuned
      systemd:
        name: tuned
      register: tuned_status
    
    - name: Показать статус tuned
      debug:
        msg: "Tuned статус: {{ tuned_status.status.ActiveState }}"
    
    - name: Показать активный профиль tuned
      command: tuned-adm active
      register: tuned_profile
      changed_when: false
    
    - name: Вывести активный профиль
      debug:
        msg: "Активный профиль: {{ tuned_profile.stdout }}"
Плейбук 3: Изменение MOTD с переменной
---
- name: Изменение приветствия системы MOTD
  hosts: localhost
  connection: local
  become: yes
  vars:
    motd_message: |
      ╔══════════════════════════════════════════════╗
      ║     Добро пожаловать на сервер!              ║
      ║     Это сервер под управлением Ansible        ║
      ║     Дата настройки: 2024 год                  ║
      ╚══════════════════════════════════════════════╝
      
      Все действия логируются!
  
  tasks:
    - name: Создать резервную копию оригинального MOTD
      copy:
        src: /etc/motd
        dest: "/etc/motd.backup_{{ ansible_date_time.date }}"
        remote_src: yes
        force: no
    
    - name: Записать новое приветствие в /etc/motd
      copy:
        content: "{{ motd_message }}"
        dest: /etc/motd
        mode: '0644'
    
    - name: Создать динамический MOTD скрипт
      copy:
        content: |
          #!/bin/bash
          echo "{{ motd_message }}"
          echo "Системная информация:"
          echo "  - ОС: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'=' -f2)"
          echo "  - Ядро: $(uname -r)"
          echo "  - uptime: $(uptime -p)"
          echo ""
          echo "Переменная MOTD: {{ motd_message | length }} символов"
        dest: /etc/update-motd.d/99-custom-message
        mode: '0755'
      when: ansible_os_family == "Debian"
    
    - name: Проверить содержимое /etc/motd
      command: cat /etc/motd
      register: motd_content
      changed_when: false
    
    - name: Показать новое приветствие
      debug:
        msg: |
          Новое приветствие MOTD:
          {{ motd_content.stdout }}
....
```

`При необходимости прикрепитe сюда скриншоты
![Название скриншота 1](ссылка на скриншот 1)`


---

### Задание 2
Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору.
`Приведите ответ в свободной форме........`

---
- name: Изменение приветствия системы MOTD с системной информацией
  hosts: localhost
  connection: local
  become: yes
  vars:
    # Переменная для дополнительного сообщения
    admin_name: "системный администратор"
    custom_message: "Хорошего дня и продуктивной работы!"
  
  tasks:
    - name: Получить IP-адрес хоста
      command: hostname -I
      register: host_ip
      changed_when: false
    
    - name: Получить hostname
      command: hostname
      register: host_hostname
      changed_when: false
    
    - name: Получить дополнительную информацию о системе
      setup:
        filter: 
          - ansible_distribution
          - ansible_distribution_version
          - ansible_kernel
      register: system_info
    
    - name: Создать резервную копию оригинального MOTD
      copy:
        src: /etc/motd
        dest: "/etc/motd.backup_{{ ansible_date_time.date }}"
        remote_src: yes
        force: no
    
    - name: Создать новое приветствие MOTD
      copy:
        content: |
          ╔══════════════════════════════════════════════════════════╗
          ║                                                          ║
          ║  Добро пожаловать, {{ admin_name }}!
          ║  
          ║  Информация о сервере:
          ║  ───────────────────────────────────────────────────
          ║  Hostname: {{ host_hostname.stdout }}
          ║  IP-адрес: {{ host_ip.stdout }}
          ║  ОС: {{ system_info.ansible_facts.ansible_distribution }} {{ system_info.ansible_facts.ansible_distribution_version }}
          ║  Ядро: {{ system_info.ansible_facts.ansible_kernel }}
          ║  
          ║  {{ custom_message }}
          ║  
          ║  Дата входа: $(date +"%d.%m.%Y %H:%M:%S")
          ║  
          ╚══════════════════════════════════════════════════════════╝
          
          Для справки используйте команду: man
        dest: /etc/motd
        mode: '0644'
    
    - name: Создать динамический MOTD скрипт (для Ubuntu/Debian)
      copy:
        content: |
          #!/bin/bash
          
          # Получаем актуальные данные при каждом входе
          HOSTNAME=$(hostname)
          IP_ADDRESS=$(hostname -I | awk '{print $1}')
          KERNEL=$(uname -r)
          UPTIME=$(uptime -p)
          
          cat << EOF
          ╔══════════════════════════════════════════════════════════╗
          ║                                                          ║
          ║  Добро пожаловать, {{ admin_name }}!
          ║  
          ║  Информация о сервере:
          ║  ───────────────────────────────────────────────────
          ║  Hostname: ${HOSTNAME}
          ║  IP-адрес: ${IP_ADDRESS}
          ║  ОС: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)
          ║  Ядро: ${KERNEL}
          ║  Uptime: ${UPTIME}
          ║  
          ║  {{ custom_message }}
          ║  
          ║  Дата входа: $(date +"%d.%m.%Y %H:%M:%S")
          ║  Пользователь: ${USER}
          ║  
          ╚══════════════════════════════════════════════════════════╝
          EOF
        dest: /etc/update-motd.d/99-custom-message
        mode: '0755'
      when: ansible_os_family == "Debian"
    
    - name: Проверить содержимое нового MOTD
      command: cat /etc/motd
      register: motd_content
      changed_when: false
    
    - name: Показать новое приветствие
      debug:
        msg: |
          ==========================
          Новое приветствие MOTD:
          ==========================
          {{ motd_content.stdout }}
          ==========================
    
    - name: Показать системную информацию
      debug:
        msg:
          - "Hostname: {{ host_hostname.stdout }}"
          - "IP-адрес: {{ host_ip.stdout }}"
          - "ОС: {{ system_info.ansible_facts.ansible_distribution }}"
          - "Приветствие установлено для: {{ admin_name }}"


`При необходимости прикрепитe сюда скриншоты
![Название скриншота 2](ссылка на скриншот 2)`


---

### Задание 3


Выполните действия, приложите архив с ролью и вывод выполнения.

Ознакомьтесь со статьёй «Ansible - это вам не bash», сделайте соответствующие выводы и не используйте модули shell или command при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

Установить веб-сервер Apache на управляемые хосты.
Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес. Используйте Ansible facts и jinja2-template. Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
Сделать проверку доступности веб-сайта (ответ 200, модуль uri).
```


`При необходимости прикрепитe сюда скриншоты
![Название скриншота](ссылка на скриншот)`

 1. Создам структуру директорий
2. Файл inventory.ini:
ini
[webservers]
localhost ansible_connection=local
3. Роль: roles/webserver/tasks/main.yml:
yaml
---
# Основные задачи роли webserver

- name: Определить ОС и выбрать менеджер пакетов
  debug:
    msg: "ОС: {{ ansible_distribution }} {{ ansible_distribution_version }}"

- name: Установить веб-сервер (Debian/Ubuntu)
  block:
    - name: Обновить кэш apt
      apt:
        update_cache: yes
      changed_when: false
    
    - name: Установить Nginx
      apt:
        name: nginx
        state: present
    
    - name: Установить пакет для управления фаерволом (UFW)
      apt:
        name: ufw
        state: present
  when: ansible_os_family == "Debian"

- name: Установить веб-сервер (RedHat/CentOS/Rocky)
  block:
    - name: Установить EPEL репозиторий
      yum:
        name: epel-release
        state: present
    
    - name: Установить Nginx
      yum:
        name: nginx
        state: present
    
    - name: Установить firewalld
      yum:
        name: firewalld
        state: present
  when: ansible_os_family == "RedHat"

- name: Открыть порт 80 (UFW - Ubuntu/Debian)
  ufw:
    rule: allow
    port: '80'
    proto: tcp
  when: ansible_os_family == "Debian"
  notify: Перезапустить UFW

- name: Открыть порт 80 (firewalld - CentOS/RHEL)
  firewalld:
    port: 80/tcp
    permanent: yes
    state: enabled
    immediate: yes
  when: ansible_os_family == "RedHat"
  notify: Перезапустить firewalld

- name: Создать тестовую веб-страницу
  template:
    src: "{{ role_path }}/templates/index.html.j2"
    dest: /var/www/html/index.html
    mode: '0644'

- name: Запустить веб-сервер
  systemd:
    name: nginx
    state: started
    enabled: yes
    daemon_reload: yes

- name: Проверить доступность порта 80
  wait_for:
    port: 80
    state: started
    timeout: 30

- name: Проверить работу веб-сервера
  uri:
    url: http://localhost
    return_content: yes
  register: web_response

- name: Показать результат проверки
  debug:
    msg:
      - "HTTP статус: {{ web_response.status }}"
      - "Содержимое страницы: {{ web_response.content[:200] }}..."
4. Роль: roles/webserver/handlers/main.yml:
yaml
---
# Обработчики событий для роли webserver

- name: Перезапустить UFW
  service:
    name: ufw
    state: restarted
  when: ansible_os_family == "Debian"

- name: Перезапустить firewalld
  service:
    name: firewalld
    state: restarted
  when: ansible_os_family == "RedHat"

- name: Перезапустить Nginx
  service:
    name: nginx
    state: restarted
5. Роль: roles/webserver/vars/main.yml:
yaml
---
# Переменные для роли webserver
server_name: "Мой веб-сервер"
server_port: 80
server_admin: "admin@example.com"

# Сообщение для веб-страницы
welcome_message: "Веб-сервер успешно установлен с помощью Ansible!"
6  Плейбук playbook.yml:
yaml
---
- name: Установка и настройка веб-сервера через роль
  hosts: webservers
  become: yes
  vars:
    webserver_port: 80
    enable_https: false
  
  pre_tasks:
    - name: Проверка свободного порта 80
      wait_for:
        port: 80
        state: stopped
        timeout: 5
      ignore_errors: yes
      register: port_check
    
    - name: Предупреждение если порт занят
      debug:
        msg: "⚠️ Порт 80 уже используется! Возможен конфликт."
      when: port_check is succeeded
  
  roles:
    - role: webserver
  
  post_tasks:
    - name: Итоговая проверка веб-сервера
      uri:
        url: http://localhost:80
        status_code: 200
      register: final_check
    
    - name: Успешное завершение
      debug:
        msg:
          - "========================================"
          - "✅ Веб-сервер успешно установлен!"
          - "Порт 80 открыт"
          - "Сервер добавлен в автозагрузку"
          - "Статус: {{ final_check.status }}"
          - "Страница доступна по адресу: http://{{ ansible_default_ipv4.address }}"
          - "========================================
