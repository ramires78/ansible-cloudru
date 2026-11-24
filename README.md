
# Развёртывание Java-приложения с помощью Ansible в Cloud.ru

## Задача

Нужно с помощью Ansible создать две виртуальные машины:
сборочную и продовую.

На сборочном сервере должен собираться Java-проект.
Готовый артефакт необходимо передать на продовый сервер
и запустить на нём приложение.

По условию задания нужно использовать AWS EC2,
но я использовал Cloud.ru.

## Что использовал

Для выполнения задания использовал три виртуальные машины
с Ubuntu:

- ansible_control — управляющий сервер с Ansible.
- ansible-build — сервер для сборки Java-проекта.
- ansible-prod — сервер для запуска приложения.

Управляющий сервер создал заранее.

Сборочный и продовый серверы создаются автоматически
с помощью Ansible.

Для выполнения задания использовал:

- Ansible — автоматизация настройки серверов.
- Git — получение исходного кода.
- Maven — сборка Java-проекта.
- Java 17 — сборка и запуск приложения.
- Tomcat 9 — запуск Java-приложения.
- MySQL — база данных приложения.

Исходный код Java-проекта:

https://github.com/ramires78/app42-java-mysql-docker-compose

## Как всё работает

### 1. Создание серверов

Сначала запускается create_vms.yml.

Ansible:

1. Подключается к Cloud.ru через API.
2. Проверяет, существуют ли нужные серверы.
3. Создаёт отсутствующие виртуальные машины.
4. Дожидается их запуска.
5. Получает IP-адреса серверов через API.
6. Автоматически создаёт inventory.ini.

Если серверы уже существуют, повторно они не создаются.

IP-адреса вручную указывать не нужно.

### 2. Сборка Java-приложения

После создания серверов запускается deploy.yml.

Ansible подключается к ansible-build и:

1. Устанавливает Java, Maven и Git.
2. Скачивает Java-проект из GitHub.
3. Запускает сборку через Maven.
4. Получает готовый WAR-файл.

WAR — архив Java-веб-приложения,
который можно развернуть в Tomcat.

### 3. Развёртывание приложения

После успешной сборки Ansible:

1. Копирует WAR-файл на управляющий сервер.
2. Передаёт его на ansible-prod.
3. Устанавливает Java и MySQL.
4. Создаёт базу данных и пользователя MySQL.
5. Настраивает подключение приложения к базе данных.
6. Устанавливает Tomcat.
7. Размещает WAR-файл в Tomcat.
8. Запускает Tomcat и включает его автозапуск.

## Файлы проекта

- create_vms.yml — создание серверов и получение IP.
- deploy.yml — сборка и развёртывание приложения.
- README.md — описание проекта.
- .gitignore — исключение служебных файлов из Git.

Файл inventory.ini создаётся автоматически
после запуска create_vms.yml.

В GitHub он не хранится.

Пароли и приватные SSH-ключи также не хранятся
в репозитории.

## Подготовка

Ansible запускается с управляющего сервера.

На нём должны быть:

- Установленный Ansible.
- Сервисный аккаунт Cloud.ru с необходимыми правами.
- Файл с ключами доступа Cloud.ru.
- SSH-ключ для подключения к виртуальным машинам.

Ключи Cloud.ru находятся в:

    /home/ramires/.config/cloudru/credentials

SSH-ключ находится в:

    /home/ramires/.ssh/id_ed25519_ansible

Приватные ключи и пароли не публикуются в GitHub.

В текущей конфигурации проекта используются пути
и параметры моего управляющего сервера.

## Запуск

### 1. Создать виртуальные машины

    ansible-playbook -i localhost, create_vms.yml

После выполнения Ansible автоматически создаёт
inventory.ini с актуальными IP-адресами.

### 2. Проверить подключение

    ansible all -i inventory.ini -m ping

При успешном подключении оба сервера отвечают:

    ansible-build | SUCCESS
    ping: pong

    ansible-prod | SUCCESS
    ping: pong

### 3. Собрать и развернуть приложение

    ansible-playbook -i inventory.ini deploy.yml

Ansible соберёт Java-проект на сборочном сервере
и развернёт его на продовом.

## SSH после пересоздания серверов

При удалении виртуальной машины и создании новой
её SSH-ключ меняется.

Если Cloud.ru повторно назначит старый IP-адрес,
при подключении может появиться предупреждение:

    REMOTE HOST IDENTIFICATION HAS CHANGED

Это связано с тем, что SSH сохранил ключ
предыдущего сервера.

Сначала нужно убедиться, что IP принадлежит
новой виртуальной машине, и проверить её SSH-ключ.

После проверки старую запись можно удалить:

    ssh-keygen -R IP_СЕРВЕРА

Затем подключиться повторно и подтвердить новый ключ.

После этого проверить Ansible:

    ansible all -i inventory.ini -m ping

## Проверка работы

Проверил следующие этапы:

