# Домашнее задание к занятию «Защита сети» Комиссаров Игорь

### Подготовка к выполнению заданий

1. Подготовка защищаемой системы:

- установите **Suricata**,
- установите **Fail2Ban**.

2. Подготовка системы злоумышленника: установите **nmap** и **thc-hydra** либо скачайте и установите **Kali linux**.

Обе системы должны находится в одной подсети.

------

### Задание 1

Проведите разведку системы и определите, какие сетевые службы запущены на защищаемой системе:

**sudo nmap -sA < ip-адрес >**

**sudo nmap -sT < ip-адрес >**

**sudo nmap -sS < ip-адрес >**

**sudo nmap -sV < ip-адрес >**

По желанию можете поэкспериментировать с опциями: https://nmap.org/man/ru/man-briefoptions.html.


*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*

### Решение 1

Используя [Vagrant](./Vagrantfile) развернем 2 ВМ : 

- защищаемая система - Debian12 с необходимым ПО;
- систем злоумышленника - KaliLinux

Из лога Suricata можно получить  информацию о сканировании портов - время, ip адрес, порт, возможно NMAP сканирование и тип сканирования.

![1.png](./img/1.png)
![2.png](./img/2.png)
 
------

### Задание 2

Проведите атаку на подбор пароля для службы SSH:

**hydra -L users.txt -P pass.txt < ip-адрес > ssh**

1. Настройка **hydra**: 
 
 - создайте два файла: **users.txt** и **pass.txt**;
 - в каждой строчке первого файла должны быть имена пользователей, второго — пароли. В нашем случае это могут быть случайные строки, но ради эксперимента можете добавить имя и пароль существующего пользователя.

Дополнительная информация по **hydra**: https://kali.tools/?p=1847.

2. Включение защиты SSH для Fail2Ban:

-  открыть файл /etc/fail2ban/jail.conf,
-  найти секцию **ssh**,
-  установить **enabled**  в **true**.

Дополнительная информация по **Fail2Ban**:https://putty.org.ru/articles/fail2ban-ssh.html.



*В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.*

### Решение 2

Лог Fail2Ban показывает информацию о попытках входа по ssh и блокировке по ip адресу , в случае 6 неудавшихся попыток логина согласно правилу maxretry. 

![3.png](./img/3.png)
![4.png](./img/4.png)

Из лога  Suricata видим запросы на порт 22:

![7.png](./img/7.png)

Установим  maxretry на 0 . Видим, что правило работает, так как блокировка ip происходит сразу после 1-й неудавшейся попытки логина.

![5.png](./img/5.png)
![6.png](./img/6.png)
