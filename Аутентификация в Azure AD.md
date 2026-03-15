### Аутентификация в Azure AD из PowerShell с помощью сертификата

Azure AD поддерживает проверку подлинности клиентов на основе сертификатов (Certificate Based Authentication, CBA). Вы можете использовать аутентификацию с помощью сертификатов для автоматического запуска ваших PowerShell скриптов без ввода пароля пользователеля и MFA аутентификации Azure. Благодаря сертификатам, вам не нужно заботиться о безопасном хранении паролей в PowerShell скриптах или использовании сторонних хранилищ (vault) через модуль SecretManagement.


В этом статье мы покажем, как использовать аутентификацию в Azure AD и Exchange Online для ваших PowerShell скриптов с помощью сертификатов.

Сначала нужно создать сертификат на вашем компьютере. Вы можете получить сертификат у вашего CA или выпустить самоподписанный сертификат. В нашем примере мы создадим самоподписанный сертификат в Windows с помощью командлета New-SelfSignedCertificate. Чтобы сгенерировать самоподписанный сертфикат со сроком действия действия три года, выполните:

$certvalid = (Get-Date).AddYears(3)
$newcert = New-SelfSignedCertificate -DnsName "test.winitpro.ru" -CertStoreLocation "cert:\LocalMachine\My" -NotAfter $certvalid -KeySpec KeyExchange -FriendlyName "Azure AD PowerShell Auth cert"
$newcert|fl Subject,Thumbprint,NotBefore,NotAfter

Скопируйте значение отпечатка сертификата ( Thumbprint ).

создать самоподписанный сертфикат в Windows

Теперь экспортируйте сертификат в CER и PFX файлы:

$newcert | Export-Certificate -FilePath "C:\PS\azuread-auth.cer"
$newcert | Export-PfxCertificate -FilePath "C:\PS\azuread-auth.pfx" -Password $(ConvertTo-SecureString -String "MyPassW@rd321" -AsPlainText -Force)

Теперь нужно создать приложение в Azure AD

Откройте Azure Portal и выберите в Azure Active Directory -> App registrations -> New registration;
Укажите имя приложения (testCertAuthPowerShell), выберите Accounts in this organizational directory only (Single tenant) и нажмите Register;
Скопируйте значение Application (client ID);создать приложение в azure ad
Теперь нужно назначить права для вашего приложения. Перейдите в раздел API permissions -> Add a permission. В нашем примере мы предоставим права на управление конфигурацией Exchange Online (APIs my organization uses -> Office 365 Exchange Online) Azure -> Application permissions -> Exchange.ManageAsApp -> Add permissions -> Grant admin consent);назначить права для приложения azure ad
Перейдите в раздел Certificates and secrets и загрузите файл с самоподписанным сертификатом с вашего компьютера;импорт самоподписанного сертфиката в Azure
Теперь нужно назначить вашему приложению роль Exchange Administrator. Перейдите в Azure Active Directory -> Roles and administrators -> выберите роль Exchange Administrator -> Add Assignments -> выберите из списка ваше приложение (testCertAuthPowerShell).права администратора для приложения Azure
Теперь вы можете попробовать подключиться к Exchange Online из PowerShell скрипта на вашем компьютере с помощью сертификата. В данном примере на компьютере уже установлен модуль Exchange Online PowerShell.

Вам нужны значения следующих переменных (который вы скопировали ранее):

$certThumbprint = "7E82C3A560737C7894562593926E495777ECDB75"
$AzureAppID = "123456-1234-1234-1234-123456789"
$tenant="winitpro.onmicrosoft.com"

Выполните следующую команду, чтобы подключиться к тенанту Exchnage Online с помощью сертификата:

Connect-ExchangeOnline -AppId $AzureAppID -CertificateThumbprint $certThumbprint -Organization $tenant

Connect-ExchangeOnline - подключение к exchnage online из powershell с помощью сертфиката

Как вы видите, при подключении не запрашивается пароль и не появляется окно Modern аутентификации.

Выведите список почтовых ящиков в вашем тенанте:

Get-EXOMailbox

В конце PowerShell скрипта не забывайте добавить команду отключения сессии с Exchange Online:
Disconnect-ExchangeOnline -Confirm:$false

По аналогии вы можете настроить аутентификацию по сертификатам и для других модулей Microsoft 365. Главное правильно назначить разрешения для вашего приложения Azure.

Например, если вы используете модуль Teams PowerShell (4.7.1 и выше), то для аутентификации с помощью сертификата нужно использовать команду:

Connect-MicrosoftTeams -CertificateThumbprint $certThumbprint -ApplicationId $AzureAppID -TenantId $tenant

Connect-MicrosoftTeams -CertificateThumbprint 

Для подключения к Azure AD из PowerShell (версия модуля Azure AD 2.0.033 и выше) с помощью сертификата, используйте команду:

Connect-AzureAD -TenantId $tenant -ApplicationId $AzureAppID -CertificateThumbprint $certThumbprint

Подробнее о создании приложений в Azure, назначении прав и использованию Microsoft Graph API в PowerShell описано здесь:
https://winitpro.ru/index.php/2021/06/02/dostup-k-azure-microsoft-graph-api-powershell/

Вы можете импортировать сертификат на другой хост Windows с помощью PFX файла (содержим закрытый ключ, который вы защитили паролем). Можно импортировать сертификат из файла с помощью MMC консоли Certificates или из PowerShell:

$password = ConvertTo-SecureString -String "MyPassW@rd321" -Force -AsPlainText
Import-PfxCertificate -Password $password -FilePath "C:\PS\azuread-auth.pfx" -CertStoreLocation Cert:\CurrentUser\My

После этого вы можете использовать аутентификацию в Azure с помощью сертификата для PowerShell скриптов и на этом компьютере.
