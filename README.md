# Развёртывание Java-приложения с помощью Ansible в Cloud.ru

## Задача

Нужно с помощью Ansible создать два сервера:
один для сборки Java-приложения, второй для его запуска.

В задании указан AWS, но я использовал Cloud.ru.

## Что сделал

Создал три виртуальные машины на Ubuntu:

- ansible_control — управляющий сервер с Ansible.
- ansible-build — сервер для сборки приложения.
- ansible-prod — сервер для запуска приложения.

Управляющий сервер создал заранее, а сборочный
и продовый серверы — через Ansible.

## Как всё работает

1. Ansible подключается к сборочному серверу.
2. Устанавливает Java, Maven и Git.
3. Скачивает Java-проект из GitHub.
4. Maven собирает приложение в WAR-файл.
5. Ansible забирает готовый WAR-файл на управляющий сервер.
6. Передаёт его на продовый сервер.
7. На продовом сервере устанавливаются Java, MySQL и Tomcat.
8. Приложение запускается через Tomcat.

Для доступа к приложению также настроил Nginx.

## Какие файлы использовал

- create_vms.yml — создание двух виртуальных машин в Cloud.ru.
- inventory.ini — адреса серверов для подключения.
- deploy.yml — сборка и развёртывание Java-приложения.
- nginx.yml — настройка Nginx.

## Запуск

Создание виртуальных машин:

    ansible-playbook -i localhost, create_vms.yml

Проверка подключения к серверам:

    ansible all -i inventory.ini -m ping

Сборка и развёртывание приложения:

    ansible-playbook -i inventory.ini deploy.yml

Настройка Nginx:

    ansible-playbook -i inventory.ini nginx.yml

## Проверка

После выполнения playbook проверил:

- Подключение Ansible к обоим серверам работает.
- Java-проект успешно собирается через Maven.
- WAR-файл передаётся на продовый сервер.
- Tomcat запущен и добавлен в автозагрузку.
- Приложение возвращает HTTP 200.
- MySQL работает, тестовая запись сохраняется в базу.

Проверку приложения проводил непосредственно на сервере.

## Результат

В итоге получилось автоматизировать сборку
и развёртывание Java-приложения на двух серверах.

Если изменить проект в GitHub и повторно запустить
deploy.yml, Ansible заново соберёт приложение
и передаст обновлённый WAR-файл на продовый сервер.

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
The build and production servers were created using Ansible.

## How It Works

1. Ansible connects to the build server.
2. It installs Java, Maven and Git.
3. It downloads the Java project from GitHub.
4. Maven builds the application and creates a WAR file.
5. Ansible copies the WAR file to the control server.
6. Then it transfers the WAR file to the production server.
7. Java, MySQL and Tomcat are installed on the production server.
8. Tomcat starts the Java application.

I also configured Nginx to provide access to the application.

## Project Files

- create_vms.yml — creates two virtual machines in Cloud.ru.
- inventory.ini — contains the server addresses.
- deploy.yml — builds and deploys the Java application.
- nginx.yml — configures Nginx.

## How to Run

Create the virtual machines:

    ansible-playbook -i localhost, create_vms.yml

Check the connection to both servers:

    ansible all -i inventory.ini -m ping

Build and deploy the application:

    ansible-playbook -i inventory.ini deploy.yml

Configure Nginx:

    ansible-playbook -i inventory.ini nginx.yml

## Testing

After running the playbooks, I checked the following:

- Ansible can connect to both servers.
- Maven builds the Java project successfully.
- The WAR file is transferred to the production server.
- Tomcat is running and starts automatically.
- The application returns HTTP 200.
- MySQL works and saves test data successfully.

I tested the application directly on the server.

## Result

I automated the process of building and deploying
a Java application on two separate servers.

If the project is updated on GitHub, I can run
deploy.yml again to build a new WAR file
and deploy the updated application to the production server.
