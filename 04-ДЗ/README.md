**Домашнее задание 4.**

**1.** Создаем VM.

**2.** Устанавливаем PostgresSQL и заходим в него.  
*sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-14
sudo -u postgres psql*

**3.** Создаем БД testdb.  
*CREATE DATABASE testdb*
