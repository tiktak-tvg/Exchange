### PowerShell: Отправка писем через Microsoft Graph API

Для отправки писем через SMTP сервер из скриптов PowerShell можно использовать встроенный командлет Send-MailMessage.Основной его недостаток в том может использовать только Basic SMTP аутентификацию, и не поддерживает современные методы проверки подлинности, в том числе OAuth и Microsoft Modern Authentication. Если вы выполните команду Send-MailMessage в новых версиях PowerShell Core 7.x , появится предупреждение:


WARNING: The command 'Send-MailMessage' is obsolete. This cmdlet does not guarantee secure connections to SMTP servers. While there is no immediate replacement available in PowerShell, we recommend you do not use Send-MailMessage at this time. See https://aka.ms/SendMailMessage for more information.
командлет 'Send-MailMessage' устарел

Microsoft по умолчанию отключает basic auth для всех новых тенантов Azure. В этой статье, мы рассмотрим, как отправлять письма через ваш тенант Exchange Online/Microsoft 365 с помощью Microsoft Graph API.

Содержание:
Разрешить отправку email через Microsoft Graph API
Отправить письмо из Exchange Online с помощью Invoke-RestMethod
Отправить письмо с помощью командлета Send-MgUserMail из модуля Microsoft.Graph
Разрешить отправку email через Microsoft Graph API
Для получения токена в Azure и аутентификации из вашего скрипта PowerShell, нужно создать в тенанте новое приложение (Azure Active Directory -> App registration -> New registration). Затем предоставить приложению разрешение Mail.Send (Api Permissions -> Add a permissions -> Microsoft Graph -> Application permissions -> Mail.Send).

Права Mail.Send в Azure

Подробнее о регистрации приложения Azure для Microsoft Graph в статье Доступ к Azure через Microsoft Graph API с помощью PowerShell.
По умолчанию ваше приложение может отправить письмо от имени любого почтового ящика в тенанте Exchange Online. Вы можете ограничить список адресов, от имени которых может отправлять почту ваше приложение с помощью Application Access Policy.

Подключитесь к Exchange Onlineс помощью PowerShell ( Connect-ExchangeOnline ), создайте группу рассылки Exchange и добавьте в нее аккаунты, от которых вы хотите разрешить отправку писем:

New-DistributionGroup -Name "azappSendasAllowed" -Type "Security" -Members @("sender@winitpro.onmicrosoft.com")

Опционально можно скрыть группу из адресной книги Exchange:
Set-DistributionGroup -Identity azappSendasAllowed -HiddenFromAddressListsEnabled $true

Теперь нужно создать политику и привязать ее к вашему Azure AppID:

New-ApplicationAccessPolicy -AppId "46692ad-f8a0-123f-8cca-432102de3bcf" -PolicyScopeGroupId azappSendasAllowed -AccessRight RestrictAccess -Description "Resrtict Sendas"

New-ApplicationAccessPolicy - ограничить отправку писем приложением Azure

Вы можете проверить теперь от имени каких адресов может отправлять письма ваше приложение:

Test-ApplicationAccessPolicy -Identity Alex@winitpro.onmicrosoft.com -AppId "46692ad-f8a0-123f-8cca-432102de3bcf"
Test-ApplicationAccessPolicy -Identity Dmitriy@winitpro.onmicrosoft.com -AppId "46692ad-f8a0-123f-8cca-432102de3bcf"

Команда вернёт Denied или Granted.

Test-ApplicationAccessPolicy - проверить права на отправку письма

Отправить письмо из Exchange Online с помощью Invoke-RestMethod
Для отправки письма через Exchange Online/Microsoft 365 можно использовать метод sendMail в REST API. Для вызова метода используется стандартный PowerShell командлет Invoke-RestMethod. Чтобы отправить письмо через Microsoft Graph API, вам нужно выполнить аутентификацию в Azure и получить токен.

$AccessSecret= "d-8jM3ZUG87du-syZd32k01q.gkssa3mH3v
$AzureAppID = "46692ad-f8a0-123f-8cca-432102de3bcf"
$tenantID="26216542-465a-407e-a17d-2bb4c3e3313b"

Теперь подключаемся к Graph API и выполняем аутентификацию:

$tokenBody = @{
Grant_Type    = "client_credentials"
Scope         = "https://graph.microsoft.com/.default"
Client_Id     = $AzureAppID
Client_Secret = $AccessSecret
}
$tokenResponse = Invoke-RestMethod -Uri "https://login.microsoftonline.com/$tenantID/oauth2/v2.0/token" -Method POST -Body $tokenBody
$headers = @{
"Authorization" = "Bearer $($tokenResponse.access_token)"
"Content-type" = "application/json"
}

Теперь можно отправить письмо:

$MailFrom = "sender@winitpro.onmicrosoft.com"
$MailTo = "dmitriy@winitpro.onmicrosoft.com"
$URLsend = "https://graph.microsoft.com/v1.0/users/$MailFrom/sendMail"
$BodyJsonsend = @"
{
"message": {
"subject": "Test email from Microsoft Graph API",
"body": {
"contentType": "HTML",
"content": "This email is sent via <br>
Microsoft GRAPH API<br>"
},
"toRecipients": [
{
"emailAddress": {
"address": "$mailto"
}
}
]
},
"saveToSentItems": "true"
}
"@
Invoke-RestMethod -Method POST -Uri $URLsend -Headers $headers -Body $BodyJsonsend

Проверьте, что письмо было успешно доставлено в ящик пользователя Exchange.

Отправить письмо через Invoke-RestMethod и Microsoft Graph API

Отправить письмо с помощью командлета Send-MgUserMail из модуля Microsoft.Graph
При использовании командлета Invoke-RestMethod для отправки письма нужно полностью построить объект письма в JSON формате. Для более простой отправки можно использовать командлет Send-MgUserMail из модуля Microsoft.Graph.

Установите модуль и импортируйте его в вашу сессию PowerShell:

Install-Module Microsoft.Graph
Import-Module Microsoft.Graph

В этом примере мы выполним аутентификацию и подключимся к тенанту Azure AD с помощью сертификата:

$certThumbprint = "7E82C3A560737C7894562593926E495777ECDB75"
$AzureAppID = "46692ad-f8a0-123f-8cca-432102de3bcf"
$tenant="winitpro.onmicrosoft.com"
Connect-MgGraph -TenantId $Tenant -ClientId $AzureAppID -CertificateThumbprint $certThumbprint
Теперь сформируем простое письмо с HTML телом и вложением:
$MailFrom = "sender@winitpro.onmicrosoft.com"
$MailTo = "dmitriy@winitpro.onmicrosoft.com"
$msgBody = “This is <br> test <br> Graph API maile <br>”
$Attachment = "C:\Temp\app.log"
$AttachmentContentBytes = [System.IO.File]::ReadAllBytes($Attachment)
$AttachmentBase64 = [System.Convert]::ToBase64String($AttachmentContentBytes)
$Message = @{
Subject = "Hello World from GraphAPI"
Body = @{
ContentType = "HTML"
Content = $msgBody
}
ToRecipients = @(
@{
EmailAddress = @{
Address = $MailTo
}
}
)
Attachments = @(
@{
"@odata.type" = "#microsoft.graph.fileAttachment"
Name = (Split-Path $Attachment -Leaf)
ContentType = "application/octet-stream"
ContentBytes = $AttachmentBase64
}
)
}

Send-MgUserMail отправить письмо через Exchange Online

После того, как письмо сформировано, его можно отправить:

Send-MgUserMail -UserId $MailFrom -Message $Message
