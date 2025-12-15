<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/05/Ansible_Logo.png/480px-Ansible_Logo.png" alt="Ansible Logo" width="100">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c5/Nginx_logo.svg/500px-Nginx_logo.svg.png" alt="Nginx Logo" width="150">
</p>

## ![Lesson](https://img.shields.io/badge/Lesson-otus__web-0A84FF?style=for-the-badge&logo=linux&logoColor=white&labelColor=111827)![Author](https://img.shields.io/badge/Author-Kamil%20Ibragimov-10B981?style=for-the-badge&logo=github&logoColor=white&labelColor=111827)![Date](https://img.shields.io/badge/Date-15.12.2025-F59E0B?style=for-the-badge&logo=calendar&logoColor=white&labelColor=111827)

### 📌 Задание
Развернуть веб-стенд с использованием **Ansible**:
1. Поднять виртуальную машину через Vagrant.
2. Пробросить порты **8081, 8082, 8083** на хост.
3. Настроить Nginx через Ansible: три разных сайта на разных портах.

### ✅ Результат
- [x] Vagrant запускает Ansible автоматически.
- [x] Nginx установлен, конфиги и сайты созданы.
- [x] Сайты открываются с хостовой машины.
- [x] Проверка пройдена (см. скриншот):
  🖼️ [Лог запуска и проверка](web_ansible_proof.png)

### 🧭 Оглавление
- [🧰 Шаг 1 - Vagrant](#one)
- [🧰 Шаг 2 - Ansible](#two)
- [🧰 Шаг 3 - Проверка](#three)

---

<a id="one"></a>
## 🧰 Шаг 1 - Vagrant
Файл `Vagrantfile`.
* Использует образ **Ubuntu 22.04**.
* Пробрасывает порты 8081-8083 на localhost.
* Запускает `ansible_local` (устанавливает Ansible внутри VM и выполняет плейбук).

```bash
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "web-deploy"
  config.vm.network "private_network", ip: "192.168.56.20"

  # Проброс портов
  config.vm.network "forwarded_port", guest: 8081, host: 8081
  config.vm.network "forwarded_port", guest: 8082, host: 8082
  config.vm.network "forwarded_port", guest: 8083, host: 8083

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
  end

  # Запуск Ansible
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.install = true
  end
end
```

<a id="two"></a>
🧰 Шаг 2 - Ansible
Файл playbook.yml. Роль простая, все задачи в одном файле:
Установка ПО: ставит nginx через apt.
Создание папок: /var/www/site1, site2, site3.
Копирование файлов: создает уникальные index.html для каждого сайта.
Настройка Nginx: удаляет дефолтный конфиг, заливает новый с тремя server {} блоками.
Рестарт: перезапускает Nginx при изменении конфигов.
```bash
---
- name: Configure Web Server
  hosts: all
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Create web directories
      file:
        path: "/var/www/site{{ item }}"
        state: directory
        mode: '0755'
      loop: [1, 2, 3]

    - name: Create index.html for each site
      copy:
        dest: "/var/www/site{{ item }}/index.html"
        content: |
          <html>
          <head><title>Site {{ item }}</title></head>
          <body><h1>Hello from Site {{ item }} deployed via Ansible!</h1></body>
          </html>
      loop: [1, 2, 3]

    - name: Remove default Nginx config
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      notify: Restart Nginx

    - name: Configure Nginx Virtual Hosts
      copy:
        dest: /etc/nginx/conf.d/multisite.conf
        content: |
          server { listen 8081; root /var/www/site1; index index.html; }
          server { listen 8082; root /var/www/site2; index index.html; }
          server { listen 8083; root /var/www/site3; index index.html; }
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

<a id="three"></a>
## 🧰 Шаг 3 - Проверка
Выполнено на хост-системе (Linux Mint). Запросы уходят на localhost, Vagrant перекидывает их в VM, Nginx отдает нужный сайт.

```Bash
curl http://localhost:8081  # Site 1
curl http://localhost:8082  # Site 2
curl http://localhost:8083  # Site 3
```
