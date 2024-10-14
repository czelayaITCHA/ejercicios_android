# Ejemplo de DataStore con JetPack Compose

**DataStore** en Jetpack Compose es una solución moderna de Android para almacenar y gestionar datos de manera eficiente y segura. Reemplaza a SharedPreferences y ofrece dos tipos de almacenamiento:

Preferences DataStore: Para guardar pares clave-valor, ideal para configuraciones o preferencias del usuario.
Proto DataStore: Para almacenar datos estructurados utilizando Protocol Buffers.
Se puede utilizar para almacenar configuraciones de la aplicación, preferencias de usuario, o cualquier dato que necesite persistencia entre sesiones de uso. DataStore es eficiente, asíncrono y seguro, con soporte para operaciones reactivas como Flow y LiveData.

Veamos un ejemplo sencillo:
## 1.- Crear nuevo proyecto con Empty Activity (JetPack Compose)
![image](https://github.com/user-attachments/assets/08b29e7d-4b61-4ea9-9d52-ef10bba84daa)

## 2.- Instalar dependencia para trabajar con DataStore en el archivo build.gradle.kts app:
Se puede agregar al final del archivo o después de la dependencia de material3
```kotlin
implementation("androidx.datastore:datastore-preferences:1.1.1")
````
Tal como se ve en la siguiente imágen, recuerde hacer click en *Sync now*
![image](https://github.com/user-attachments/assets/41afaacf-1ae8-4c77-87ff-86bebb5d9e35)
**Nota:** es recomendable después de instalar dependencias o agregar plugins a los archivos gradles correr la aplicación aunque le instale bien las dependencias, pero al ejecutar se garantiza que no habrá problemas
## 3.- Crear una función composable en MainActivity capturar el email de una cuenta de usuario
```kotlin
@Composable
fun HomeView(){
    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center,
        modifier = Modifier.fillMaxSize()
            .padding(horizontal = 16.dp))
    {
        var email by rememberSaveable { mutableStateOf("") }

        TextField(
            value = email,
            onValueChange = {email = it},
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Email),
            modifier = Modifier.fillMaxWidth()
        )
        Spacer(modifier = Modifier.height(16.dp))
        Button(onClick = { /*TODO*/ }) {
            Text(text = "Guardar Email")
        }
        Spacer(modifier = Modifier.height(16.dp))
        Text(text = "Email: $email", modifier = Modifier.wrapContentWidth())
    }
}
```
Usamos **rememberSaveable** en la variable de estado para mantener el valor cuando se dé un cambio de orientación del dispositivo y se reconstruya la UI, si usamos **remember** el valor se pierde. Este problema no ocurre cuando se usa ViewModel

Hacemos la llamda de la función desde el evento onCreate de MainActivity y la clase queda así:
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            DataStoreAppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                )
                {
                    HomeView()
                }
            }
        }
    }
}    
```
Ejecutamos la aplicación y el resultado debe ser como la siguiente imágen

