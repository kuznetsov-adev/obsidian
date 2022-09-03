#android/async #android/bestpractice
Решил написать о некоторых вещах, которых, по моему мнению, стоит и не стоит избегать при использовании корутин Kotlin.

## Оборачивайте асинхронные вызовы в coroutineScope или используйте SupervisorJob для обработки исключений

![[cros.jpeg|20]] Если в блоке `async` может произойти исключение, не полагайтесь на блок `try/catch`.

```kotlin
val job: Job = Job()
val scope = CoroutineScope(Dispatchers.Default + job)

// may throw Exception
fun doWork(): Deferred<String> = scope.async { ... }   // (1)

fun loadData() = scope.launch {
    try {
        doWork().await()                               // (2)
    } catch (e: Exception) { ... }
}
```

В приведённом выше примере функция `doWork` запускает новую корутину (1), которая может выбросить необработанное исключение. Если вы попытаетесь обернуть `doWork` блоком `try/catch` (2), приложение всё равно упадёт.

Это происходит потому, что отказ любого дочернего компонента job приводит к немедленному отказу его родителя.

![[tick.jpeg|20]] Один из способов избежать ошибки — использовать [`SupervisorJob`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html) (1).

> _Сбой или отмена выполнения дочернего компонента не приведёт к сбою родителя и не повлияет на другие компоненты._

```kotlin
val job = SupervisorJob()                               // (1)
val scope = CoroutineScope(Dispatchers.Default + job)

// may throw Exception
fun doWork(): Deferred<String> = scope.async { ... }

fun loadData() = scope.launch {
    try {
        doWork().await()
    } catch (e: Exception) { ... }
}
```

_Примечание_: это будет работать, только если вы явно запустите свой асинхронный вызов в рамках корутины с `SupervisorJob`. Таким образом, приведённый ниже код всё равно приведёт к сбою вашего приложения, потому что `async` запускается в рамках родительской корутины (1).

```kotlin
val job = SupervisorJob()                               
val scope = CoroutineScope(Dispatchers.Default + job)

fun loadData() = scope.launch {
    try {
        async {                                         // (1)
            // may throw Exception 
        }.await()
    } catch (e: Exception) { ... }
}
```

![[tick.jpeg|20]] Другой способ избежать сбоя, который является более предпочтительным, заключается в том, чтобы обернуть `async` в `coroutineScope` (1). Теперь, когда исключение происходит внутри `async`, оно отменяет все другие корутины, созданные в этой области, не касаясь при этом внешней области. (2)

```kotlin
val job = SupervisorJob()                               
val scope = CoroutineScope(Dispatchers.Default + job)

// may throw Exception
fun doWork(): Deferred<String> = coroutineScope {     // (1)
    async { ... }
}

fun loadData() = scope.launch {                       // (2)
    try {
        doWork().await()
    } catch (e: Exception) { ... }
}
```

Кроме того, вы можете обрабатывать исключения внутри блока `async`.

## Используйте главный диспетчер для корневых корутин

![[cros.jpeg|20]] Если вам нужно выполнить фоновую работу и обновить пользовательский интерфейс внутри своей корневой корутины, запускайте её с помощью главного диспетчера.

```kotlin
val scope = CoroutineScope(Dispatchers.Default)          // (1)

fun login() = scope.launch {
    withContext(Dispatcher.Main) { view.showLoading() }  // (2)  
    networkClient.login(...)
    withContext(Dispatcher.Main) { view.hideLoading() }  // (2)
}
```

В приведённом выше примере мы запускаем корневую корутину, используя в `CoroutineScope` диспетчер по умолчанию (1). При таком подходе каждый раз, когда нам нужно будет обновлять пользовательский интерфейс, мы будем должны переключать контекст (2).

![[tick.jpeg|20]] В большинстве случаев предпочтительнее создать `CoroutineScope` сразу с главным диспетчером, что приведёт к упрощению кода и менее явному переключению контекста.

```kotlin
val scope = CoroutineScope(Dispatchers.Main)

fun login() = scope.launch {
    view.showLoading()    
    withContext(Dispatcher.IO) { networkClient.login(...) }
    view.hideLoading()
}
```

## Избегайте использования ненужных async/await

![[cros.jpeg|20]] Если вы используете функцию `async` и сразу же вызываете `await`, то вам следует прекратить это делать.

```kotlin
launch {
    val data = async(Dispatchers.Default) { /* code */ }.await()
}
``` 

![[tick.jpeg|20]] Если вы хотите переключить контекст корутины и немедленно приостановить родительскую корутину, то `withContext` — это самый предпочтительный для этого способ.

