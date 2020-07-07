
 **[Back](https://orientcue.github.io/ios/ "Table of Content")**

# Networking

<!-- TOC -->
- [HTTP](#http)
- [Cocoa Networking](#cocoa-networking)
   - [NSURLSession](#nsurlsession)
   - [Configuration creation](#configuration-creation)
   - [Ephemeral Session Configuration](#ephemeral-session-configuration)
   - [Session creation](#session-creation)
- [NSURLSession tasks](#nsurlsession-tasks)
   - [DataTask](#datatask)
   - [Upload Task](#upload-task)
   - [Download Task](#download-task)
   - [Common methods](#common-methods)
   - [TIPS:](#tips)
- [NSURLSession delegate](#nsurlsession-delegate)
   - [NSURLSessionDelegate](#nsurlsessiondelegate)
   - [NSURLSessionTaskDelegate](#nsurlsessiontaskdelegate)
   - [NSURLSessionDataDelegate](#nsurlsessiondatadelegate)
   - [NSURLSessionDownloadDelegate](#nsurlsessiondownloaddelegate)
- [NSURLSession Summary](#nsurlsession-summary)
- [Background sessions](#background-sessions)
   - [Things to avoid 📛](#things-to-avoid-)
   - [Best practices 👌](#best-practices-)
- [Handle App Suspension](#handle-app-suspension)
- [Security](#security)
   - [App Transport Security (ATS)](#app-transport-security-ats)
- [Authentication challenge](#authentication-challenge)
- [Data Formats](#data-formats)
   - [JSON](#json)
      - [NSJSONSerialization](#nsjsonserialization)
   - [XML](#xml)
      - [NSXMLParser](#nsxmlparser)
      - [Parser creation](#parser-creation)
      - [Common NSXMLParserDelegate method](#common-nsxmlparserdelegate-method)
- [Useful materials 🤓](#useful-materials-)
<!-- /TOC -->

## HTTP
**HTTP (Hypertext Transfer Protocol)** - протокол прикладного уровня для установления связи между системами. Общение между клиентом и сервером осуществляется через сообщения, есть два типа сообщений - **Request** (запрос) который клиент отправляет чтобы инициировать какие-то действия на сервере и **Response** (ответ) который клиент получает от сервера. 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/2d964899c474496392e4ceb94b59ccfe.png?raw=true">

Структура запроса и ответа довольно-таки похожа: 

<img src="https://github.com/OrientCue/ios/blob/master/_resources/09e610820852426796fa244b5ec7218b.png?raw=true">

**Start line** - описывает запрос, или его статус. В нем указывается HTTP метод, который описывает действие (**GET**, **POST** etc.). Всего бывает 9 методов, но основные из них 4:

- **GET (Read)** - используется чтобы получить какой-то существующий ресурс. В целевой URL обычно включаются параметры необходимые для сервера чтобы найти и вернуть требуемый ресурс. 
- **POST (Create)** - обычно метод используется для создания нового ресурса, обычно в теле запроса отправляется какой-то контент. Но зачастую метод POST используется и для других целей. 
- **PUT (Update)** - используется чтобы обновить существующий ресурс, тело запроса должно содержать данные необходимые для обновления. 
- **DELETE (Delete)** - удаляет определенный ресурс

Кроме HTTP методов тут указывается целевой путь и версия протокола HTTP. В ответе еще возвращается код статуса запроса и его текст. 

HTTP Headers - опциональный набор заголовков описывает либо весь запрос-ответ, либо тело запроса или ответа. Условно все заголовки можно разделить на три группы: **general headers** которые применяются ко всему сообщению, **request headers** которые корректируют запрос, **entity headers** которые применяются к телу запроса. 

**<Пустая строка>**

**Body** - опциональное тело, которое содержит данные, ассоциированные с запросом. Не все запросы нуждаются в этом, поэтому тело может отсутствовать.

## Cocoa Networking

В iOS выделяют три больших слоя API которые работают с сетью. 
- **BSD Networking** - Низкоуровневый слой, сокеты Беркли. 
- **CoreFoundation** - Низкоуровневый слой Apple. 
- **Foundation** - высокоуровневый API Apple.



<img src="https://github.com/OrientCue/ios/blob/master/_resources/cfcb84f1f372468aa9d1fb87e8cbc419.png?raw=true">

## NSURLSession
Класс **NSURLSession** и связанные с ним классы предоставляют API для скачивания и загрузки данных по указанному URL. 

Concepts
Session and configuration
Session tasks
Session delegate
Credentials (credential storage) 
Cookies (cookie storage) 
Cache
configuration
Объект **NSURLSessionConfiguration** определяет поведение и набор свойств, влияющих на загрузку и скачивание данных с использование объекта **NSURLSession**. 
Когда вы скачиваете или загружаете данные создание объекта конфигурации сессии — это первый шаг, который вы должны сделать. Важно создать этот объект и сконфигурировать его должным образом перед тем, как вы инициализируете объект NSURLSession. Объект NSURLSession создает копию объекта конфигурации для своего использования, когда сессия создана, она игнорирует любые изменения, которые вы делаете в объекте конфигурации. Если вам понадобилось изменить какие-то параметры конфигурации, вам необходимо модифицировать объект конфигурации и ипользовать его для создания новой сессии. 

## Configuration creation

```objc
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
```

Вернет вам экземпляр с настройками по умолчанию. Далее вы можете изменять его, либо использовать как есть.


## Ephemeral Session Configuration

 ```objc
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration ephemeralSessionConfiguration];
```

Данная конфигурация отличается тем, что настраивает сессию так чтобы не сохранять кэш, куки, криды. Такое поведение соответствует режиму инкогнито в браузере. 

## Background Session Configuration

```objc
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration backgroundSessionConfigurationWithIdentifier:@"com.rsschool.background"];
```

Бэкграунд конфигурация. Создается с идентификатором, по которому можно потом восстановить конфигурацию. Такая конфигурация позволяет сессии выполнять работу даже в режиме background, suspended или даже когда приложение остановлено или произошла его аварийной остановка. 

## Properties that affect transfers
Примеры некоторых свойств, которые можно настроить в объекте конфигурации:
- **TLS (Transport Layer Security)** - протокол шифрования, который был разработан для обеспечения безопасности в процессе коммуникации в сети между клиентом и сервером. До TLS использовался протокол SSL. 
- **Cellular usage** - имеется возможность настроить использование мобильной сети. Например, ограничить доступ если устройство имеет доступ в интернет только через мобильную сеть. 
- **Network service type** - данная настройка позволяет вам указать системе для чего используется передаваемый трафик в рамках сессии. Например, это может быть трафик для передачи голосовых сервисов, для передачи видео, стриминга, работы в бэкграунде и т.д. Данная настройка поможет системе приоритизировать потоки. 
- **Cookie polices** - настройка различных политик для куки. По умолчанию куки хранятся в shared cookie storage, если вам нужно хранить их в другом месте, вы можете это переопределить. 
- **Cache polices** - настройка различных политик для кэша. По умолчанию хранится в shared url cache. Но если нужно, это можно изменить. 

## Additional headers
В конфигурации вы можете добавлять свои HTTP headers для сессии посылаемой с использованием данной конфигурации. 

```objc
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration
                                                      defaultSessionConfiguration];
NSString *userAgentString = @"AppName/com.rsschool.app (iPhone 11 Pro; iOS 13.5.1)";
configuration.HTTPAdditionalHeaders = @{@"Accept" : @"application/json", @"Accept-Language" : @"en",@"User-Agent" : userAgentString};
```

В данном примере мы устанавливаем формат JSON, язык английский и User-Agent. Это информация необходимая серверу что бы он понимал, например с какого устройства ему посылается запрос. 

## Session creation
Есть три способа создать сессию. Первый, вызвать sharedSession: 

```objc
NSURLSession *session = [NSURLSession sharedSession];
```

Функция возвращает синглтон для сессии, для которой не нужен объект конфигурации. Самый простой способ создать сессию для базовых операций. 

Второй способ, вызвать метод sessionWithConfiguration:

```objc
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration
                                             defaultSessionConfiguration];
NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration];                                          
```

На выходе получаем сессии, но у которой все еще нет делегата. Если вы создаете сессиию с конфигурацией по умолчанию, то она ничем не будет отличатся от sharedSession. 

Третий способ

```objc
NSOperationQueue *queue = [NSOperationQueue new];
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration
                                           defaultSessionConfiguration];
NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration
                                                     delegate:self  delegateQueue:queue];
```

## NSURLSession tasks
Если вам нужно сформировать запрос, получить ответ - вам необходимо создать задачу(task) для вашей сессии. Иерархия классов для задач выглядит следующим образом:

<img src="https://github.com/OrientCue/ios/blob/master/_resources/60e9b879641048879278ce58c7e985fd.png?raw=true">

Базовым классом является **NSURLSessionTask**, данный класс по своей сути является абстрактным, использовать его напрямую вы не можете. У него есть такие наследники как   **NSURLSessionDataTask** и **NSURLSessionDownloadTask**. У NSURLSessionDataTask есть свой наследник **NSURLSessionUplaodTask**. В итоге мы имеем три наиболее используемых класса для задач, которые соответствуют трем типам задач: Data Task,  Download Task и UplaodTask. Есть еще WebSocket task.

## DataTask
Посылает и принимает данные использует NSData объект. Эти задачи предназначены для коротких и интерактивных запросов. С помощью этих задач мы скачиваем данные в оперативную память. Файлы тоже можно сказать, однако с начала они будут скачаны в оперативную память, но позже можно будет сохранить их на диск. 
Первый способ создать data task, это вызвать dataTaskWithURL: и передать туда целевую URL. 

```objc
NSURL *url = [NSURL URLWithString:@"https://rs.school"];
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDataTask *dataTask = [session dataTaskWithURL:url];
```

Второй способ, это вызвать **dataTaskWithRequest**: с передачей объекта типа NSURLRequest. Обычно реквест содержит какую-то дополнительную информацию.

```objc
NSURL *url = [NSURL URLWithString:@"https://rs.school"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request];
```

Также есть важно отличие между инициализации с dataTaskWithURL в отличии от dataTaskWithRequest. По умолчанию, HTTP метод у запроса - GET. Если вам нужен любой другой метод, то вы никак не можете его настроить для создания объекта NSURLRequest. Поэтому вам нужно создать свой экземпляр NSURLRequest и настроить его. 

## Upload Task

Эти задачи похожи на Data Task но чаще используется для отправки данных, чаще в форме файлов и в отличии от Data Task поддерживают бэкграунд загрузки когда приложение например не запущено.
Создать такую задачу можно тремя способами. Первый использовать метод **uploadTaskWithRequest:fromData:** куда передаем инстанс NSURLRequest и объект класса NSData. 

```objc
NSURL *url = [NSURL URLWithString:@"https://rs.school"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
NSData *data = [NSData new];
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionUploadTask *uploadTask = [session uploadTaskWithRequest:request
                                                           fromData:data];
```

Второй способ с использованием метода **uploadTaskWithRequest:fromFile:** куда передаем инстанс реквеста и локальный URL к файлу который необходимо загрузить.

```objc
NSURL *url = [NSURL URLWithString:@"https://rs.school"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
NSURL *fileUrl = [NSURL fileURLWithPath:[NSTemporaryDirectory() stringByAppendingPathComponent:@"quiz.pdf"]];
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionUploadTask *uploadTask = [session uploadTaskWithRequest:request fromFile:fileUrl];
```

третий способ с использованием метода uploadTaskWithStreamedRequest: куда передаем инстанс NSURLRequest

```objc
NSURL *url = [NSURL URLWithString:@"https://rs.school"];
NSURLRequest *request = [NSURLRequest requestWithURL:url]; 
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionUploadTask *uploadTask = [session uploadTaskWithStreamedRequest:request];
```


## Download Task
Эти задачи позволяют скачивать данные с сохранением их напрямую на диск и также они поддерживают бэкграунд скачивание. 
Для создания также существую три способа: 
Первый, с помощью вызова метода **downloadTaskWithURL**: куда передаем URL источника. 

```objc
NSURL *url = [NSURL URLWithString:@"https://rs.school"];
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDownloadTask *downloadTask = [session downloadTaskWithURL:url];
```

Второй, с использованием инстанса NSURLRequest. 

```objc
NSURL *url = [NSURL URLWithString:@"https://rs.school"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDownloadTask *downloadTask = [session downloadTaskWithRequest:request];
```

Третий, который поможет продолжить скачивание если ранее оно было отменено, вызов метода **downloadTaskWithResumeData**: который содержит истанса класса NSData который содержит данные для продолжения загрузки. 

```objc
NSData *data = [NSData new];
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDownloadTask *downloadTask = [session downloadTaskWithResumeData:data];
```

## Common methods
По умолчанию, все задачи приостановлены, необходимо вызвать метод **resume**: чтобы начать выполнение новой или приостановленной задачи. 
`[downloadTask resume];`
Чтобы временно приостановить задачу, вызываем метод suspend:
`[downloadTask suspend];`
Чтобы отменить задачу, вызываем метод cancel:
`[downloadTask cancel];`

## TIPS:
Если пользователь зашел на экран и началась загрузка, а потом он закрыл его, то загрузка не будет приостановлена, хотя данные уже не нужны. Пользуйтесь отменой загрузок. 

Состояние текущей задачи определено в свойстве state. Задача может находится в состоянии **Running**, **Suspended**, **Canceling** и **Completed**

`@property (readonly) NSURLSessionTaskState state;`

Для задачи можно установить приоритет выполнения который выражается в числе от 0 до 1. 

`@property float priority`

Свойство содержит ссылку на объект текущего запроса. 

`@property (nullable, readonly, copy) NSURLRequest  *currentRequest;`

Задача может менять объект реквеста в течении времени, и данная ссылка содержит объект реквеста с которым создавалась задача. 

`@property (nullable, readonly, copy) NSURLRequest  *originalRequest;`  

Это свойство содержит ответ объекта от сервера для текущей задачи.

`@property (nullable, readonly, copy) NSURLResponse *response;`

## NSURLSession delegate
Есть 4 протокола относящиеся к NSURLSession delegate:

- `NSURLSessionDelegate` 
- `NSURLSessionTaskDelegate` 
- `NSURLSessionDataDelegate` 
- `NSURLSessionDownloadDelegate`

**!!!ВАЖНО!!! Сессия держит сильную ссылку на объект делегата пока приложение не завершит работу или вы явно не вызовите invalidateAndCancel: или finishTasksAndInvalidate:. Если этого не сделать явно, будет наблюдаться утечка памяти.**



## NSURLSessionDelegate

Ответственен за методы, относящиеся непосредственно к сессии. Если вы используете задачи без completion, то ошибки и результат необходимо обработать через NSURLSessionDelegate. Также данный делегат позволяет наблюдать за аутентификацией для всей сессии. 

## NSURLSessionTaskDelegate

Расширяет возможности NSURLSessionDelegate, также позволяет наблюдать за аутентификацией для отдельных запросов и также позволяет обрабатывать результат и ошибки. 

## NSURLSessionDataDelegate

Расширяет возможности NSURLSessionTaskDelegate. Позволяет получать данные уже в процессе загрузки. 
Когда мы создаем задачу, по умолчанию ее состояние suspended. После вызова метода resume: состояние задачи переходит в running. После того как мы получили хедеры от сервера, вызывается метод: **`didReceiveResponse`**:. Когда получили данные от сервера, вызывается метод: **`didReceiveData`**:, дальше вызывается **`willCacheResponse`**:. В конце вызывается делегатный метод: **`didCompleteWithError`**:, одновременно с этим состояние переходит в Finished 

## NSURLSessionDownloadDelegate
Расширяет возможности NSURLSessionTaskDelegate. Позволяет отслеживать процесс загрузки и предоставляет URL к локальному файлу, в который будет записан результат загрузки. 
Когда мы создаем задачу, по умолчанию ее состояние suspended. После вызова метода resume: состояние задачи переходит в running. После того как мы получили хедеры от сервера, вызывается метод: **`didWriteData`**:. После завершения скачивания вызывается метод: **didFinishDownloadingToUrl**:. В конце вызывается делегатный метод: didCompleteWithError:, одновременно с этим состояние переходит в Finished.

## NSURLSession Summary
Процесс работы с NSURLSession задачами это по большому счету процесс из трех шагов. 
1. Создание конфигурации
2. Создание сессии
3. Создание задачи(таски)

**TIPS**:
Не нужно для каждой задачи создавать новую сессию. Если вы, например, постоянно работаете с одним сервером или для запросов подходит одна и та же конфигурация то используйте одну сессию.  

<img src="https://github.com/OrientCue/ios/blob/master/_resources/7d63771c3e2d4844bc8e6326bdee42de.png?raw=true">

## Background sessions

Эти сессии можно использовать даже когда приложение не запущено и аварийной завершилось. На практике бэкграунд сессии используются не для больших и тяжелых операций, а для завершения операций, которые были начаты, когда приложение было запущено. Система имеет ряд ограничений, с которыми вы можете столкнутся при выполнении задач на бэкграунд сессии, например вы можете выполнять какие-то большие операции или делать много запросов, в этом случае система может убивать эти запросы. После завершения работы задач, система может разбудить ваше приложение для выполнения каких-то дополнительных действий.

## Things to avoid 📛

После того как система перезапускает или будит ваше приложение по завершению бэкграунд загрузки, она может понижать приоритет ваших загрузок чтобы реже запускать приложение. 
Не очень хорошей идеей будет качать много мелких файлов, лучше скачивать один большой архив.
Система использует delay для старта новых загрузок в бэкграунде, чтобы вы не злоупотребляли бэкграунд загрузками. Ваша задача не начнется пока не пройдет задержка. Кроме того, delay увеличивается каждый раз, когда система перезапускает или будит ваше приложении. 

## Best practices 👌

Скачивайте один большой, а не много мелких файлов. 
Правильно реализуйте механизм восстановления вашей сессии завершения задач после завершения загрузок.

## Handle App Suspension
Разные состояния приложения влияют на то, как ваше приложение взаимодействует с бэкграунд загрузкой. Если на foreground происходит все как обычно, то в состоянии suspended или not-running все иначе. Если ваше приложении в бэкграунде, то система может перевести его в состояние suspended в любой момент, а бэкграунд загрузка продолжится в другом процессе. Когда бэкграунд загрузка завершается, система будит ваше приложении и по идентификатору вызывает метод:

```objc
- (void)application:(UIApplication *)application 
handleEventsForBackgroundURLSession:(NSString *)identifier 
completionHandler:(void (^)(void))completionHandler
```

Этот метод в качестве второго параметра получает идентификатор сессии, который вы указываете при создании. Вам нужно снова создать сессию с этим идентификатором и восстановить делегата. Также этот метод в качестве третьего параметра получает completionHandler, который нужно сразу-же в подходящем для этого месте, например в классе, который является делегатом для вашей сессии. 
После того как все необходимые события были доставлены, система вызывает NSURLSessionDelegate метод: 

```objc
- (void)URLSessionDidFinishEventsForBackgroundURLSession:(NSURLSession *)session
```

внутри этого метода вам необходимо вызвать ранее сохраненный completionHandler. Этот метод может вызываться не на главной очереди, так что необходимо перейти на главную очередь. После вызова completionHandler задача завершает свою работу и вызывается метод:

```objc
- (void)URLSession:(NSURLSession *)session
      downloadTask:(NSURLSessionDownloadTask *)downloadTask
      didFinishDownloadingToURL:(NSURL *)location
```

После него загрузка считается завершенной. 

## Security
### App Transport Security (ATS)

Начиная с iOS9 запрещено использовать HTTP соединение, в iOS появилась такая технология как ATS, которая призвана улучшить приватность и сохранность данных для всех приложений. ATS требует, чтобы все HTTP соединения, которые вы совершаете, используя URLSession использовали HTTPS и блокирует те соединения, которые не соответствуют требованиям безопасности. Вы можете ослабить или расширить уровень защиты добавив словарь настроек к ключу
**`NSAppTransportSecurity`** в info.plist файле. 
По ключу **NSExceptionDomains** можно задать исключения для конкретных доменов. Есть несколько свойств и исключений, которые доступны по свои ключам. 



```objc
NSExceptionDomains : Dictionary {
      <domain-name-string> : Dictionary {
       NSIncludesSubdomains : Boolean
        NSExceptionAllowsInsecureHTTPLoads : Boolean
        NSExceptionMinimumTLSVersion : String
        NSExceptionRequiresForwardSecrecy : Boolean
        NSRequiresCertificateTransparency : Boolean

} }
```

**`NSExceptionAllowsInsecureHTTPLoads`**

Самый часто используемый. Например, у нас есть сервер, которые поддерживает HTTP соединения, можно добавить этот сервер в исключения установив для него по этому ключу значение YES. Это позволит системе соединение только с этим сервером по протоколу HTTP. 

**`NSExceptionMinimumTLSVersion`**

Позволяет установить минимальную допустимую версию TLS. 

**`NSExceptionRequiresForwardSecrecy`**

Если установить значение в NO, то приложение не будет требовать у сервера поддержки perfect forward secrecy (PFS). 

**`NSRequiresCertificateTransparency`**

Ключ, который позволяет игнорировать ошибки Certificate Transparency (CT) — это протокол, который ATS может использовать для идентификации ошибочно или злонамеренно выданных сертификатов X.509.

Несмотря на то, что вы можете ослабить ограничения ATS важно понимать, вы всегда в первую очередь вы должны искать способы улучшения безопасности вашего сервера и поддерживать актуальные версии протокола безопасности, прежде чем добавлять исключения. Снижение ограничений ATS уменьшает безопасность вашего приложения. 

## Authentication challenge

Например, вы отправили запрос, который требует авторизации на сервере, например у вас истекло время для токена который вы получили ранее или вовсе хидер не предполагает авторизации. В таком случае, нам необходимо запрос каким-то образом авторизовать. Когда мы сталкиваемся с authentication challenge уведомляет об этом своего делегата чтобы мы могли это должным образом обработать. Поэтому если вы используете задачу с completionBlock, **без указания делегата**, вы не сможете обработать authentication challenge. 

Когда мы получаем валидный authentication challenge, первым делом сессия вызывает делегатный метод относящейся к задаче:

```objc
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
    	didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
    	completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition,
                   	          NSURLCredential * _Nullable))completionHandler
``` 

Если делегат задачи не отвечает на данное сообщение, то сессия вызывает свой делегатный метод:

```objc
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
 completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential * _Nullable))completionHandler
```

У нас есть три варианта как мы можем обработать authentication challenge:

- Предоставить пользователю ввести данные аутентификации
- Попробовать продолжить запрос без данных аутентификации
- Отменить authentication challenge

Чтобы понять какой вариант является подходящим, экземпляр challenge содержит информацию о том, чем он был вызван, сколько попыток уже было сделано и.т.д. 

## Data Formats
Когда клиент и сервер обмениваются информацией, большое значение имеет формат данных. А именно, отправитель должен использовать тот формат данных, с которым умеет работать получатель. Одними из самых распространенных форматов являются 

- **XML**
- **JSON**

## JSON

В настоящее время, многие новые API используют JSON - это очень простой формат представляющий собой отношение ключ-значение.  
```
{  //ключ   значение
    "crust": "original",
    "toppings": ["cheese", "pepperoni", "tomatoes"],
    "status": "cooking"
}
```

Иногда вам может понадобится записать целый объект в качестве какого-то ключа, это тоже возможно сделать. В примете атрибут "customer" в который помещен объект. 

```json
{
"crust": "original",
"toppings": ["cheese", "pepperoni", "tomatoes"], "status": “cooking”,
"customer": {
       "name": "Brian",
       "phone": "573-111-1111" 
   }
}
```

## NSJSONSerialization

Используйте NSJSONSerialization для парсинга JSON. Чтобы преобразовать полученный JSON объект, с которым вы можете работать в своем приложении используйте метод класса: 


```objc
+ (id)JSONObjectWithData:(NSData *)data
                 options:(NSJSONReadingOptions)opt
                   error:(NSError **)error;
```

Где data это объект содержащий JSON, options содержит опции для чтения, и error куда будет помещена ошибка в случае неудачного парсинга. 

## XML
Довольно старый, но мощный формат данных, в основном используется в Enterprice решениях, например в банковских приложениях. Как и JSON представляет из себя несколько простых строительных блоков которые помогают структурировать данные. Основным блоком в XML является Node, каждый node должен начинатся с открывающим тегом и заканчивается с закрывающим. Имя ноды говорит об атрибуте. Между тегами ноды находятся значения. 

```xml
<order>
    <crust>original</crust>
    <toppings>//Node open tag
       <topping>cheese</topping>
        <topping>pepperoni</topping> //Values
        <topping>garlic</topping>
    </toppings>//Node close tag
   <status>cooking</status>
</order>
```

## NSXMLParser
Используйте NSXMLParser для парсинга XML. Данный класс в отличии от NSJSONSerialization который с помощью одного метода позволяет получить Foundation объект, использует поэтапный парсинг используя несколько делегатных методов. 

## Parser creation
Чтобы создать парсер, используем инит метода куда передаем данные, содержащие XML. 

```objc
- (instancetype)initWithData:(NSData *)data;
```

Стартуем парсер вызовом метода: 

`- (BOOL)parse;`

## Common NSXMLParserDelegate method

Когда начинается парсинг документа, у делегата вызывается метод:

```objc
- (void)parserDidStartDocument:(NSXMLParser *)parser;
```

Когда заканчивается парсинг документа, у делегата вызывается метод:

```objc
- (void)parserDidEndDocument:(NSXMLParser *)parser;
```

Данный метод вызывает у делегата парсером когда парсер встречает открывающийся тег:

```objc
- (void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName
  namespaceURI:(NSString *)namespaceURI
 qualifiedName:(NSString *)qName
    attributes:(NSDictionary<NSString *, NSString *> *)attributeDict;
```

и вызывается у делегата парсером когда парсер встречает закрывающийся тег:

```objc
- (void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName
  namespaceURI:(nullable NSString *)namespaceURI
 qualifiedName:(nullable NSString *)qName;
```

.. и данный метод вызывается у делегата парсером когда передается весь или часть встреченного контента у текущего элемента:

```objc
- (void)parser:(NSXMLParser *)parser foundCharacters:(NSString *)string;
```


## Useful materials 🤓

[Apple. NSURLSession](https://developer.apple.com/documentation/foundation/nsurlsession)

[Apple. Networking](https://developer.apple.com/documentation/network)

[Apple. Downloading Files in the Background](https://developer.apple.com/documentation/foundation/url_loading_system/downloading_files_in_the_background)

[Mozilla HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)

[w3schools Introduction to XML](https://www.w3schools.com/xml/)

[w3schools JSON - Introduction](https://www.w3schools.com/js/js_json_intro.asp)

[Apple. NSXMLParser](https://developer.apple.com/documentation/foundation/nsxmlparser?language=objc)

[Apple. NSJSONSerialization](https://developer.apple.com/forums/thread/110414)

