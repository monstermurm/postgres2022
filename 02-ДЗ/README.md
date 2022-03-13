**Домашнее задание 2.**

**1)** Создаем инстанс в GCE с Ubuntu 20.04.

**2)** Устанавливаем Docker Engine [scrin](https://github.com/monstermurm/postgres2022/blob/main/02-%D0%94%D0%97/InstallDocker.jpg)

**3)** Создаем контейнер с PostgreSQL 14 и монтируем его /var/lib/postgres [scrin](https://github.com/monstermurm/postgres2022/blob/main/02-%D0%94%D0%97/DockerPostgress.jpg)

**4)** Создаем контейнер с клиентом [scrin](https://github.com/monstermurm/postgres2022/blob/main/02-%D0%94%D0%97/DockerClient.jpg)

**5)** Создаем базу данных, таблицу и несколько тестовых строк [scrin](https://github.com/monstermurm/postgres2022/blob/main/02-%D0%94%D0%97/CreateTable.jpg)
*(тут немного затупил, т.к. постгрес принимал команду select from test; и ничего не выводил, а я как-то привык, что если не задал выводимые столбцы, то будет ошибка синтаксиса)*

**6)** Подключаемся с внешнего ПК. Для этого сначала зададим правило в firewall [scrin](https://github.com/monstermurm/postgres2022/blob/main/02-%D0%94%D0%97/FireWall.jpg). После чего подключаемся к серверу postgres с внешнего ПК [scrin](https://github.com/monstermurm/postgres2022/blob/main/02-%D0%94%D0%97/Connect.jpg)
*(Под рукой не оказалось консоли, но удачно подвернулся сервер под виндой, откуда тоже можно было проверить доступ)*

**7)** Останавливаем контейнер [scrin](https://github.com/monstermurm/postgres2022/blob/main/02-%D0%94%D0%97/StopDocker.jpg) и удаляем контейнер с сервером, после создаем его занова и проверяем, что данные на месте [scrin](https://github.com/monstermurm/postgres2022/blob/main/02-%D0%94%D0%97/DeleteDocker.jpg)
*(в текстовом файле не было команды удаления докера, пришлось поискать самому)*
