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
            Text(text = "Guardad Email")
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

![image](https://github.com/user-attachments/assets/ceb633d0-5469-4658-a80d-34fa8ebe8760)

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
