## Coroutines
#android/practice/coroutines

[Теория](27.%20Coroutines.md)  
[Зависимости](Dependencies)  

### Создание простой корутины
**Корутина запущенная на главном потоке не блокирует поток**
```kotlin
class MyFragment : Fragment() {
	private val fragmentScope = CoroutineScope(Dispatchers.Main)

	override fun onViewCreated() {
		fragmentScope.launch {
			delay(1000)
			Timber.d("BasicCoroutineFragment launch-1")
		}
		
		fragmentScope.launch {
			while(true) {
				delay(100)
				Timber.d("BasicCoroutineFragment launch-2")
			}
		}
	}
}
```

### Смена диспечера
```kotlin
class MyFragment() : Fragment() {

	override fun onViewCreated() {
		CoroutineScope(Dispatchers.Main).launch {
			calculateNumber()
		}
	}
	

	private suspend fun calculateNumber() : Int {
		return withContext(Dispatchers.IO) {
			return 5
		}
		
	}

}
```
*withContext()* - позволяет сменить диспетчер выполнения для какого-то блока кода

### Смена потока
```kotlin
class BasicCoroutineFragment : Fragment() {
	priv ate val fragmentIOScope = CoroutineScope(Dispatchers.IO)

	fragmentIOScope.launch {
		(0..200).forEach {
			Timber.d("start from thread = ${Thread.currentThread().name}")
			delay(1000)
			Timber.d("finish from thread = ${Thread.currentThread().name}")
		}
	}
}
```

### Async
```kotlin
class BasicCoroutineFragment : Fragment() {
	
	private fun asyncExample() {
		CoroutineScope(Dispatchers.IO).launch {
			val deferredResult = async {
				calculateNuamber()
			}
		
			val result = defferedResult.await()
		}
		
	}
	
	private suspend fun calculateNumber() : Int {
		return withContext(Dispatchers.IO) {
			BigInteger.probablePrime(2000, Random.asJavaRandom())
		}
		
	}
}
```
*Deferred\<out T\>* : Job - объект, который может вернуть результат по окончанию выполнения корутины

### Параллельный запуск корутин
```kotlin
class BasicCoroutineFragment : Fragment() {
	
	private fun asyncExample() {
		CoroutineScope(Dispatchers.IO).launch {
			(0 until 2).map{
				async {
					calculateNumber()
				}
			}
			.map { deffered ->
				deffered.await()
			}
		}
	}
}
```

### Обработка ошибок
```kotlin
class ErrorCancelFragment : Fragment(R.layout.fragment_error_cancel) { 
	private val errorHandler = CoroutineExceptionHandler { coroutineContext, throwable -> 
		Log.e("ErrorCancelFragment", "error from coroutineExceptionHandler", throwable) 
	} 
	private val scope: CoroutineScope = CoroutineScope(SupervisorJob() + Dispatchers.Main + errorHandler) 

	override fun onCreate(savedInstanceState: Bundle?) {
		super.onCreate(savedInstanceState) scope.launch { 
			error("test exception") 
		} 
		
		scope.launch { 
			delay(3000) 
			error("test exception") 
		} 
			
		scope.launch { 
			var i = 0 
			while(true) { 
				delay(500) 
				Log.d("ErrorCancelFragment", "log $i" 
			} 
		} 
	} 
}

```
Блок scope.launch{...} нельзя просто так обернуть в try-catch потому что, всё что делается в scope.launch{...} - происходит запуск корутины. Запуск произошёл и выполнение мдёт дальше. И если в процессе ывполнения корутины возникнет ошибка, её некому будет обработать.

Для обработки ошибки можно перенести блок try - catch внутрь корутины.

Второй способ - использовать ExceptionHandler

Для того чтобы при возникновении ошибки в одной из дочерних корутин, остальные сестринские корутины не отменялись, нужно при создании scope явно указать специальный Job - _SupervisorJob()_

**Если не использовать _SupervisorJob,_ то после возникновения ошибки в одной из корутин, scope будет отменеён, и на нём уже нельзя будет запустить новых корутин.**

### Отмена корутины
```kotlin
class ErrorCancelFragment : Fragment(R.layout.fragment_error_cancel) {
	
	private val errorHandler = CoroutineExceptionHandler { coroutineContext, throwable -> 
		Log.e("ErrorCancelFragment", "error from coroutineExceptionHandler", throwable)
	}
	
	private val scope: CoroutineScope = CoroutineScope(SupervisorJob() + Dispatchers.Main + errorHandler)
	 
	private fun nonCancellableExample(){
		scope.launch {
			var i = 0
			withContext(Dispatchers.Default){ 
				while(isActive) { //<- поддержка остановки корутины, проверяем, если корутина активна, продолжаем выполнение 
					Tread.sleep(500) 
					Log.d("ErrorCancelFragment", "log $i" } 
				}
			}
		}
	}

	override fun onActivityCreated(savedInstanceState: Bundle?) { 
		super.onActivityCreated(savedInstanceState) 
		cancelButton.setOnClickListener { //scope.cancel() <- не лучший вариант т.к. после вызова cancel scope использовать уже нельзя. 
			scope.coroutineContext.cancelChildren() 
		} 
	}
	
}

```
Важно! Если код который выполняется внутри корутины не умеет реагировать на изменение состояния корутины, то такой код будет продолжать свою работу, даже после отмены корутины. Необходимо в код корутины добавлять логику, которая будет прерывать выполнение когда корутина перестанет быть активной.

