#android/dependencies

### Coroutindes
[Теория](theory/Coroutines)  
[Практика](practice/Coroutines)  
#android/dependencies/coroutines
```groovy
def coroutinesVersion = '1.3.9'
implementation "org.jetbrains.kotlin:kotlinx-coroutines-core: $coroutinesVersion"
implementation "org.jetbrains.kotlin:kotlinx-coroutines-android: $coroutinesVersion"
```





-----------------------------------------------------------------------
-   🔄 **RecyclerView**
    
    ```groovy
    //RecycleView
    implementation 'androidx.recyclerview:recyclerview:1.2.1'
    implementation 'com.github.bumptech.glide:glide:4.13.1'
    implementation 'com.google.android.material:material:1.7.0-alpha01'
    ```
    
-   **Floating action button**
    
-   **Delegate adapter**
    
    ```kotlin
    // delegate adapter
    implementation 'com.hannesdorfmann:adapterdelegates4:4.3.2'
    ```
    
-   **Floating action button**
    
    ```kotlin
    //floating button
    implementation 'com.google.android.material:material:1.7.0-alpha01'
    implementation 'com.jakewharton.threetenabp:threetenabp:1.2.4'
    ```
    
-   **LayoutContainer**
    
    ```kotlin
    plugins {
        ...
        id 'kotlin-android-extensions'
    }
    
    android {
        ...
        androidExtensions {
            experimental = true
        }
    }
    ```
    
-   🔀 **view binding**
    
    ```groovy
    buildFeatures {
      viewBinding true
    }
    
    allprojects {
      repositories {
        mavenCentral()
      }
    }
    
    dependencies {
        implementation 'com.github.kirich1409:viewbindingpropertydelegate:1.5.3'
    
        // To use only without reflection variants of viewBinding
        implementation 'com.github.kirich1409:viewbindingpropertydelegate-noreflection:1.5.3'
    }
    ```
    
-   **Nav graph**
    
    -   build.gradle:project
        
        ```groovy
        buildscript {
            dependencies {
                classpath "androidx.navigation:navigation-safe-args-gradle-plugin:2.5.0"
            }
        }
        
        plugins {
        		...
            id 'androidx.navigation.safeargs.kotlin' version '2.5.0' apply false
        }
        ```
        
    -   build.gradle:module
        
        ```groovy
        plugins {
            id 'androidx.navigation.safeargs'
        }
        dependencies {
        	def nav_version = "2.5.0-alpha02"
            implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
            implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
            implementation "androidx.navigation:navigation-dynamic-features-fragment:$nav_version"
            androidTestImplementation "androidx.navigation:navigation-testing:$nav_version"
            implementation "androidx.navigation:navigation-compose:2.5.0-alpha02"
        )
        }
        ```
        
-   **LiveData**
    ```kotlin
    implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.3.1'
    implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.3.1'
    ```

-   **Retrofit**
    
    -   **Attansion!**
        
        Если рассмотривать пример с [omdb.com](http://omdb.com) там нужно добавлять xml security config
        
        ![[Pasted image 20220902092600.png]]
        
        После чего прописывать его в манифесте
        
       ![[Pasted image 20220902092632.png]]
        
    
    ```groovy
    //Retrofit
    def retrofitVersion = '2.9.0'
    implementation "com.squareup.retrofit2:retrofit:$retrofitVersion"
    implementation "com.squareup.retrofit2:converter-moshi:$retrofitVersion"
    
    implementation "com.squareup.okhttp3:logging-interceptor:4.8.0"
    ```
    
-   Moshi
   
    ```groovy
   id 'kotlin-kapt'
    
    def moshiVersion = '1.9.3'
    implementation "com.squareup.moshi:moshi:$moshiVersion"
    implementation "com.squareup.moshi:moshi-kotlin:$moshiVersion"
    kapt "com.squareup.moshi:moshi-kotlin-codegen:$moshiVersion"
    ```
    
-   **Glide**
    
    ```groovy
    implementation 'com.github.bumptech.glide:glide:4.13.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.13.0'
    ```
    
-   Material Design
    
    ```groovy
    implementation 'com.google.android.material:material:1.6.1'
    ```
    
-   Dagger Hilt

```groovy
	plugins {
    ...
	    id 'kotlin-android-extensions'
	}

	android {
	    ...
	    androidExtensions {
	        experimental = true
	    }
	}
```
