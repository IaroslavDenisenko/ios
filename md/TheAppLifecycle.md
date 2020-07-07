
 **[Back](https://orientcue.github.io/ios/ "Table of Content")**

# The App Life Cycle

<!-- TOC -->
- [How to app start](#how-to-app-start)          
- [Функция main()](#функция-main)    
- [UIApplication](#uiapplication)    
- [AppStates](#appstates)    
- [AppDelegate-Based Life-Cycle](#appdelegate-based-life-cycle)    
- [Application termination 🔥](#application-termination-)    
- [Scene-Based Life-Cycle](#scene-based-life-cycle)    
- [Background execution](#background-execution)    
- [Useful materials 🤓](#useful-materials-)
<!-- /TOC -->


Жизненный цикл приложения в iOS довольно важная тема если вы хотите создавать действительно классные приложения с точки зрения User Experience.


## How to app start

Рассмотрим этапы запуска приложения:


<img src="https://github.com/OrientCue/ios/blob/master/_resources/7c96cd6bbd2648de890740963a4f3198.png?raw=true">


### The App is not running
Включив телефон, и находясь на экране Home, или, с точки зрения системы его можно назвать Springboard ваше приложение находится в не запущенном состоянии
### Springboard launches the App
После того как пользователь нажимает на иконку приложения, Springboard инициирует запуск приложения. 
### The App is loading into the memory
Ваше приложение и все связанные с ним библиотеки будут загружены в память, пока Springboard анимирует запуск приложения и отображает на экране его livescreen. 
### The App is launched. App delegate receives the notifications
В итоге ваше приложение запускается и ApplicationDelegate начинает получать уведомления. =
### Функция main()  
Точкой входа в приложение является функция main(). Внутри функции main вызывается функция UIApplicationMain(), которая создает экземпляр класса UIApplication, и назначает этому экземпляру делегата, относящейся к типу, который мы указываем в качестве последнего параметра функции  UIApplicationMain. В нашем случае, это тип AppDelegate
```objc
int main(int argc, char * argv[]) {
    NSString * appDelegateClassName;
    @autoreleasepool {
        // Setup code that might create autoreleased objects goes here.
        appDelegateClassName = NSStringFromClass([AppDelegate class]);
    }
    return UIApplicationMain(argc, argv, nil, appDelegateClassName);
}
```


## UIApplication
Обеспечивает централизованный контроль и координацию приложений, работающих в iOS. по-простому, это представитель системы в вашем приложении. Приложение всегда имеет только один экземпляр класса UIApplication. Т.е. он реализует шаблон синглтон. 

* * *

## AppStates
Наше приложение всегда имеет нейкое состояние, и эти состояния могут изменятся во времени. Всего у приложения могут быть пять состояний. 

- **Not Running**
 Приложение не было запущено или было остановлено системой. Не запущено, не загружено в память и ничего не выполняет. 
- **Inactive**
 Состояние, когда приложение уже запущено, находится на переднем фоне(Foreground), но еще не получает никаких событий. Обычно, это состояние сохраняется на короткий промежуток времени, и является промежуточным или переходным между другими состояниями. Пока приложение находится в этом состоянии, мы не можем взаимодействовать с пользовательским интерфейсом. 
- **Active**
 Состояние, когда приложение уже запущено, находится на переднем фоне и получает событий. Это состояние является нормальным и рабочим для приложений на переднем фоне. В данном состоянии пользователь может взаимодействовать с пользовательским интерфейсом и наблюдать отлик на свои действия. Вся основная работа выполняется в данный момент. 
- **Background**
 Приложение находится в фоновом режиме(background) и может выполняет некоторый код. Приложение переходит в это состояние, после чего через короткий промежуток времени переходит в состояние Suspended. В состоянии background приложение находится небольшой промежуток времени, который зависит от текущего состояния системы и текущего приложения. 
- **Suspended**
 Приложение находится в фоновом режиме(background) и НЕ может выполняет код. Система переводит приложение в данное состояние из состояния background автоматически и не уведомляет приложение перед тем, как сделать это. Пока приложение находится в состоянии Suspended, оно все еще находится в памяти, но уже ничего не выполняет. Когда система не хватает ресурсов, система может убить приложение без каких-либо предупреждений. 


## AppDelegate-Based Life-Cycle
В iOS 12 и раньше, и в приложениях, которые не поддерживают такие вещи как сцены, UIKit доставляет все события, связанные с жизненным циклом в UIApplicationDelegate. Объект UIApplicationDelegate управляет всеми окнами вашего приложения включая те которые отображаются на отдельных экранах, в результате, изменения состояния приложения затрагивают все приложение включая пользовательский интерфейс, находящийся на внешних дисплеях. 
Схема показывает переходы состояния приложения в рамках AppDelegate: 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/e59d43998cb14d5e96189e27134d3f82.png?raw=true">

После запуска, система вводит приложение в состояние Inactive или background, в зависимости от того, отображается ли UI на экране. Если приложение выходит на передний фон (Foreground), система переводит приложение в состояние Active. После этого состояние изменяется между Active и Background пока приложение не будет убито. 
Переходы между состояниями сопровождаются вызовами методов внутри вашего AppDelegate. Эти методы позволяют вам реагировать на изменения состояния.

Список методов которые могут вызываться при смене состояния внутри AppDelegate:
```objc
- (BOOL)application:(UIApplication *)applicationwillFinishLaunchingWithOptions:(NSDictionary *)launchOptions;
- (BOOL)application:(UIApplication *)applicationdidFinishLaunchingWithOptions:(NSDictionary *)launchOptions;
- (void)applicationDidBecomeActive:(UIApplication *)application;
- (void)applicationWillResignActive:(UIApplication *)application;
- (void)applicationDidEnterBackground:(UIApplication *)application; - (void)applicationWillEnterForeground:(UIApplication *)application;
- (void)applicationWillTerminate:(UIApplication *)application;
```


## Application termination 🔥
Приложение всегда должно быть готово к завершению в любое время, и мы не должны ждать удобного момента чтобы сохранить пользовательские данные или завершить критические задачи. 
**‼️Bad practice ‼️**
Не стоит рассчитывать на метод **`applicationWillTerminate`**:
Он вызывается только для приложений которые находятся не в состоянии Suspended. Т.е. если вы вышли на HomeScreen и пытаетесь убить ваше приложение из режима многозадачности, то велика вероятность что метод не сработает. Также метод не вызовется если произошла перезагрузка устройства. 

* * *
## Scene-Based Life-Cycle
Если ваше приложение поддерживает сцены, UIKit посылает события жизненного цикла для каждой из сцен. Сцена представляет собой один экземпляр пользовательского интерфейса вашего приложения, работающего на устройстве. Пользователь может создавать несколько сцен для каждого приложения, а также отображать и скрывать их отдельно. Так как каждая сцена независимо от других может быть скрыта или показана, то у каждой из сцен должен быть свой жизненный цикл, каждая из сцен может находится в своем состоянии. SceneDelegate (или другой класс, соответствующий протоколу UIWindowSceneDelegate) управляет жизненным циклом сцен. 
Например, в iOS13 в приложении Safari у вас есть возможность создавать несколько отдельных окон.
Схема показывает изменения состояния для сцен: 



<img src="https://github.com/OrientCue/ios/blob/master/_resources/3797e51351eb44de89f1cd4ac3196570.png?raw=true">


Когда пользователь или система запрашивают новую сцену для вашего приложения, UIKit создает ее и устанавливает ей состояние Unattached. После этого сцена переходит в Foreground, где она отображается на экране. Сцена, запрошенная системой, обычно переходит в Background где она может обрабатывать некоторые события, например, трекинг локации. Когда пользователь сворачивает приложение, UIKit переводит ассоциированную с ним сцену в состояние Background и затем в состояние Suspended. UIKit может в любое время перевести сцену из состояния Suspended и Background в состояние Unattached, например, для освобождения ресурсов. 
Список методов которые могут вызываться при смене состояния внутри SceneDelegate:

```objc
- (void)scene:(UIScene *)scene
    willConnectToSession:(UISceneSession *)session
                 options:(UISceneConnectionOptions *)connectionOptions;
- (void)sceneDidDisconnect:(UIScene *)scene;
- (void)sceneDidBecomeActive:(UIScene *)scene;
- (void)sceneWillResignActive:(UIScene *)scene;
- (void)sceneWillEnterForeground:(UIScene *)scene;
- (void)sceneDidEnterBackground:(UIScene *)scene;
```

* * *

## Background execution
Возможно ли что-либо свое выполнять в состоянии background? Да, можно. Существует только два способа выполнять какие-либо задачи в состоянии background, это: 

1. 	**Краткосрочные задачи (short running tasks)** 
1. 	**URLSession background configuration**

Выполняя краткосрочные задачи, вы можете придерживаться следующих трех техник:

1. 	Если вы выполняете задачу, которую начали на **Foreground**, то можете запросить у системы дополнительное время чтобы выполнить данную задачу, когда приложение будет в состоянии **Background**. В данном случае система выделит вам некоторый промежуток времени после перехода приложения в состояние Background, после завершения задачи, переводит приложение в состояние Suspended. Однако, нет никакого стандартизированного промежутка времени, которое система может вам выделить. Никто не дает гарантии что вы успеете завершить вашу задачу. 
1. 	Приложение, которое инициировало загрузку на **Foreground**, может передать управление этими загрузками в систему, однако это не гарантирует завершение загрузки, так как система может в любой момент перевести приложение в состояние **Suspended** или вовсе убить его. 
1. 	Если приложению требуется в состоянии Background выполнять какие-то специфические типы задач, то вы как разработчик можете определить типы этих задач в capabilities вашего проекта. Это могут быть такие задачи, как например воспроизведение аудио, отслеживание локации, работа с bluetooth. 

Система накладывает жесткие ограничения на работу в состоянии **Background**. Поэтому вы никогда не должны выполнять какую-то работу в Background, за исключением случаев когда это улучшает общий UX вашего приложения.  

* * *
## Useful materials 🤓


[Apple Managing Your App's Life Cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)

[UIApplicationDelegate - Apple Developer](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)

[UISceneDelegate - UIKit | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiscenedelegate)




