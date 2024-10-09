# Ejemplo de CRUD para almacenamiento local con ROOM
## 1.- Crear nuevo proyecto con JetPack Compose
### 1.1 Seleccione Empty Activity de Jetpack Compose como se muestra en la imágen
![image](https://github.com/user-attachments/assets/48a10302-3e83-41a0-a13f-0446dc532a35)
### 1.2 Coloque el nombre de **CRUDRoom** al proyecto y haga click en **finish** para crear el proyecto y que Android Studio descargue dependencias básicas y cree la estructura del proyecto
![image](https://github.com/user-attachments/assets/6f6b9aec-cdd8-426d-a7f3-66551bdb5594)

## 2.- Instalar dependencias para Room, Navigation y ViewModel
### 2.1 agregar KSP en la sección plugins del build.gradle.kts de nivel superior
```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.jetbrains.kotlin.android) apply false

    id("com.google.devtools.ksp") version "2.0.20-1.0.25" apply false
}
```
Haga click en *Sync Now* para sincronizar el proyecto
![image](https://github.com/user-attachments/assets/8a33227b-f9e7-4cc9-9aa3-fab5fc7a3417)

**KSP (Kotlin Symbol Processing):** es una herramienta que facilita la generación de código en proyectos Kotlin. Es parte de la suite de herramientas de desarrollo para Android y Kotlin, y su principal objetivo es reemplazar o mejorar las limitaciones de KAPT (Kotlin Annotation Processing Tool), que se utilizaba tradicionalmente para procesar anotaciones y generar código en Kotlin.
KSP permite que los desarrolladores accedan a información de los programas de Kotlin (como clases, funciones, propiedades, etc.) en tiempo de compilación para crear bibliotecas y herramientas que generen código automáticamente.

### 2.2 Agregar plugin ksp en el archivo build.gradle.kts a nivel de modulo
```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.jetbrains.kotlin.android)

    id("com.google.devtools.ksp")
}
```
Haga click en *Sync Now* para sincronizar el proyecto nuevamente
### 2.3 Agregar dependencias para Room, Navigation y ViewModel en la seccion **dependencies** en el archivo build.gradle.kts a nivel de módulo
```kotlin
    val room_version = "2.6.1"
    implementation("androidx.room:room-runtime:$room_version")
    annotationProcessor("androidx.room:room-compiler:$room_version")
    ksp("androidx.room:room-compiler:$room_version")
    val nav_version = "2.8.0"
    // Jetpack Compose integration
    implementation("androidx.navigation:navigation-compose:$nav_version")
    //viewmodel
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.6")
```
Recuerde hacer click en *Sync Now* para sincronizar el proyecto nuevamente, y con esto hemos instalado las dependencias necesarias para trabajar con ROOM

## 3.- Crear estructura de packages para un mejor orden del código y seperación de los componentes
Creamos los packages models, navigation, room, states, viewmodels y views, en el package principal del proyecto como se muestra en la siguiente imágen

![image](https://github.com/user-attachments/assets/2c0ab6dc-e494-449a-952e-ded833e675a6)

## 4.- Crear la clase modelo (data class) Producto en el package models
```kotlin
@Entity(tableName = "productos")
data class Producto(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    @ColumnInfo("nombre")
    val nombre: String,
    @ColumnInfo("precio")
    val precio: Double
)
```

Verifique que Android Studio importe los espacios de nombre para PrimaryKey, ColumnInfo y Entity arriba de la definición de la data class
```kotlin
import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey
```
## 5.- Crear la clase para gestionar la base de datos con Room en el package room
Esta clase la vamos a definir como abstracta con el nombre de ProductosDatabase y la extender de RoomDatabase

```kotlin
import androidx.room.Database
import androidx.room.RoomDatabase
import com.zs.crudroom.models.Producto

@Database(entities = [Producto::class], version = 1, exportSchema = false)
abstract class ProductosDatabase : RoomDatabase() {
    abstract fun productoDao(): ProductoDao // Método abstracto para obtener el DAO
}```