```kotlin
launch {
    val data = withContext(Dispatchers.Default) { /* code */ }
}
```

С точки зрения производительности это не такая большая проблема (даже если учесть, что `async` создаёт новую корутину для выполнения работы), но семантически `async` подразумевает, что вы хотите запустить несколько корутин в фоновом режиме и только потом ждать их.

## Избегайте отмены job

![[cros.jpeg|20]] Если вам нужно отменить корутину, не отменяйте job.

```kotlin
class WorkManager {

    val job = SupervisorJob()
    val scope = CoroutineScope(Dispatchers.Default + job)

    fun doWork1() {
        scope.launch { /* do work */ }
    }

    fun doWork2() {
        scope.launch { /* do work */ }
    }

    fun cancelAllWork() {
        job.cancel()
    }
}

fun main() {
    val workManager = WorkManager()

    workManager.doWork1()
    workManager.doWork2()
    workManager.cancelAllWork()
    workManager.doWork1() // (1)
}
```

Проблема с приведённым выше кодом заключается в том, что когда мы отменяем job, мы переводим его в _завершённое_ состояние. Корутины, запущенные в рамках _завершённого_ job, выполнены не будут (1).

![[tick.jpeg|20]] Если вы хотите отменить все корутины в определённой области, вы можете использовать функцию [`cancelChildren`](https://github.com/Kotlin/kotlinx.coroutines/issues/787). Кроме того, хорошей практикой является предоставление возможности отмены отдельных job (2).

```kotlin
class WorkManager {

    val job = SupervisorJob()
    val scope = CoroutineScope(Dispatchers.Default + job)

    fun doWork1(): Job = scope.launch { /* do work */ } // (2)

    fun doWork2(): Job = scope.launch { /* do work */ } // (2)

    fun cancelAllWork() {
        scope.coroutineContext.cancelChildren()         // (1)                             
    }
}

fun main() {
    val workManager = WorkManager()

    workManager.doWork1()
    workManager.doWork2()
    workManager.cancelAllWork()
    workManager.doWork1()
}
```

## Избегайте написания функции приостановки, используя неявный диспетчер

![[cros.jpeg|20]] Не пишите функцию `suspend`, выполнение которой будет зависеть от определенного диспетчера корутин.

```kotlin
suspend fun login(): Result {
    view.showLoading()
    val result = withContext(Dispatcher.IO) {  
        someBlockingCall() 
    }
    view.hideLoading()

    return result
}
```

В приведённом выше примере функция входа в систему является функцией приостановки и она завершится сбоем, если вы запустите её из корутины, которая не будет использовать главный диспетчер.

```kotlin
launch(Dispatcher.Main) {     // (1) всё в порядке
    val loginResult = login()
    ...
}

launch(Dispatcher.Default) {  // (2) возникнет ошибка
    val loginResult = login()
    ...
}
```

> _CalledFromWrongThreadException: только исходный поток, создавший иерархию View-компонентов, имеет к ним доступ._

![[tick.jpeg|20]] Создайте свою функцию приостановки таким образом, чтобы её можно было выполнять из любого диспетчера корутин.

```kotlin
suspend fun login(): Result = withContext(Dispatcher.Main) {
    view.showLoading()
    val result = withContext(Dispatcher.IO) {  
        someBlockingCall() 
    }
    view.hideLoading()

return result
}
```

Теперь мы можем вызвать нашу функцию входа в систему из любого диспетчера.

```kotlin
launch(Dispatcher.Main) {     // (1) no crash
    val loginResult = login()
    ...
}

launch(Dispatcher.Default) {  // (2) no crash ether
    val loginResult = login()
    ...
}
```

## Избегайте использования глобальной области видимости

![[cros.jpeg|20]] Если вы используете [`GlobalScope`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/) везде в своём Android-приложении, вам следует прекратить это делать.

```kotlin
GlobalScope.launch {
    // code
}
```

> _Глобальная область видимости используется для запуска корутин верхнего уровня, которые работают в течение всего времени жизни приложения и не отменяются раньше времени._  
>   
> _Код приложения обычно должен использовать определяемый приложением [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html), поэтому использование [async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) или [launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) в [GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.md) крайне не рекомендуется._

![[tick.jpeg|20]] В Android корутина может быть легко ограничена жизненным циклом Activity, Fragment, View или ViewModel.

```kotlin
class MainActivity : AppCompatActivity(), CoroutineScope {

    private val job = SupervisorJob()

    override val coroutineContext: CoroutineContext
        get() = Dispatchers.Main + job

    override fun onDestroy() {
        super.onDestroy()
        coroutineContext.cancelChildren()
    }

    fun loadData() = launch {
        // code
    }
}
```