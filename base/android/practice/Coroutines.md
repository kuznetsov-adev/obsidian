## Coroutines
#android/practice/coroutines

[Теория](theory/Coroutines)  
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
