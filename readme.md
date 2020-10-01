Задание:
1. Запретить всем пользователям, кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников
* дать конкретному пользователю права работать с докером
и возможность рестартить докер сервис

Решение:

Создадим группы:
Пользователи - UUSERS
Администратор - ADMINS

$ sudo groupadd uusers
$ sudo groupadd admins

Проверяем созданные группы:

$ cat /etc/group | grep -P 'uusers|admins'

Добавляем тестовых пользователей в группы:

$ sudo useradd -g uusers usman
$ sudo useradd -g uusers usman2
$ sudo useradd -g admins admiman

Проверяем пользователей

$ cat /etc/passwd | grep -P 'usman|usman2|admiman'

Теперь добавляем в файл /etc/security/time.conf
правило запрета доступа пользователей из группы uusers к системе на выходные, кроме пользователя из группы admins 
(https://www.opennet.ru/base/dev/pam_linux.txt.html
https://www.opennet.ru/man.shtml?topic=time.conf&category=5&russian=2#lbAD)

$ echo '*;*;usman|usman2|!admiman|;Wk0000-2400' | sudo tee --append /etc/security/time.conf

Добавялем правило в /etc/pam.d/sshd

$ echo 'account required pam_time.so' | sudo tee --append /etc/pam.d/sshd

Теперь в выходные дни, доступ пользователям из группы uusers в систему запрещен.


2. Возможность реатртовать конкретному пользователю контейнеры docker не от sudo

Есть следующие варианты:
Если пользователь один 
Выставить на директорию suid-бит +s  (https://ru.wikipedia.org/wiki/Chmod)

$ sudo chmod ug+s /usr/bin/docker

После чего можно полноценно управлять контейнерами без опции суперпользователя

Если пользователей несколько, а доступ нужно организовать одному, то включаем этого пользователя в группу docker

Команда добавления в группу docker текущего пользователя
$ sudo usermod -aG docker ${USER}

Пример с другим пользователем

$ sudo usermod -aG docker admiman

Проверяем, что он добавлен в группу

$ id -Gn admiman

Авторизуемся пользователем admiman через $ su admiman   
и проверяем возможность запуска контейнера docker
