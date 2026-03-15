### Не обновляется офлайн адресная книга (OAB) в Outlook

В некоторых случаях вы можете заметить, что на клиентах Outlook перестает обновляться глобальная адресная книга Exchange с ошибкой 
“Outlook cannot update Global Address Book 0x80200049 ” или 0x80070057 . Вы добавляете в Global Address List (GAL) новые почтовые ящики пользователей, 
группу рассылки, скрыли пользователя из адресной книги, а изменения не отображаются на компьютерах пользователей.


При внесении изменений в адресную книгу Exchange, Offline Address Book (OAB) обновляется раз в 8 часов, 
и в течении 24 часов Outlook пользователей должен загрузить обновленный файл (загружаются только изменения). 
Таким образом, максимальное время обновления адресной книги составляет 32 часа, прежде чем пользователь увидит изменения в адресной книге в Outlook.

В Windows для загрузки Offline Address Book (OAB) с сервера Exchange используется протокол BITS. 
Протокол BITS позволяет скачать файл адресной книги с сервера на компьютер пользователя (используется в том числе для получения файлов обновлений Windows). 
Если в очереди BITS более 50 заданий, задание загрузки адресной книги Outlook может завершаться с ошибкой 0x80200049 или 0x80070057.


ошибка загрузки адресной книги в outlook

Task Username@domain.com reported error (0x80070057): ‘Sorry, something went wrong. You may want to try again’
При этом в логе синхронизации Outlook могут быть такие ошибки:

Microsoft Exchange offline address book
Not downloading Offline address book files. A server (URL) could not be located
0X80004005
Или:

Source: Outlook
Event ID: 27
Description: The operation failed.
Проверьте, что служба BITS разрешена к запуску на клиенте (по умолчанию тип запуска Manual). Выполните следующую PowerShell команду чтобы получить состояние службы:

Get-Service BITS | select -Property Name, StartType, Status

служба BITS используется для загрузки офлайн адресной книни

Служба BITS должна быть настроена на ручной или автоматический запуск

Выведите список задания в очереди BITS:

bitsadmin /list

Если в очереди более 50 задания загрузки, тогда вы можете столкнуться с ошибками получения адресной книги в Outlook

Для решения проблемы нужно очистить очередь BITS на компьютере пользователя командой:

bitsadmin.exe /reset

Или от имени администратора:

bitsadmin.exe /reset /allusers

Если проблема не решена, попробуйте закрыть Outlook и очистить каталог с текущей версией адресной книги. Достаточно просто переименовать папку C:\Users\%username%\AppData\Local\Microsoft\Outlook\Offline Address Books\ в профиле пользователя.

Запустите Outlook и попробуйте вручную загрузить адресную книгу:

Нажмите кнопку Send/Receive в Outlook и выберите пункт Download Address Book;скачать offlne address book
Включите опцию Download changes since last Send/Receive, выберите в выпадающем списке \Offline Global Address List и нажмите OK.загрузка изменений в адресной книге exchange
Проверьте, что в Outlook включен режим кэширования Exchange (Account Settings -> Edit -> Use Cached Exchange Mode).

режим кэширования exchange в outlook

Если ничего не помогло, пересоздайте профиль Outlook (можно воспользоваться ярлыком Почта в Панели управления Windows). Если при настройке ящика появится ошибка: Outlook: Не удается сопоставить это имя ни одному из имен в списке адресов, следуйте этому гайду.

Если проблема с загрузкой адресной книги возникает у всех клиентов Outlook, проверьте на сервере Exchange что у пользователей достаточно NTFS полномочий для доступа к файлу C:\Program Files\Microsoft\Exchange Server\V13\ExchangeOAB (в Exchange 2010).

Проверьте, что на сервере создана хотя бы одна адресная книга:

Get-OfflineAddressBook | fl

Если при загрузке OAB появляется ошибка Task ‘username@domain.com’ reported error (0x80190194): The operation failed , проверьте что в Exchange Server вашем включены службы публикации Offline Address Book:

Get-OfflineAddressBook | fl Name,GlobalWebDistributionEnabled,VirtualDirectories

Разрешите публикацию OAB:

Get-OfflineAddressBook | Set-OfflineAddressBook -VirtualDirectories $null -GlobalWebDistributionEnabled $true

включить публикацию адресной книги в exchange

Обновите все адресные книги:

Get-OfflineAddressBook | Update-OfflineAddressBook
