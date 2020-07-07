
 **[Back](https://orientcue.github.io/ios/ "Table of Content")**

#  Multithreading. Part 2. NSOperation. NSOperationQueue.

<!-- TOC -->
- [NSOperation](#nsoperation)
- [Create NSBlockOperation instance](#create-nsblockoperation-instance)
- [Add multiple blocks into NSBlockOperation instance](#add-multiple-blocks-into-nsblockoperation-instance)
- [Track completion of all tasks inside NSBlockOperation instance](#track-completion-of-all-tasks-inside-nsblockoperation-instance)
- [NSOperation subclassing](#nsoperation-subclassing)
- [NSOperationQueue](#nsoperationqueue)
- [Create queue](#create-queue)
- [Quality of Service](#quality-of-service)
- [Add operation into a queue](#add-operation-into-a-queue)
- [Maximum number of operations](#maximum-number-of-operations)
- [Waiting for completion](#waiting-for-completion)
- [Operation Dependencies](#operation-dependencies)
- [Cancelling Operations](#cancelling-operations)
- [Asynchronous operations](#asynchronous-operations)
- [Useful materials 🤓](#useful-materials-)
<!-- /TOC -->

## NSOperation 

**NSOperation** — это абстрактный класс, содержащий в себе логику и данные ассоциируемые с единой задачей. Т.е. в GCD задачи представляли собой обычно блоки кода, здесь задача — это класс. Как и GCD, NSOperation позволяют вам запускать какие-то задачи на разных очередях, а соответственно и разных потоках. Однако NSOperation дают вам больше контроля над запущенными задачами. 

**NSOperation pros**

NSOperation построен на основе GCD, но в NSOperation были добавлены дополнительные возможности, которые дают плюсы над GCD. Это:

- Установка зависимостей (Dependencies)
- Отмена (Easy cancellation)
- Наличие KVO свойств (KVO-compliant properties)
- Наследование (Inheritance) 

**NSOperation states**

У операций есть свой жизненный цикл, состоящий из следующих состояний:

- **Pending** 
- **Ready** 
- **Executing** 
- **Finished** 
- **Cancelled**

<img src="https://github.com/OrientCue/ios/blob/master/_resources/d8ca715e55224af09176d7d9559895bc.png?raw=true">

При создании экземпляра класса, операция находится в состоянии **Pending**.
Когда операция подготовлена, и готова к запуску, она переходит в состояние **Ready**. За это состояние отвечает вычисляемое свойство ready, оно возвращает YES если задача готова к запуску прямо сейчас. или возвращает NO если задача зависит от каких-то других операций которые еще не завершены. В большинстве случаев вам не нужно управлять этим свойством. Однако если в вашем приложении значение этого свойства зависит от каких-то иных факторов, отличных от завершенности операции от которых зависит текущая задача, то можно переопределить геттер чтобы поместить туда свою логику определения готовности операции.
В какой-то момент вы можете вызвать метода **start** у операции, этим действием вы переводите операцию в состояние **Executing**. Этому состоянию соответствует вычисляемое свойство executing, это свойство возвращает YES если операция находится в процессе выполнения, либо NO если операция не выполняется. Если вы переопределите метод start в вашем классе наследнике NSOperation, вам также следует переопределить геттер isExecuting и сгенерировать KVO нотификации для изменения executing состояния операции.
Если приложением операции был вызван метод cancel, то операция переходит в состояние **Cancelled**, данному состоянию соответствует вычисляемое свойство cancelled. Данное свойство возвращает YES, если была запрошена отмена операции. В данное состояние операция может перейти из любого ранее описанного состояния. 
Если же операция не была отменена, то после состояния **Executing** операция переходит в состояние **Finished**. Данному состоянию соответствует вычисляемое свойство finished, оно возвращает YES, если операция завершена, иначе NO. Операция не удаляется из очереди пока свойство finished не возвращает YES. Если вы переопределите метод start в вашем классе наследнике NSOperation, вам также следует переопределить геттер isFinished и сгенерировать KVO нотификации для изменения executing состояния операции.

## Create NSBlockOperation instance
Как сказано выше, NSOperation это абстрактный класс. Чтобы работать с этими операциями, как правило необходимо создать дочерний класс. Иногда такое решение бывает избыточным и вам нужно быстро создать операция для простой задачи. Для этого можно использовать класс наследник NSOperation который поставляется вместе Foundation - **NSBlockOperation**. В инициализатор просто передается блок кода. В отличии от GCD, здесь есть все приемущества NSOperation - KVO нотификации, зависимости, и.т.д. Чтобы начать выполнение можно просто вызвать метод start. 

```objc
NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
    // Some task
}];
[blockOperation start];
```


**ВАЖНО
!!!NSBlockOperation запускает задачу на дефолтной глобальной конкурентной очереди.!!!** 

## Add multiple blocks into NSBlockOperation instance

Хотя из названия класса это не так очевидно, но в классе NSBlockOperation можно запускать несколько блоков для выполнения. Все блоки будут выполнены на конкурентной очереди после вызова метода start. Для добавления дополнительных блоков в экземпляр NSBlockOperation, используем метод **addExecutionBlock**.

```objc
NSBlockOperation *blockOperation = [NSBlockOperation new];
[blockOperation addExecutionBlock:^{
    // Task 1
}];
[blockOperation addExecutionBlock:^{
    // Task 2
}];
[blockOperation start];
```

## Track completion of all tasks inside NSBlockOperation instance

NSOperation, как и DispatchGroup может отслеживать выполнение операций внутри себя. Как только все задачи завершают свою работу, экземпляр NSOperation помечает себя как завершенный, и вызывает блок completionBlock. 

```objc
blockOperation.completionBlock = ^{
    // Some completion action
};
```

## NSOperation subclassing
Класс **NSBlockOperation** хорош для выполнения простых задач. Но если вам надо выполнить сложные задачи, инкапсулировать логику для работы задачи, переиспользовать задачу, то вам необходимо создать свой подкласс NSOperation. 

```objc
@interface CustomOperation : NSOperation
@end
```

При создании подкласса, обычно необходимо переопределить метод main: 

```objc
- (void)main {
   //Some work
}
```

Внутри которого вы размещаете код необходимый для выполнения задачи. Также можно создать кастомные инициализаторы, геттеры, сеттеры для доступа к вашим данным. Однако, если вы их определяете, вы должны убедится, что они безопасны для доступа из разных потоков. 
Для старта кастомной операции необходимо создать экземпляр и вызвать метод start. 

```objc
CustomOperation *operation = [CustomOperation new];
[operation start];
```

**ВАЖНО
!!! Если вы вызываете метод start, операция будет выполнена синхронно на текущей очереди. В процессе выполнения операции, геттер isExecuting возвращает YES. Прямой вызов метода start может привести к RunTime ошибке, если операция еще не готова к выполнению. Поэтому метод старт можно вызывать только если геттер isReady возвращает YES.!!!** 

---

## NSOperationQueue

**NSOperationQueue** - класс, представляющий очередь для запуска операций. По умолчанию, NSOperationQueue является конкурентной очередью и ассоциирована с таким количество потоков, которое ей может предоставить система. Чтобы получить все бенефиты конкурентной очереди, нам достаточно просто добавить NSOperation в очередь. После того как мы добавили операцию в очередь, NSOperationQueue запускает операцию, как только она переходит в состояние ready. 
Как только мы добавили операцию в очередь, мы не может добавить этот же экземпляр в другую очередь, экземпляр NSOperation может быть выполнен только один раз. Операция остается в очереди пока она не будет завершена, либо отменена. Вы не можете напрямую удалить операцию из очереди после того как операция была добавлена. 
Очередь увеличивает счетчик ссылок операции пока она не была завершена. Сама очередь существует пока все операции в ней не были выполнены. Приостановка работы очередей, в которых есть невыполненные операции может привести к утечкам памяти. 

## Create queue
Чтобы создать очередь достаточно воспользоваться инициализатором по умолчанию.

`NSOperationQueue *queue = [NSOperationQueue new];`

## Quality of Service
Вы можете установить значение QoS как для очередей, так и для операций, можно добавлять в очередь операции с разным значение QoS и они будут выполнятся в соответствии с приоритетом. 

```objc
queue.qualityOfService = NSQualityOfServiceUtility;
operation.qualityOfService = NSQualityOfServiceUtility;
```

Значение по умолчанию для очереди - **background**. 

### Add operation into a queue

Все что нужно для добавления операции в очередь это вызвать метод addOperation: дальнейшую работу очередь проделает самостоятельно. 

`[queue addOperation:operation];`

## Maximum number of operations
NSOperationQueue это конкурентная очередь, которая может выполнять столько операций сколько позволяют ресурсы. Иногда необходимо ограничить количество одновременно выполняемых задач. Для это желаемое значение устанавливается в свойстве maxConcurrentOperationCount. Для того чтобы сделать очередь серийной, значение необходимо установить в 1. 

```objc
queue.maxConcurrentOperationCount = 3;
```

##  Waiting for completion
Чтобы заблокировать текущую очередь на время пока все операции в очереди выполняются, т.е. сделать запуск синхронным, используем метод waitUntilAllOperationsAreFinished: 

```objc
[queue waitUntilAllOperationsAreFinished];
```

## Operation Dependencies

NSOperation предоставляет устанавливать зависимости между операциями. Зависимости между операциями дают нам следующие преимущества:

1.	Мы точно знаем, что зависимая операция не начнет своего выполнения пока операция от которой не зависит не будет завершена. 
2.	Зависимости дают понятный способ передачи данных из первой операции во вторую.

Для добавления зависимостей используем метод **addDependency**: 

```objc
[operation2 addDependency:operation1];
```

Вызываем его у зависимой операции, в параметры передаются операцию, от которой зависит. В листинге operation2 запустится только после того, как будет завершена operation1. Если по какой-то причине нужно удалить зависимость, воспользуемся методом removeDependency:

```objc
[operation2 removeDependency:operation1];
```

У NSOperation есть ридонли свойство, которое содержит массив зависимостей

```objc
@property (readonly, copy) NSArray<NSOperation *> *dependencies;
```

## Cancelling Operations

NSOperation имеет возможность отмены операции. Для отмены операции необходимо вызвать метод cancel: 

```objc
[operation cancel];
```

После этого свойство isCancelled будет возвращать YES. Это все, важно понимать, что очередь не остановит операцию автоматически. Вся работа по отмене операции лежит на программисте. Такие задачи как освобождение памяти, очистка загруженных данных и.т.д вам необходимо реализовать самостоятельно.

```objc
- (void)main {
    // Some hard work part 1
    if (self.isCancelled) { return; } // Some hard work part 2
    if (self.isCancelled) { return; }
    // Some hard work part 3
    // Task is completed
}
```

В случае если вам необходимо остановить все операции в очереди, вам необходимо вызвать метод очереди **cancelAllOperations**:

```objc
[queue cancelAllOperations];
```

Данный метод очереди вызовет cancel: у всех своих операций. 

---

## Asynchronous operations

Как было сказано выше, все операции, по умолчанию, синхронные. Если мы запускаем операцию вручную, с помощью метода start:, то операция запускается синхронно на текущей очереди. Если же мы добавляем операцию в NSOperationQueue, то он самостоятельно запускает задачу асинхронно на другом потоке. Но бывают задачи, когда нам необходимо запускать задачу вручную, не блокируя текущий поток. Для этого мы можем реализовать задачу как асинхронную. Если мы просто внутри операции задачу вынесем на другой поток и запустим операцию методом start: то метод может вернуть управление раньше времени, и задача раньше времени будет считаться завершенной. Поэтому, когда мы делаем операцию асинхронной, от нас требуется написать чуть больше кода чем это требуется обычно. Потому что мы должны самостоятельно мониторить все состояния задачи и уведомлять об их изменении используя KVO нотификации. 

```objc

//  AsyncOperation.h

#import <Foundation/Foundation.h>

@interface AsyncOperation : NSOperation
- (void)finishOperation;
@end

//  AsyncOperation.m

#import "AsyncOperation.h"

@interface AsyncOperation () {
  BOOL executing;
  BOOL finished;
}

@end

@implementation AsyncOperation

- (BOOL)isAsynchronous {
  return YES;
}

- (BOOL)isExecuting {
  return executing;
}

- (BOOL)isFinished {
  return finished;
}

- (void)start {
  if (self.isCancelled) {
    [self willChangeValueForKey:@"isFinished"];
    finished = YES;
    [self didChangeValueForKey:@"isFinished"];
    return; 
  }
  [self main];
  [self willChangeValueForKey:@"isExecuting"];
  executing = YES;
  [self didChangeValueForKey:@"isExecuting"];

}

- (void)finishOperation {
  [self willChangeValueForKey:@"isExecuting"];
  [self willChangeValueForKey:@"isFinished"];
  executing = NO;
  finished = YES;
  [self didChangeValueForKey:@"isExecuting"];
  [self didChangeValueForKey:@"isFinished"];
}
@end
```

---

## Useful materials 🤓

[Apple. NSOperation](https://developer.apple.com/documentation/foundation/nsoperation)

[Apple. NSOperationQueue](https://developer.apple.com/documentation/foundation/nsoperationqueue)

[WWDC 2015. Advanced NSOperations](https://developer.apple.com/videos/play/wwdc2015/226/)