![image](https://github.com/user-attachments/assets/036ce367-07c0-45d3-9dea-494384b9077b)

## 4.- Crear la clase DataStore para persistir datos en el dispositivo
Creamos a nivel de package principal una clase con el nombre **StoreUserEmail**
```kotlin
class StoreUserEmail(private val context: Context) {

    companion object {
        private val Context.dataStore : DataStore<Preferences> by preferencesDataStore("UserEmail")
        private val USER_EMAIL_KEY = stringPreferencesKey("user_email")

    }

    //creamos la variable para obtener el email
    val getEmail: Flow<String?> = context.dataStore.data
        .map { preferences -> preferences[USER_EMAIL_KEY] ?: ""}

    //creamos la funcion para guardar el email
    suspend fun saveEmail(email: String) {
        context.dataStore.edit { preferences ->
            preferences[USER_EMAIL_KEY] = email
        }
    }
}
```
## 5.- Editar la función composable HomeView para implementar la clase StoreUserEmail
Obtenemos el contexto para pasarselo a la clase StoreUserEmail, después de la definición de la funcion HomeView, creamos la instancia de la clase e implementamos el método para guardar el email mediante DataStore, la función queda de la siguiente manera:
```kotlin
@Composable
fun HomeView(){
    //obtenemos el contexto para pasarselo a la clase StoreUserEmail
    val context = LocalContext.current
    val scopelaunch = rememberCoroutineScope()
    val dataStore = StoreUserEmail(context)

    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center,
        modifier = Modifier.fillMaxSize()
            .padding(horizontal = 16.dp))
    {
        var email by rememberSaveable { mutableStateOf("") }
        val userEmail = dataStore.getEmail.collectAsState(initial = "")
        TextField(
            value = email,
            onValueChange = {email = it},
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Email),
            modifier = Modifier.fillMaxWidth()
        )
        Spacer(modifier = Modifier.height(16.dp))
        Button(onClick = {
            scopelaunch.launch {
                dataStore.saveEmail(email)
            }
        }) {
            Text(text = "Guardar Email")
        }
        Spacer(modifier = Modifier.height(16.dp))
        Text(userEmail.value.toString(), modifier = Modifier.wrapContentWidth())
    }
}


```
## 6.- Correr la aplicación y hacer pruebas

| Vista 1   | Vista 2   | Vista 3   |
|-------------|-------------|-------------|
| ![image](https://github.com/user-attachments/assets/895fd43e-dcae-4c48-bd0b-f3b8ca6f96d0) | ![image](https://github.com/user-attachments/assets/f3a31d2e-b2e5-4038-a242-6a6686a98526)  | ![image](https://github.com/user-attachments/assets/fed63651-9543-4ef6-9803-b28e1b9a0cbf)  |
| Captura de email con teclado tipo email habilitado  | Después de guardar (botón guardar email) el email  | Después de haber cerrado e iniciado aplicación e incluso de hacer reiniciado dispositivo  |

Puede verificar que el dato del email se hace persistente en el dispositivo, puede cambiar su valor y solo se borrará si se desintala la aplicación. Esto lo permite DataStore, que anteriormente se hacia con SharedPreferences.

## 7.- Cambiar a modo dark o light de la aplicación con DataStore
### 7.1 Creamos la clase StoreDarkMode

```kotlin
class StoreDarkMode(private val context: Context) {

    companion object {
        private val Context.dataStore : DataStore<Preferences> by preferencesDataStore("DarkMode")
        private val DARK_MODE_KEY = booleanPreferencesKey("dark_mode")

    }

    val getMode: Flow<Boolean> = context.dataStore.data
        .map { preferences -> preferences[DARK_MODE_KEY] ?: false}

    suspend fun saveMode(value: Boolean) {
        context.dataStore.edit { preferences ->
            preferences[DARK_MODE_KEY] = value
        }
    }

}
```
### 7.2 Definir el contexto en MainActivity, después de setContent
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            val darkModeStore = StoreDarkMode(this)
            val darkMode = darkModeStore.getMode.collectAsState(initial = false)

            DataStoreAppTheme(
                darkTheme = darkMode.value
            ) {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                )
                {
                    HomeView()
                }
            }
        }
    }
}

```
### 7.3 modificamos la función HomeView para que reciba los parametros de tipo StoreDarkMode y el Booleano
```kotlin
fun HomeView(darkModeStore: StoreDarkMode, darkMode: Boolean)
``` 
### 7.4 En la llamada a la función HomeView dentro del evento onCreate de MainActivity, le pasamos los parámetros

```kotlin
HomeView(darkModeStore, darkMode.value)
```
### 7.5 Crear botones para cambiar el modo en la función composable HomeView

```kotlin
//Para cambiar el modo de la app
        Spacer(modifier = Modifier.height(16.dp))
        Button(onClick = {
            scopelaunch.launch {
                darkModeStore.saveMode(!darkMode)
            }
        }) {
            Text(text = "Cambiar Modo")
        }

        //boton tipo Switch para cambiar el modo
        Switch(checked = darkMode, onCheckedChange = { isChecked ->
            scopelaunch.launch {
                darkModeStore.saveMode(isChecked)
            }
        })
```

Ejecute la aplicación y realice pruebas
