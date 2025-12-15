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

<a id="two"></a>
## 🧰 Шаг 2 - Ansible
Файл `playbook.yml`. Роль простая, все задачи в одном файле:
1. **Установка ПО:** ставит `nginx` через apt.
2. **Создание папок:** `/var/www/site1`, `site2`, `site3`.
3. **Копирование файлов:** создает уникальные `index.html` для каждого сайта.
4. **Настройка Nginx:** удаляет дефолтный конфиг, заливает новый с тремя `server {}` блоками.
5. **Рестарт:** перезапускает Nginx при изменении конфигов.

<a id="three"></a>
## 🧰 Шаг 3 - Проверка
Выполнено на хост-системе (Linux Mint).
Запросы уходят на `localhost`, Vagrant перекидывает их в VM, Nginx отдает нужный сайт.

```bash
curl http://localhost:8081  # Site 1
curl http://localhost:8082  # Site 2
curl http://localhost:8083  # Site 3
```
