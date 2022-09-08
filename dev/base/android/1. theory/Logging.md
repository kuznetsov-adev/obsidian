#android/logging
Иерархия уровней логирования (от низкого приоритета к высокому)

**VERBOSE** - все сообщения
**DEBUG** - отладочные сообщения
**INFO** - информационные сообщения
**WARNING**
**ERROR** - ошибки приложения
**ASSERT** - критические ошибки которые не должны были случиться

Logcat → настройка цветов  
		file → settings → Editor → Color Scheme → Android Logcat

Обертка для логера, которая исключает логирование в релизной версии приложения
```kotlin
object DebugLogger {
	fun d (tag: String, message: String) { 
		if(BuildConfig.DEBUG) { 
			Log.d(tag, message) 
		} 
	} 
}
```


Подключить библиотеку [Timber](https://github.com/JakeWharton/timber).

```groovy
implementation 'com.jakewharton.timber:timber:4.7.1'
```
