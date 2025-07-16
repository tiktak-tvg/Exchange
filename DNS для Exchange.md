#### Настраиваем внешний DNS для Exchange.
После установки Exchange Server следующим шагом будет настройка внешнего DNS для Exchange. <br>Важно сохранить одинаковое внутреннее и внешнее пространство имён.<br> 
Мы войдем в публичный DNS и создадим записи для Exchange Server.

Мы добавим в публичный DNS три типа записей. Что касается значений, вам нужно ввести только IP-адрес для записи A.<br>  
Запись MX отвечает за фильтрацию спама, а CNAME (псевдоним) должен указывать на DNS-имя записи A для сервера Exchange.<br>  
Если ваш IP-адрес изменился из-за смены интернет-провайдера, вам нужно будет обновить только запись A.<br> 

<img width="753" height="267" alt="image" src="https://github.com/user-attachments/assets/28888ee8-0ee5-4bff-8358-4f3287faf2f4" />

Архитектурное представление пространства имен Exchange показано на схеме ниже.

<img width="669" height="861" alt="image" src="https://github.com/user-attachments/assets/966f0a79-50cc-49bd-a1cc-ed9e7d9013e3" />

##### Создать публичные записи DNS.
Войдите в систему внешнего DNS-регистратора. Настройте записи.

<img width="754" height="296" alt="image" src="https://github.com/user-attachments/assets/489f991f-297a-470d-be70-25142033fd02" />

##### Проверьте записи публичного DNS.
Давайте запустим командлет Resolve-DnsName для проверки:

- MX-запись
- Запись
- Запись автообнаружения

Запустите `PowerShell` от имени администратора. Выполните командлет `Resolve-DnsName` с параметром `-Server`.<br> 
Параметр `-Server` разрешит имя на DNS-серверах Google.<br>  
Мы можем проверить публичные DNS-записи в локальной сети (LAN) или любой внешней сети.

##### Проверьте запись MX.

<img width="679" height="160" alt="image" src="https://github.com/user-attachments/assets/c5adb407-4e33-4f2d-919c-0ea5cea73c39" />

##### Проверить запись А.
Давайте посмотрим на публичную запись А.

<img width="678" height="113" alt="image" src="https://github.com/user-attachments/assets/d61ffe23-ae25-4e0c-a744-0c7b5d0fa278" />

##### Проверьте запись автообнаружения.
Напоследок давайте проверим публичную запись автообнаружения.

<img width="678" height="107" alt="image" src="https://github.com/user-attachments/assets/74b75f6d-2ff0-4d92-bdd5-db4417ae2ef4" />


