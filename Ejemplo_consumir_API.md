# Consumir APIs con Retrofit

**Retrofit** es una potente librería de Android para realizar solicitudes HTTP de forma sencilla y eficiente. Permite interactuar con APIs REST y procesar sus respuestas de manera automática, convirtiéndolas en objetos Kotlin. En aplicaciones que usan Jetpack Compose, Retrofit es una excelente opción para manejar datos de servidores y mostrarlos en las interfaces modernas y reactivas que Compose ofrece.

Para este ejemplo vamos a consumir Fake Store API, que esta en linea y se puede usar para propósitos de aprendizaje. 

## 1.- Crear nuevo Proyecto con el nombre ProductsApiApp
![image](https://github.com/user-attachments/assets/6654d129-565e-4a7f-85c6-922e00102356)

## 2.- Instalar dependencias
### 2.1 Agregar plugins KSP en el *build.gradle.kts* y luego hacer click en *Sync Now*
![image](https://github.com/user-attachments/assets/c5528646-89bf-4c3e-ab3a-bb98cd31af62)

### 2.2 Agregar dagger para inyección de dependencia en el archivo *build.gradle.kts* de nivel superior, haga click en *Sync Now*
```kotlin
id("com.google.dagger.hilt.android") version "2.46.1" apply false
```
### 2.3 agregar dependencias en la sección de dependencies
```kotlin
    //dependencias agregadas
    implementation("com.google.dagger:hilt-android:2.46.1")
    val nav_version = "2.8.0"
    implementation("androidx.navigation:navigation-compose:$nav_version")
    //retrofit
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    //Coil
    implementation("io.coil-kt:coil-compose:2.5.0")
    //hilt
    implementation("com.google.dagger:hilt-android:2.46.1")
    val room_version = "2.6.1"
    implementation("androidx.room:room-runtime:$room_version")
    annotationProcessor("androidx.room:room-compiler:$room_version")
    implementation("androidx.room:room-ktx:$room_version")
    ksp("androidx.room:room-compiler:$room_version")
    //viewmodel
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.6")
```

Recordar sincronizar el proyecto y ejecutar para verificar que todo este bien hasta el momento

## 3.- Crear estructura de packages de acuerdo a la siguiente imagen

![image](https://github.com/user-attachments/assets/46176b6d-8b4c-4aec-9c40-a9e99069979f)

## 4.- Crear clase ProductApplication dentro del package principal
```kotlin
@HiltAndroidApp
class ProductApplication : Application(){
}
```
## 4.- Definir entrada de la clase y configurar permisos en el archivo AndroidManifest.xml
En la sección de *Application* definir la entrada para la clase ProductApplication
```xml
android:name=".ProductApplication"
```
Definir permisos para acceder a Internet desde la aplicación antes del inicio de la sección Application
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
## 5.- Para utilizar inyección de dependencias anotamos la clase MainActivity con @AndroidEntryPoint
## 6.- Crear clase Constants que contenga un companion Object para definir constantes en el package *utils* 
```kotlin
class Constants {
    companion object{
        const val BASE_URL = "https://fakestoreapi.com/"
        const val ENDPOINT = "products"
        const val CUSTOM_BLACK = 0xFF2B2626
        const val CUSTOM_GREEN = 0xFF209B14
    }
}
```
## 7.- Crear definición de interface *ApiProduct* en el package data
La estructura de la interface debe quedar como el código siguiente, después se definirán los metodos cuando se cree el modelo
```kotlin
interface ApiProduct {
}
```
## 8.- En el package *di* crear objeto con el nombre AppModule, para la configuración de la inyección de dependencias 
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    //para inyectar dependencia primero se define el Singleton y despues el provider
    @Singleton
    @Provides
    fun providesRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    @Singleton
    @Provides
    fun providesApiProducts(retrofit: Retrofit): ApiProduct {
        return retrofit.create(ApiProduct::class.java)
    }
}
```
