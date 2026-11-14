# Домашнее задание №1. Изучение средств защиты ОС GNU/Linux

Виртуальная машина с РЕД ОС 8.0.3 (Standard Server Minimal), без графического окружения, ядро 6.12, имя хоста mephi-security.domain.local. Все настройки сделаны из командной строки и сохраняются после перезагрузки. Перезагрузка выполнена (`systemctl reboot`, строка 55 в history.out), все проверки и итоговые файлы сняты после неё.

Уникальный номер студента – 234.

## Как выполнен каждый пункт

| Раздел | Что сделано | Подтверждение |
|---|---|---|
| 1. Пользователь | `groupadd students`, `useradd -u 1234 -G students -m -s /bin/bash user1`, пароль задан `passwd user1`. Смена пароля каждые 3 месяца – `chage -M 90 user1` (поле максимального срока в /etc/shadow = 90) | passwd, group, shadow, stat.out, history.out |
| 2.1 Поиск файлов с set-UID | `find / -type f -perm -4000 -exec ls -l {} \; 2>/dev/null > find_suid.out`. Найдено 19 системных файлов (passwd, su, sudo, mount и др.) и копия cat из раздела 3 | find_suid.out |
| 2.2 Процессы с EUID 0 и реальным UID обычного пользователя | `ps -eo pid,ppid,ruid,ruser:12,euid,euser:12,tty,args \| awk 'NR==1 \|\| ($5==0 && $3>=1000 && $3<65534)'`. Во втором терминале user1 запустил `passwd` (ждёт ввода текущего пароля), в первом выполнен поиск. Найден процесс passwd с RUID 1234 (user1) и EUID 0 (root), а также sudo администратора | ps_euid0.out, user1_history.out |
| 3. set-UID | Утилита cat, операция – чтение /etc/shadow (права `----------`, доступно только root). Без set-UID: `cat: /etc/shadow: Permission denied`. Копия `cp /usr/bin/cat /home/user1/cat`, владелец root, `chmod 4755` (`-rwsr-xr-x`). От имени user1 `~/cat /etc/shadow` выводит файл | stat.out, history.out |
| 4. Привилегии (capabilities) | Утилита chown, операция – смена владельца файла на другого пользователя. Обычный chown от user1: `Operation not permitted`. Копия `cp /usr/bin/chown /home/user1/chown`, права 0755 без set-UID, `setcap cap_chown+ep /home/user1/chown`. От имени user1 `~/chown root:root ~/cap_test.txt` выполняется успешно | getcap.out, stat.out, history.out |
| 5. sudo | В /etc/sudoers через visudo добавлена строка `user1 ALL=(root) /usr/bin/date, /usr/bin/timedatectl`, проверка `visudo -c` и `sudo -l -U user1`. Без sudo `date -s` даёт `Operation not permitted`. В сеансе `su - user1` команда `sudo date -s "+1 hour"` меняет время, `sudo date -s "-1 hour"` возвращает, `sudo cat /etc/shadow` запрещено (`Sorry, user user1 is not allowed`). Потом время синхронизировано `chronyc makestep` | sudoers, history.out, user1_history.out |
| 6. Артефакты | Скриншот консоли с номером, history, stat, getcap, вывод поиска файлов и процессов | все файлы ниже |

Попытки в строках 44–47 history.out (`su - user1 -c 'sudo ...'`) не прошли, потому что у `su -c` нет терминала для ввода пароля sudo. Поэтому sudo проверен в интерактивном сеансе `su - user1` (строка 51). Команды этого сеанса записаны в истории user1 (user1_history.out).

## Файлы

| Файл | Раздел | Содержимое |
|---|---|---|
| mephi-screenshot.png | 6 | консоль ВМ, в терминале номер студента 234, user1, утилиты из разделов 3 и 4, правило sudo |
| history.out | все | история команд root (`history > ~/history.out`) |
| user1_history.out | 2, 5 | история команд user1 (passwd во втором терминале, проверка sudo) |
| stat.out | 1, 3, 4 | `stat /home/user1`, `stat /home/user1/cat`, `stat /home/user1/chown` |
| getcap.out | 4 | `getcap /home/user1/chown` |
| passwd | 1 | /etc/passwd |
| shadow | 1 | /etc/shadow |
| group | 1 | /etc/group |
| sudoers | 5 | /etc/sudoers |
| find_suid.out | 2 | вывод поиска файлов с set-UID |
| ps_euid0.out | 2 | вывод мониторинга процессов |
