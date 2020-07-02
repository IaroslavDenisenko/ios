# Autolayout.

<!-- TOC -->
           
- [Manual layout](#manual-layout)    
- [**Autoresizing Mask**](#autoresizing-mask)                       
- [**Autolayout**](#autolayout)        
    - [View attributes](#view-attributes)    
    - [**Constraint Priorities**](#constraint-priorities)    
    - [**Intrinsic content size**](#intrinsic-content-size)    
    - [Size that fits](#size-that-fits)    
    - [Content hugging](#content-hugging)    
    - [Content Compression Resistance](#content-compression-resistance)
    - [Create constraints in code](#create-constraints-in-code)    
    - [Constraint constructor](#constraint-constructor)    
    - [Visual Format Language (VFL)](#visual-format-language-vfl)    
    - [Anchors](#anchors)
- [Stack View](#stack-view)    
- [Safe area](#safe-area)    
- [Margins](#margins)
- [Trait collection](#trait-collection)    
- [Size Classes](#size-classes)    
- [UITraitEnvironment protocol](#uitraitenvironment-protocol)    
- [UIContentContainer protocol](#uicontentcontainer-protocol)    
- [Useful links 🤓](#useful-links-)
<!-- /TOC -->

**layout** - это просто вычисление размеров и позиции всех ваших представлений в иерархии. В идеале, любой layout должен отвечать на внешние и внутренние изменения. 

Примеры **внешних изменений**: 
1.  Изменения ориентации.
2.  Пользователь входит или выходит из Split View на iPad.
3.  Появление или изменения бара в верху экрана во время входящего звонка или записи звука. 
4.  Вы хотите поддерживать разные классы размеров.
5.  Вы хотите поддерживать разные размеры экрана.

##### Т.е все то что влияет на ваше приложение извне. 
---
**Внутренние изменения** происходят при изменении размера вьюх или элементов управления в интерфейсе. Примеры внутренних изменений: 
1.  Динамические изменения контента отображаемого приложения.
2.  Приложение поддерживает интернационализацию(несколько языков).
3.  Приложение поддерживает Dynamic Type (iOS)(когда пользователь может изменять размер шрифтов во время работы приложений).

---
Существует три основных подхода при разработке пользовательского интерфейса: 
1.  Ручная компоновка 
2.  Используя Autoresizing Mask
3.  Используя Autolayout 

***
## Manual layout
Подход основан ручном выставлении начального положения и размера каждого вью относительно родительского. Однако при любом изменении размера или ориентации экрана придется пересчитывать положение и размер элементов. Данный подход является мощным, гибким и быстрым инструментом, однако даже для сравнительного простого интерфейса может оказаться очень трудоемким.
* * *
## **Autoresizing Mask**
Позволяет указать как будут меняться размеры вью, при изменении размеров супервью. 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/f3e9e9534b234c32820ec70bc42d09c4.png?raw=true"> 


**Autoresizing Mask** поддерживает сравнительно небольшое количество настроек для компоновки интерфейса и не может обработать внутренние изменения. Обычно этот инструмент комбинируется с ручной компоновкой. До появления autolayout это был основной инструмент компоновки, хотя сейчас “под капотом” он основан на autolayout. Вкратце, принцип работы основан на указании того, какие края и какие размеры могут меняться при изменении родительского вью. 

#### UIView Property:

```objc
@property(nonatomic) UIViewAutoresizing autoresizingMask;    // simple resize. default is UIViewAutoresizingNone
```

#### UIViewAutoresizing
```objc
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```
**Использование:** 

```objc
self.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
```

В данном примере мы говорим, что наша проперти должна реагировать на изменение высоты и широты родительского вью.

## **Autolayout**

Система компоновки элементов интерфейса основания на ограничениях(constraint). Она позволяет разработчикам создавать адаптивный пользовательский интерфейс который нужным образом реагирует и на внешние, и на внутренние изменения. Autolayout динамически вычисляет размер и положение всех представлений в вашей иерархии на основе заданных правил. Вместо того чтобы думать о размерах каждого представления, вы думаете о том как эти представления связаны друг с другом. Одна связь между двумя представлениями и есть ограничение(constraint). Autolayout рассчитывает положение представлений основываясь на этих связях. В результате чего мы получаем адаптивный интерфейс. 

The relationship between two user interface objects that must be satisfied by the constraint-based layout system.
Declaration

`@interface NSLayoutConstraint : NSObject`

Each constraint is a linear equation with the following format:

`item1.attribute1 = multiplier × item2.attribute2 + constant`


<img src="https://github.com/OrientCue/ios/blob/master/_resources/2e0cdf83b2c64f72b7113326570d85e6.png?raw=true"> 


- **Item1**- первый объект должен быть либо вью, либо лэяут гайд.
- **Attribute1**- свойство первого элемента, на основе которого строится констр.
- **Relationship** это отношения левой и правой стороны, то есть это может быть и оператор equal, >=,<=.
- **Muliplier**- это число с плавающей точкой, на которое умножается значение с второго атрибута, т.е. если мы хотим, чтобы ширина одной вьюхи была больше другой, то мы можем сделать это через мультиплайер.
- **Constant**- смещение, которая в нашем случае говорит, что между вьюхами будет 8 точек.

Расположение ваших представлений можно представить как набор линейных уравнений, где каждый констрэйнт и является таким уравнением. Задача разработчика - составление такой группы уравнений которое имеет одно возможное решение. Когда Autolayout engine будет пытаться решить все ваши ограничения, он сможет прийти к вашему решению. 

* * *
### View attributes


<img src="https://github.com/OrientCue/ios/blob/master/_resources/acf2927bc8ff49f0b6deb242b2dd6d77.png?raw=true"> 

- Height
- Width
- Top
- Bottom 
- Leading / Left
- Trailing / Right 
- Center Y
- Center X
- Baseline


```objc
typedef NS_ENUM(NSInteger, NSLayoutAttribute) {
    NSLayoutAttributeLeft = 1,
    NSLayoutAttributeRight,
    NSLayoutAttributeTop,
    NSLayoutAttributeBottom,
    NSLayoutAttributeLeading,
    NSLayoutAttributeTrailing,
    NSLayoutAttributeWidth,
    NSLayoutAttributeHeight,
    NSLayoutAttributeCenterX,
    NSLayoutAttributeCenterY,
    NSLayoutAttributeLastBaseline,
#if TARGET_OS_IPHONE
    NSLayoutAttributeBaseline NS_SWIFT_UNAVAILABLE("Use 'lastBaseline' instead") = NSLayoutAttributeLastBaseline,
#else
    NSLayoutAttributeBaseline = NSLayoutAttributeLastBaseline,
#endif
    NSLayoutAttributeFirstBaseline API_AVAILABLE(macos(10.11), ios(8.0)),

#if TARGET_OS_IPHONE
    NSLayoutAttributeLeftMargin API_AVAILABLE(ios(8.0)),
    NSLayoutAttributeRightMargin API_AVAILABLE(ios(8.0)),
    NSLayoutAttributeTopMargin API_AVAILABLE(ios(8.0)),
    NSLayoutAttributeBottomMargin API_AVAILABLE(ios(8.0)),
    NSLayoutAttributeLeadingMargin API_AVAILABLE(ios(8.0)),
    NSLayoutAttributeTrailingMargin API_AVAILABLE(ios(8.0)),
    NSLayoutAttributeCenterXWithinMargins API_AVAILABLE(ios(8.0)),
    NSLayoutAttributeCenterYWithinMargins API_AVAILABLE(ios(8.0)),
#endif
    
    NSLayoutAttributeNotAnAttribute = 0
};
```


Есть одна тонкость, Left и Right всегда будут относится к левому и правому краю, а Leading и Trailing могут относится и к левому, и к правому краю в зависимости от локализации, где начинается письмо. Соответственно в еврейской или арабской локализации Leading и Trailing будут зеркальны. 
Атрибут Baseline обозначает линию выравнивания контента. Если взять, к примеру UITextView или UILabel то это будет линия, на которой пишется текст. 
Пример того, как можно построить интерфейс при помощи констрейнтов:


<img src="https://github.com/OrientCue/ios/blob/master/_resources/3c5f216ea7044a8e88378c3484c5ecb3.png?raw=true"> 


Допустим у нас есть вью, которая располагается внутри супервью, вью должно располагаться по середине. Есть несколько способов как это можно сделать:
	
- Задать отступ слева (Leading) и задать ширину (Width)
- Задать отступ слева и справа (Leading и Trailing). Ширина будет рассчитана автоматически.
- Задать отступ с одной из сторон, например слева, и сказать что наша вью будет располагаться по центру (Center X). В данном случае отступ справа будет рассчитан автоматически. 
Всегда следует придерживаться правила, указывая два констрейнта, т.е на каждое измерение (Размер и позиция). В данном примере опущены отступы по оси Y.

* * *
Пример адаптивного интерфейса, на нем в разных ориентациях размещено две вью с одинаковой шириной и высотой, с отступами по краям и между ними. 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/324010037df549b69d3228365dbfba9b.png?raw=true"> 

Есть несколько вариантов построить такой интерфейс. 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/93e360d457c74f9794aa6244407d2a56.png?raw=true"> 

<img src="https://github.com/OrientCue/ios/blob/master/_resources/bdfcf23e072b4490bd68ab594f82f53f.png?raw=true"> 


Для каждой вью задаются отступы по каждому краю и между ними и говорим что у них будет одинаковая ширина. 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/f2f517c8e4db4a71b467a2ffdcd8f4ac.png?raw=true"> 


В этом варианте для первой указываются отступы по краям и между вью, а для синей вью нужно указать что она будет такой же как первая по высоте или сделать для второй привязку Top и Bottom к аналогичным атрибутам красной вью. 

* * *

## **Constraint Priorities**


У каждого констрейнта есть приоритет от 1 до 1000. Констрейнт с приоритетом 1000 является обязательным, с более низким приоритетом являются необязательными. По умолчанию у всех констрейнтов приоритет 1000, т.е. они обязательные. Также можно добавлять свои необязательные констреинты, в данном случае Autolayout действует по следующему принципу: сперва он пытается найти соотношение, которое бы удовлетворяло всем констрейнтам начиная от высшего приоритета к низшему. Если заданное условие заданное каким-либо необязательным констрейнтом не может быть выполнено, то оно пропускается. Но стоит учесть, даже если необязательный констрейнт был отброшен, он все равно может оказать влияние на компоновку интерфейса, если после отбрасывания возникает неоднозначность, система пытается найти такое решение, которое устранит неоднозначность. 

* * *

## **Intrinsic content size**

Некоторые представления вью имеют внутренний размер, который определяется их контентом. У классов UIView есть свойство:
```objc
@property(nonatomic, readonly) CGSize intrinsicContentSize;
```
Это свойство возвращает размер, который описывает минимальное пространство, которое необходимо для отображения контента. Например, размер таких элементов как UILabel или UIButton зависит от количества текста и используемого шрифта. Размер UIImageView будет определятся изображение которые необходимо отобразить. Autolayout engine учитывает intrinsicContentSize и создает для каждого представления неявные внутренние констрейнты для ширины и высоты. Не все классы поддерживают это свойство.

**View** | **Intrinsic content size**
:---|:---
UIView and NSView |	No intrinsic content size
Sliders |	Defines only the width (iOS) Defines the width, the height, or the both - depending on the slider’s type (OS X)
Labels, buttons, switches, and text fields	| Defines both the height and the width
Text views and image views	| Intrinsic content size can vary

Неявные констрейнты представлений имеет свои приоритеты. 

Для вью, созданных в IB будут следующие приоритеты.
Name | Horizontal hugging |	Vertical hugging |	Horizontal resistance |	Vertical resistance
:---|:---|:---|:---|:---
Label |	251 | 251 |	750 | 750
Text Field | 250 |	250 | 750 | 750


**Для label, созданных в коде horizontal hugging priority = 250.**


* * *

## Size that fits
В отличии от intrinsicContentSize который сообщает о размере с учетом всех вложенных в нее элементов, sizeThatFits: говорит о том какого размера должна быть супервью чтобы его вместить.  



## Content hugging
Устанавливает приоритет, с которым вью будет препятствовать увеличению своего размера относительно увеличения своего контента. В данном примере вью по размерам обхватывает свой контент, не больше, не меньше. Если кто-то пытается растянуть наше вью, ContentHuggingPriority показывает то, на сколько сильно вью не хочет увеличиваться. Если его приоритет будет выше, чем констрейт который хочет его растянуть, то оно будет препятствовать этому. 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/820d543caf59444fbd5104e04ce15bfe.png?raw=true"> 


## Content Compression Resistance
Свойсво ContentCompressionResistancePriority описывает приоритет, с которым вью будет препятствовать уменьшению своего размера. Если какието констрейнты будут уменьшать размер вью и их приоритет будет ниже чем ContentCompressionResistancePriority, то вью будет сопротивлятся этому. 

Эти приоритеты бывают как горизонтальные так и вертикальные.
```objc
[textField setContentCompressionResistancePriority:500 forAxis:UILayoutConstraintAxisHorizontal];
[textField setContentHuggingPriority:500 forAxis:UILayoutConstraintAxisVertical];
```

* * *
# Create constraints in code

## Constraint constructor
Листинг создания констрейнта через конструктор: 
```objc
NSLayoutConstraint *constraint = [NSLayoutConstraint 					  		
	constraintWithItem:redSubview
	attribute:NSLayoutAttributeWidth
	relatedBy:NSLayoutRelationEqual
	toItem:blueSubview
	attribute:NSLayoutAttributeWidth
	multiplier:1
	constant:0];
[constraint setActive:true];
```
Такой констрейт по умолчанию не активный. Вызов setActive: является обязательным


## Visual Format Language (VFL)
Позволяет создавать сразу несколько констрейнтов используя строки. В строке прописываются отношения наших вью, и на основе строки генерируются строки. Листинг с примером создания двух групп констрейнтов: 
```
NSDictionary *views = @{ @"redSubview": redSubview, @"blueSubview": blueSubview };
NSString *verticalConstraintsFormat = @"V:|-24-[redSubview]-24-[blueSubview(==redSubview)]-24-|";
NSString *horizontalConstraintsFormat = @"H:|-24-[redSubview(==blueSubview)]-24-|";
NSArray *verticalConstraints = [NSLayoutConstraint constraintsWithVisualFormat:verticalConstraintsFormat
                                                                       options:NSLayoutFormatAlignAllCenterX
                                                                       metrics:nil views:views];
NSArray *horizontalConstraints = [NSLayoutConstraint 
     constraintsWithVisualFormat:horizontalConstraintsFormat
                                                                          options:NSLayoutFormatAlignAllCenterX
[NSLayoutConstraint activateConstraints:verticalConstraints];
[NSLayoutConstraint activateConstraints:horizontalConstraints];
```

Не смотря на кажущуюся простоту и понятность такого подхода, в таком подходе очень легко сделать и трудно отследить ошибку. На реальных проектах, данный способ используется крайне редко. 



---
## Anchors

Наиболее часто используемый способ - использование т.н. anchors(якоря). У каждого атрибута есть свой якорь, между которым и заводятся отношения.  

```objc
[NSLayoutConstraint activateConstraints:@[
[redSubview.widthAnchor constraintEqualToAnchor:blueSubview.widthAnchor],
[redSubview.heightAnchor constraintEqualToAnchor:blueSubview.heightAnchor],
[redSubview.topAnchor constraintEqualToAnchor:self.view.topAnchor constant:24],
[blueSubview.topAnchor constraintEqualToAnchor:redSubview.bottomAnchor constant:24], 
[blueSubview.bottomAnchor constraintEqualToAnchor:self.view.bottomAnchor constant:-24],
[redSubview.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor constant:24], 
[redSubview.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor constant:-24],
[redSubview.centerXAnchor constraintEqualToAnchor:blueSubview.centerXAnchor]
]];
```
**Note!**
Перед тем как добавить любой констрейнт необходимо добавить сабвью в иерархию:
`[self.view addSubview:redSubview];`
и также отключить преобразование AutoresizingMask
`redSubview.translatesAutoresizingMaskIntoConstraints = false;`

* * *


# Stack View
UIStackView позволяет реализовать всю мощь Auto layout без использования констрэйнтов. По сути, это UIView, дочерние элементы которого упорядочены по горизонтали или вертикали. UIStackView упорядочивает элменты на основе следующих свойств:

- Axis
 Определяет ориентацию (Горизонтальная\вертикальная) 
- Distribution
 Определяет расположение элементов в текущей ориентации
- Alignment
 Определяет расположение перпендикулярно ориентации 
- Spacing
 Определяет отступы между соседними элементами


<img src="https://github.com/OrientCue/ios/blob/master/_resources/1f75bf6642864e988c5808f403d3d390.png?raw=true"> 


## Safe area
В iOS7 Apple представила такие свойства как topLayoutGuide и bottomLayoutGuide. Они принадлежали UIViewController которые устанавливали размер экрана, который не перекрывается другими представлениями такими как StatusBar, NavigationBar, ToolBar, TabBar. Размещая констрейнты в рамках этих свойств, можно было гарантировать что ваш контент не будет перекрыт ничем другим при любой ориентации или размере. Начиная с iOS11 на смену пришел Safe Area, это свойство относится к UIView. Главное отличие в том, что можно задавать не только верхние и нижние границы, а также боковые. Цель Safe Area - помочь разработчику разместить контент в видимой области интерфейса, и обезопасить его от наложений других представлений. 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/d7ce86f4bbb14bceae2e632c4383c896.png?raw=true"> 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/69089ea67a2c41a385ca24fd790c3e65.png?raw=true"> 


## Margins 
 
Также атрибуты положений могут указывать не только от Safe Area, а еще и от Margins. Margins — это отступы по краям от представления. Они обеспечивают визуальный зазор между контентом и грaницей представления. По умолчанию margins равно 8pt. 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/05554c884aec4967a6fd71ef35fcfe99.png?raw=true">

* * *


# Trait collection
Apple представила концепцию, основанную на сочетании Auto Layout и UITraitCollection.
Класс UITraitCollection описывает свойства интерфейса, такие как: коэффициент масштабирования, тип интерфейса и такое важное свойство как Size Classes. 

## Size Classes
Size Class определяет относительное пространство на дисплее по высоте или ширине. Существует два типа size class: 

- Horizontal size class 
- Vertical size class 

Любое свойство этого типа может иметь одно из трех значений: 

- Compact
 Говорит о том, что в контейнере мало места для отображения контента
- Regular
 В контейнере достаточно места для контента 
- Any (по умолчанию)
 Для всех случаев

Данные size class изменяются в зависимости от размера экрана и ориентации.  Например, на картинке видно, как у IPhone 5, при Landscape ориентации значение size class будет compact как по ширине, так и по высоте. А в портретной ориентации высота будет regular, а ширина так и останется compact. Исходя из этих size classes можно размещать контент нужным вам образом. 


<img src="https://github.com/OrientCue/ios/blob/master/_resources/97cd0e8e74604b02a3a862b99c9f837b.png?raw=true"> 

У всех IPad во всех ориентациях значение size class будет regular. Однако если войти в Split View Mode, значение size class может менятся. 

Свойства горизонтальных и вертикальных классов определены следующим образом: 
```
@property(nonatomic, readonly) UIUserInterfaceSizeClass horizontalSizeClass;
@property(nonatomic, readonly) UIUserInterfaceSizeClass verticalSizeClass;
```

Перечисление **UIUserInterfaceSizeClass** определено в следующем перечислении: 
```objc
typedef NS_ENUM(NSInteger, UIUserInterfaceSizeClass) {
    UIUserInterfaceSizeClassUnspecified = 0,
    UIUserInterfaceSizeClassCompact     = 1,
    UIUserInterfaceSizeClassRegular     = 2,
} API_AVAILABLE(ios(8.0));
```
* * *

## UITraitEnvironment protocol
Для того чтобы отследеть изменения у UITraitCollection имеются свойство протокола UITraitEnvironment. Этот протокол реализуют такие классы как UIScreen, UIViewController, UIPresentationController. Apple настоятельно не рекомендует переопределять этот протокол или добавлять свою реализацию. У этого протокола имеется свойство: 

```objc
- (void)traitCollectionDidChange:(UITraitCollection *)previousTraitCollection;
```

Это свойство вызывается каждый раз когда происходит какое либо изменения интерфейса. По умолчанию этот метод реализации не имеет и его можно смело переопределять. 

## UIContentContainer protocol
Другой способ отследить изменения в интерфейсе. Его реализуют UIViewController и UIPresentationController. Можно переопределить следующие методы: 
```objc
- (void)viewWillTransitionToSize:(CGSize)size   withTransitionCoordinator:(id<UIViewControllerTransitionCoordinator>)coordinator;
- (void)willTransitionToTraitCollection:(UITraitCollection *)newCollection withTransitionCoordinator:(id<UIViewControllerTransitionCoordinator>)coordinator;
```
При переопределении рекомендуется вызывать super метод. Метод viewWillTransitionToSize: удобно использовать на небольших девайсах, таких как IPhone5, когда при перевороте экрана size class остается неизменным, этот методы вызывается при любом изменении.

## Useful links 🤓

[Auto Layout Guide](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/index.html)

[Mysteries of Auto Layout](https://developer.apple.com/videos/play/wwdc2015/218/)