Можно так же вместо Thread.sleep() использовать delay

```kotlin
class ErrorCancelFragment : Fragment(R.layout.fragment_error_cancel) {  
    private fun nonCancellableExample(){  
        scope.launch {  
            var i = 0  
            withContext(Dispatchers.Default){  
                while(true) { //<- поддержка остановки корутины, проверяем, если корутина активна, продолжаем выполнение   
					delay(500)  
	                Log.d("ErrorCancelFragment", "log $i"  
                }  
            }  
  
        }    
    }  
}
```

В этом случае, после отмены корутины, так же произойдёт завершения выполнения кода.

Это произойдёт потому, что при вызове delay происходит проверка активности корутины. Внутри delay() используется _**suspendCancelableCoroutine**_ - это функция, создаёт suspend функцию которая реагирует на сигнал отмены.

В корутинах отмена реализована на механизме ошибок. Когда необходимо отменить корутину, то в ней возникает специальное исключение - **CancelableException**

Это исключение не распространяется вверх как обычное исключение. Это исключение распространяется на дочерние корутины которые тоже отменяются.

```kotlin
	private fun cancelableExample() {  
    scope.launch {  
        suspendCancelableCoroutine<Unit> { continuation ->  
            continuation.invokeOnCancellation {  
                //когда корутина будет отменена, выполнится этот блок кода  
                //например отменить запрос и закрыть соединение.            }  
            var i = 0  
            withContext(Dispatchers.Default){  
                while(true) { //<- поддержка остановки корутины, проверяем, если корутина активна, продолжаем выполнение   
					delay(500)  
                    Log.d("ErrorCancelFragment", "log $i"  
                }  
            }  
        }    
    }
}
```

другим способом отменить блок кода в корутине является метод **yield**

Данный метод проверяет, является ли корутина активной, если нет, то он выбросит исключение.

```kotlin
private fun cancellableWithYieldExample() {  
    scope.launch {  
        var i = 0  
        withContext(Dispatchers.Default){  
            while(true) { //<- поддержка остановки корутины, проверяем, если корутина активна, продолжаем выполнение   
			suspend  
                delay(500)  
                Log.d("ErrorCancelFragment", "log $i"  
            }  
        }  
    }
}
```

### Корутины в Android
#### Fragment
```kotlin
class RepositoryListFragment: Fragment(R.layout.fragment_repositories_list) {  
    ...  
  
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {  
        super.onViewCreated(view, savedInstanceState)  
  
        // корутина будет отменена, когда фрагмент закончит своё существование  
        lifecycleScope.launch {  
  
        }  
        
        // корутина будет отменена, при уничтожении view  
        viewLifecycleOwner.lifecycleScope.launch {  
  
        }  
        
        initList()  
        bindViewModel()  
    }  
}
```
Фрагмент является LifecycleOwner-ом. У lifecycle owner в библиотеке kotlin-ktx определено extension - св-во lifecycleScope

**lifecycleScope** - CoroutineScope который привязан к жц lifecycle owner - если используется во fragment то привязан к жц фрагмента.

_lifecycleScope.launch_ - корутина будет отменена, когда фрагмент закончит своё существование. (вызовется метод onDestroy)

_viewLifecycleOwner_*.*_lifecycleScope.launch_ - корутина будет отменена, при уничтожении view

Оба варианта не обрабатывают ошибки, это должен делать пользователь. Обработчик ошибок можно передать в метод _launch()_

#### ViewModel
```kotlin
class GithubRepoViewModel : ViewModel() {  
    private val repository = GithubRepoRepository()  
    private val githubRepositoriesLiveData = MutableLiveData<List<GithubRepoEntity>>()  
    private val isLoadingLiveData = MutableLiveData<Boolean>()  
  
    val githubRepoList: LiveData<List<GithubRepoEntity>>  
        get() = githubRepositoriesLiveData  
  
    val isLoading: LiveData<Boolean>  
        get() = isLoadingLiveData  
  
    fun searchGithubRepositories() {  
        viewModelScope.launch {  
            isLoadingLiveData.postValue(true)  
            try {  
                val repos = repository.searchUserRepositories()  
                githubRepositoriesLiveData.postValue(repos)  
            } catch() {  
                githubRepositoriesLiveData.postValue(emptyList())  
            } finally {  
                isLoadingLiveData.postValue(false)  
            }  
        }  
    }  
}
```
viewModelScope - корутина будет отменена в тот момент, когда у viewModel будет вызван метод _onCleared()_

При работе с корутинами можно избавиться от callBack
```kotlin
class GithubRepoRepository {  
    suspend fun searchUserRepositories(): List<GithubRepoEntity> {  
        return Network.githubApi.getRepoInfo().ietms  
    }  
  
    //Если используемая библиотека не поддерживает смену потоков  
    suspend fun searchUserRepositories(): List<GithubRepoEntity> {  
        return withContext(Dispatchers.IO) {  
            Network.githubApi.getRepoInfo().ietms  
        }  
    }  
}


//Retrofit позволяетиспользовать suspend ф-ии в своих запросах т.к. поддерживает работу с корутинами  
//Запрос выполнится на фоновом потоке на ThreadPool который находится внутри Retrofit.  
interface GithubApi {  
    @GET("user/repos")  
    suspend fun getRepoInfo(): List<GithubRepoEntity>  
}
```
