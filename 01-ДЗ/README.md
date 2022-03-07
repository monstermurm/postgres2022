**Домашнее задание 1.**

**1)** Создан проект в Google Cloud Platform по именем "postgres2022-190384". [scrin](https://github.com/monstermurm/postgres2022/blob/main/01-%D0%94%D0%97/Project.jpg)

**2)** Созадана VM. [scrin1](https://github.com/monstermurm/postgres2022/blob/main/01-%D0%94%D0%97/VM1.jpg), [scrin2](https://github.com/monstermurm/postgres2022/blob/main/01-%D0%94%D0%97/VM2.jpg)

**3)** Добавлен ключ GCE metadata. [scrin](https://github.com/monstermurm/postgres2022/blob/main/01-%D0%94%D0%97/ssh_keys.jpg)

**4)** Установлен Postgres 14.

**5)** Создаем таблицу и наполняем данными согласно ДЗ.

**6)** Запускаем 2 сессии с отключенным автокомитом и уровнем изоляции Read Committed.
Если в одной ссесии мы добавим строку, а во второй вызовем чтение таблицы, то во второй ссесии мы не увидим записи.
Это связано с тем, что Postgres не допускает грязного чтения. [scrin](https://github.com/monstermurm/postgres2022/blob/main/01-%D0%94%D0%97/committed1.jpg)
Если в первой ссесии зафиксировать транзацию и сделать во второй ссесии чтение - мы увидим запись, т.к. транзакция в первой закрыта и мы видим текущее состояние таблицы. [scrin](https://github.com/monstermurm/postgres2022/blob/main/01-%D0%94%D0%97/committed2.jpg)

**7)** Запускаем 2 ссессии, но устанавливаем уровень изоляции Repeatable Read.
Если в одной ссесии мы добавим строку, а во второй вызовем чтение таблицы, то во второй ссесии мы не увидим записи, по причине отсутвия грязного чтения.
После фиксации транзакции в сессии 1 и получения данных во второй сессии - мы опять не видим данные. [scrin](https://github.com/monstermurm/postgres2022/blob/main/01-%D0%94%D0%97/REPEATABLE1.jpg)
Если во второй сессии зафиксировать транзакцию, открыть новую и сделать опять чтение - мы увидим добавленные данные.
Причина этого, что вторая ссесия в режиме изоляции Repeatable Read не будет видеть результаты завершенных транзакций до окончания текущей транзакции, при ее завершении и отрытии новой транзакции - данные предыдущих транзакций нам станут доступны. [scrin](https://github.com/monstermurm/postgres2022/blob/main/01-%D0%94%D0%97/REPEATABLE2.jpg)

*P.S. почему-то в одной ссесии я мог переглючить уровень изоляции коммандой set transaction isolation level repeatable read, а во второй сессии мне на эту команду выдавало предупреждение "SET TRANSACTION can only be used in transaction blocks" и уровень изоляции не менялся, пришлось открывать транзакцию принудительно с указанием уровня изоляции "BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ", поэтому в моих примерах 2 строки со светланой.*
