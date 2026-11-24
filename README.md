# Развёртывание Java-приложения с помощью Ansible в Cloud.ru

## Задача

Нужно с помощью Ansible создать два сервера:
один для сборки Java-приложения, второй для его запуска.

В задании указан AWS, но я использовал Cloud.ru.

## Что сделал

Для выполнения задания использовал три сервера на Ubuntu:

- ansible_control — управляющий сервер с установленным Ansible.
- ansible-build — сервер для сборки Java-приложения.
- ansible-prod — сервер для запуска приложения.

Управляющий сервер создал заранее.

Два остальных сервера создаются автоматически с помощью Ansible.

## Как всё работает

1. Ansible подключается к Cloud.ru через API.
2. Проверяет, существуют ли сборочный и продовый серверы.
3. Если серверов нет, создаёт их.
4. Дожидается запуска серверов.
5. Получает их IP-адреса через Cloud.ru API.
6. Автоматически создаёт файл inventory.ini.
7. Подключается к сборочному серверу.
8. Устанавливает Java, Maven и Git.
9. Скачивает Java-проект из GitHub.
10. Maven собирает приложение в WAR-файл.
11. Ansible забирает WAR на управляющий сервер.
12. Передаёт WAR на продовый сервер.
13. Устанавливает Java, MySQL и Tomcat.
14. Настраивает базу данных и запускает приложение.

Дополнительно настроил Nginx для доступа к приложению по HTTP.

## Файлы проекта

- create_vms.yml — создание серверов в Cloud.ru и получение IP.
- deploy.yml — сборка и развёртывание Java-приложения.
- nginx.yml — установка и настройка Nginx.
- README.md — описание проекта.
- .gitignore — исключение паролей и временных файлов из Git.

Файл inventory.ini создаётся автоматически после запуска
create_vms.yml и не хранится в GitHub.

## Запуск

Сначала запускаем создание серверов:

    ansible-playbook -i localhost, create_vms.yml

После этого Ansible автоматически создаёт inventory.ini
с актуальными IP-адресами серверов.

Проверяем подключение:

    ansible all -i inventory.ini -m ping

Запускаем сборку и развёртывание приложения:

    ansible-playbook -i inventory.ini deploy.yml

Настраиваем Nginx:

    ansible-playbook -i inventory.ini nginx.yml

## Проверка

После запуска плейбуков проверил:

- Оба сервера создаются через Cloud.ru API.
- Повторный запуск не создаёт дубликаты серверов.
- IP-адреса автоматически записываются в inventory.ini.
- Ansible подключается к обоим серверам.
- Maven успешно собирает Java-проект.
- WAR-файл передаётся на продовый сервер.
- Tomcat запускается и добавляется в автозагрузку.
- Java-приложение возвращает HTTP 200.
- MySQL работает, тестовая запись сохраняется в базу данных.

Работу приложения проверял непосредственно на сервере.

## Результат

В итоге автоматизировал создание двух виртуальных машин,
сборку Java-проекта и его развёртывание на продовом сервере.

IP-адреса серверов вручную указывать не нужно.

При повторном запуске Ansible использует существующие серверы.

Если проект обновится в GitHub, можно повторно запустить
deploy.yml, чтобы собрать новую версию и развернуть её.

---

# Deploying a Java Application with Ansible in Cloud.ru

## Task

The task was to create two virtual machines using Ansible:
one for building a Java application and another for running it.

The original task required AWS, but I used Cloud.ru instead.

## What I Did

I used three Ubuntu servers:

- ansible_control — the control server with Ansible.
- ansible-build — the server for building the application.
- ansible-prod — the server for running the application.

I created the control server manually.

The other two servers are created automatically using Ansible.

## How It Works

1. Ansible connects to Cloud.ru using its API.
2. It checks if the build and production servers already exist.
3. If the servers do not exist, Ansible creates them.
4. It waits until both servers are running.
5. It gets their IP addresses from the Cloud.ru API.
6. It automatically creates the inventory.ini file.
7. Ansible connects to the build server.
8. It installs Java, Maven and Git.
9. It downloads the Java project from GitHub.
10. Maven builds the application and creates a WAR file.
11. Ansible copies the WAR file to the control server.
12. It transfers the WAR file to the production server.
13. It installs Java, MySQL and Tomcat.
14. It configures the database and starts the application.

I also configured Nginx for HTTP access to the application.

## Project Files

- create_vms.yml — creates the servers and gets their IP addresses.
- deploy.yml — builds and deploys the Java application.
- nginx.yml — installs and configures Nginx.
- README.md — project description.
- .gitignore — excludes passwords and temporary files from Git.

The inventory.ini file is created automatically
and is not stored in GitHub.

## How to Run

First, create the virtual machines:

    ansible-playbook -i localhost, create_vms.yml

Ansible automatically creates inventory.ini
with the current server IP addresses.

Check the connection to both servers:

    ansible all -i inventory.ini -m ping

Build and deploy the application:

    ansible-playbook -i inventory.ini deploy.yml

Configure Nginx:

    ansible-playbook -i inventory.ini nginx.yml

## Testing

After running the playbooks, I checked the following:

- Both servers are created using the Cloud.ru API.
- Running the playbook again does not create duplicate servers.
- Server IP addresses are added to inventory.ini automatically.
- Ansible can connect to both servers.
- Maven builds the Java project successfully.
- The WAR file is transferred to the production server.
- Tomcat is running and starts automatically.
- The Java application returns HTTP 200.
- MySQL works and saves test data successfully.

I tested the application directly on the server.

## Result

I automated the creation of two virtual machines,
the build process and the deployment of a Java application.

There is no need to enter server IP addresses manually.

When the playbook runs again, it uses the existing servers.

If the project is updated on GitHub, I can run deploy.yml
again to build and deploy the new version.