- Виртуальные машины создаются через Cloud.ru API.
- Повторный запуск не создаёт дубликаты.
- IP-адреса получаются автоматически.
- inventory.ini создаётся автоматически.
- Ansible подключается к обоим серверам.
- Maven успешно собирает Java-проект.
- WAR-файл передаётся на продовый сервер.
- Tomcat запускается через systemd.
- Приложение отвечает HTTP 200 при проверке на сервере.
- MySQL сохраняет тестовые данные.

После повторного создания виртуальных машин
отдельно проверил подключение Ansible к обоим серверам.

Проверка доступности приложения из внешнего интернета
не входит в подтверждённые результаты.

## Результат

С помощью Ansible автоматизировал создание двух
виртуальных машин в Cloud.ru, сборку Java-проекта
и его развёртывание на продовом сервере.

IP-адреса серверов получаются автоматически.

Для повторного развёртывания приложения достаточно
запустить deploy.yml.

---

# Deploying a Java Application with Ansible in Cloud.ru

## Task

The task was to create two virtual machines using Ansible:
one for building a Java application and another for running it.

The original task required AWS EC2,
but I used Cloud.ru instead.

The Java project must be built on the build server.
The artifact must be transferred to the production server
and started there.

## What I Used

I used three Ubuntu servers:

- ansible_control — the control server with Ansible.
- ansible-build — the server for building the application.
- ansible-prod — the server for running the application.

I created the control server manually.

The build and production servers are created
automatically using Ansible.

I used the following tools:

- Ansible — server automation.
- Git — downloading the source code.
- Maven — building the Java project.
- Java 17 — building and running the application.
- Tomcat 9 — running the Java web application.
- MySQL — storing application data.

Java project:

https://github.com/ramires78/app42-java-mysql-docker-compose

## How It Works

### 1. Creating the Servers

First, I run create_vms.yml.

Ansible:

1. Connects to Cloud.ru using its API.
2. Checks if the servers already exist.
3. Creates missing virtual machines.
4. Waits until both servers are running.
5. Gets their IP addresses from the API.
6. Automatically creates inventory.ini.

If the servers already exist,
Ansible does not create them again.

There is no need to enter IP addresses manually.

### 2. Building the Java Application

Next, I run deploy.yml.

Ansible connects to ansible-build and:

1. Installs Java, Maven and Git.
2. Downloads the Java project from GitHub.
3. Builds the application using Maven.
4. Gets the WAR file.

A WAR file is an archive of a Java web application
that can be deployed to Tomcat.

### 3. Deploying the Application

After a successful build, Ansible:

1. Copies the WAR file to the control server.
2. Transfers it to ansible-prod.
3. Installs Java and MySQL.
4. Creates the MySQL database and user.
5. Configures the database connection.
6. Installs Tomcat.
7. Deploys the WAR file to Tomcat.
8. Starts Tomcat and enables automatic startup.

## Project Files

- create_vms.yml — creates servers and gets their IP addresses.
- deploy.yml — builds and deploys the application.
- README.md — project description.
- .gitignore — excludes temporary and private files.

The inventory.ini file is created automatically.

It is not stored in GitHub.

Passwords and private SSH keys are also not stored
in the repository.

## Requirements

The control server needs:

- Ansible.
- A Cloud.ru service account with the required permissions.
- Cloud.ru API credentials.
- An SSH key for connecting to virtual machines.

Cloud.ru credentials:

    /home/ramires/.config/cloudru/credentials

SSH key:

    /home/ramires/.ssh/id_ed25519_ansible

Private keys and passwords are not published in GitHub.

The current configuration uses paths and settings
from my control server.

## How to Run

### 1. Create the Virtual Machines

    ansible-playbook -i localhost, create_vms.yml

Ansible automatically creates inventory.ini
with the current server IP addresses.

### 2. Check SSH Connections

    ansible all -i inventory.ini -m ping

Both servers should return:

    ping: pong

### 3. Build and Deploy the Application

    ansible-playbook -i inventory.ini deploy.yml

Ansible builds the project on the build server
and deploys it to the production server.

## SSH After Recreating Servers

When a virtual machine is deleted and created again,
its SSH host key changes.

If Cloud.ru assigns the same IP address,
SSH may display this warning:

    REMOTE HOST IDENTIFICATION HAS CHANGED

This happens because SSH remembers
the key of the previous server.

First, check that the IP address belongs
to the new virtual machine and verify its SSH key.

After verification, remove the old entry:

    ssh-keygen -R SERVER_IP

Then connect again and confirm the new key.

Finally, check Ansible:

    ansible all -i inventory.ini -m ping

## Testing

I checked the following:

- Virtual machines are created using the Cloud.ru API.
- Running the playbook again does not create duplicate servers.
- Server IP addresses are received automatically.
- inventory.ini is created automatically.
- Ansible connects to both servers.
- Maven builds the Java project successfully.
- The WAR file is transferred to the production server.
- Tomcat runs as a systemd service.
- The application returns HTTP 200 when tested on the server.
- MySQL saves test data successfully.

After recreating the virtual machines,
I also checked SSH access to both servers.

External access to the application has not been
confirmed as part of testing.

## Result

I automated the creation of two virtual machines
in Cloud.ru, the Java build process
and the deployment to the production server.

Server IP addresses are obtained automatically.

To deploy the application again,
I only need to run deploy.yml.
