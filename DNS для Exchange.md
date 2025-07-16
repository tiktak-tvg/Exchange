#### Настраиваем внешний DNS для Exchange.
После установки Exchange Server следующим шагом будет настройка внешнего DNS для Exchange. <br>Важно сохранить одинаковое внутреннее и внешнее пространство имён.<br> 
Мы войдем в публичный DNS и создадим записи для Exchange Server.

Мы добавим в публичный DNS три типа записей. Что касается значений, вам нужно ввести только IP-адрес для записи A.<br>  
Запись MX отвечает за фильтрацию спама, а CNAME (псевдоним) должен указывать на DNS-имя записи A для сервера Exchange.<br>  
Если ваш IP-адрес изменился из-за смены интернет-провайдера, вам нужно будет обновить только запись A.<br> 

<img width="753" height="267" alt="image" src="https://github.com/user-attachments/assets/28888ee8-0ee5-4bff-8358-4f3287faf2f4" />

<img width="669" height="861" alt="image" src="https://github.com/user-attachments/assets/966f0a79-50cc-49bd-a1cc-ed9e7d9013e3" />
