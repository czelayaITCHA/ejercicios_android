# Ejemplo de CRUD para almacenamiento local con ROOM
## 1.- Crear nuevo proyecto con JetPack Compose
### 1.1 Seleccione Empty Activity de Jetpack Compose como se muestra en la imágen
![image](https://github.com/user-attachments/assets/48a10302-3e83-41a0-a13f-0446dc532a35)
### 1.2 Coloque el nombre de **CRUDRoom** al proyecto y haga click en **finish** para crear el proyecto y que Android Studio descargue dependencias básicas y cree la estructura del proyecto
![image](https://github.com/user-attachments/assets/6f6b9aec-cdd8-426d-a7f3-66551bdb5594)

## 2.- Instalar dependencias para Room, Navigation y ViewModel

### 2.1 Agregar plugin ksp en el archivo build.gradle.kts a nivel de modulo
```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.jetbrains.kotlin.android)

    id("com.google.devtools.ksp") version "1.9.0-1.0.12"
}
```
Haga click en *Sync Now* para sincronizar el proyecto

**KSP (Kotlin Symbol Processing):** es una herramienta que facilita la generación de código en proyectos Kotlin. Es parte de la suite de herramientas de desarrollo para Android y Kotlin, y su principal objetivo es reemplazar o mejorar las limitaciones de KAPT (Kotlin Annotation Processing Tool), que se utilizaba tradicionalmente para procesar anotaciones y generar código en Kotlin.
KSP permite que los desarrolladores accedan a información de los programas de Kotlin (como clases, funciones, propiedades, etc.) en tiempo de compilación para crear bibliotecas y herramientas que generen código automáticamente.

### 2.2 Agregar dependencias para Room, Navigation y ViewModel en la seccion **dependencies** en el archivo build.gradle.kts a nivel de módulo
```kotlin
    val room_version = "2.6.1"
    implementation("androidx.room:room-runtime:$room_version")
    annotationProcessor("androidx.room:room-compiler:$room_version")
    implementation("androidx.room:room-ktx:$room_version")
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
    val precio: Float
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
}
```
## 6.- Definir métodos de la interface ProductoDao, para ello primero crear la interface en el package room

```kotlin
@Dao
interface ProductoDao {
    @Query("SELECT * FROM productos")
    fun getAll(): Flow<List<Producto>>

    @Query("SELECT * FROM productos WHERE id = :id")
    fun getById(id: Int): Flow<Producto>

    @Insert
    suspend fun insert(producto: Producto)

    @Update
    suspend fun update(producto: Producto)

    @Delete
    suspend fun delete(producto: Producto)
}
```
Los métodos con **@Query**, como getAll() y getById(), retornan un Flow de datos. Flow es una característica de Kotlin Coroutines que permite trabajar con secuencias de datos asincrónicos de manera reactiva. Los métodos como insert(), update(), y delete() tienen la palabra clave suspend porque estos métodos no retornan un Flow, sino que realizan operaciones que pueden tomar tiempo y podrían bloquear el hilo principal.

Una **corrutina** en Android es una herramienta proporcionada por Kotlin para manejar tareas asincrónicas de manera eficiente y sencilla. Las corrutinas permiten ejecutar operaciones largas (como acceder a la red o leer/escribir en una base de datos) sin bloquear el hilo principal, lo que es crucial para mantener la interfaz de usuario (UI) fluida y evitar bloqueos.

## 7.- Definir el archivo de estados en el package states, para cuando la lista se actualiza guarde su estado y lo pinte en la UI
En este caso creamos una data class
```kotlin
import com.zs.crudroom.models.Producto

data class ProductoState(
    //definimos todos los elementos que guardan el estado
    val listProductos: List<Producto> = emptyList()
)

```
## 8.- Crear el ViewModel para vincular el modelo con la base de datos y la UI, colocar el nombre a la clase *ProductoViewModel* en el package viewmodels
```kotlin
class ProductoViewModel(private val productoDao: ProductoDao) : ViewModel() {

    var state by mutableStateOf(ProductoState())
        private set // el estado contendra el metodo set
    
    //definimos el comportamiento cuando se inicie el ViewModel
    init {
        viewModelScope.launch {
            //obtenemos el listado de productos y la convertimos a una coleccion
            productoDao.getAll().collectLatest {
                state = state.copy(
                    listProductos = it
                )
            }
        }
    }
    //implementamos los demas metodos
    fun addProducto(producto: Producto) = viewModelScope.launch {
        productoDao.insert(producto)
    }
    fun updateProducto(producto: Producto) = viewModelScope.launch {
        productoDao.update(producto)
    }
    fun deleteProducto(producto: Producto) = viewModelScope.launch {
        productoDao.delete(producto)
    }
}

```
## 9.- Ahora creamos la navegación de la aplicación para interactuar entre las diferentes vistas, para ello creamos un archivo con el nombre de NavManager en el package navigation

El archivo **NavManager** debe contener la función composable en donde se definen las rutas para la navegación de la app
```kotlin
@Composable
fun NavManager(viewModel: ProductoViewModel){
    val navController = rememberNavController()
    NavHost(navController = navController,startDestination = "home"){
        composable("home"){
            HomeView(navController, viewModel)
        }
        composable("agregar"){
            AddView(navController, viewModel)
        }

        composable("editar/{id}/{nombre}/{precio}", arguments = listOf(
            navArgument("id"){type = NavType.IntType},
            navArgument("nombre"){type = NavType.StringType},
            navArgument("precio"){type = NavType.FloatType}
        )){
            EditView(
                navController, viewModel,
                it.arguments!!.getInt("id"),
                it.arguments?.getString("nombre"),
                it.arguments?.getFloat("precio")
            )
        }
    }
}
```

## 10.- Creamos las vistas de la aplicación
### 10.1 creación de la vista HomeView
Primero creamos la función composable ContentHomeView, con contiene los wigets de la vista principal
```kotlin
@SuppressLint("DefaultLocale")
@Composable
fun ContentHomeView(paddingValues: PaddingValues, navController: NavController, viewModel: ProductoViewModel) {
    val state = viewModel.state
    Column(modifier = Modifier.padding(paddingValues)) {
        LazyColumn {
            items(state.listProductos) { producto ->
                Card(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(8.dp),
                    shape = RoundedCornerShape(8.dp),
                    colors = CardDefaults.cardColors(
                        containerColor = MaterialTheme.colorScheme.primary,
                    )
                ) {
                    Row(
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(16.dp),
                        verticalAlignment = Alignment.CenterVertically
                    ) {
                        Column(
                            modifier = Modifier.weight(1f)
                        ) {
                            Text(
                                text = producto.nombre,
                                style = MaterialTheme.typography.titleMedium, 
                                color = MaterialTheme.colorScheme.onPrimary,
                                modifier = Modifier.padding(bottom = 4.dp)
                            )
                            Text(
                                text = String.format("$%.2f", producto.precio),
                                style = MaterialTheme.typography.bodySmall,
                                color = MaterialTheme.colorScheme.onPrimary
                            )
                        }
                        Row(
                            horizontalArrangement = Arrangement.End,
                            modifier = Modifier.wrapContentWidth()
                        ) {
                                IconButton(onClick = {
                                val precioFormat = String.format("%.2f", producto.precio)
                                navController.navigate("editar/${producto.id}/${producto.nombre}/$precioFormat") }) {
                                Icon(imageVector = Icons.Default.Edit, contentDescription = "Editar")
                            }
                            IconButton(onClick = { viewModel.deleteProducto(producto) }) {
                                Icon(imageVector = Icons.Default.Delete, contentDescription = "Borrar")
                            }
                        }
                    }
                }
            }
        }
    }
}
```

Segundo creamos la funcion composable HomeView

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun HomeView(navController: NavController, viewModel: ProductoViewModel){
    Scaffold(
        topBar = {
            CenterAlignedTopAppBar(
                title = {
                    Text(text = "Home View", color = Color.White, fontWeight = FontWeight.Bold)
                },
                colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primary
                ))
        },
        //creando un fab
        floatingActionButton = {
            FloatingActionButton(
                onClick = {navController.navigate("agregar") },
                containerColor = MaterialTheme.colorScheme.primary,
                contentColor = Color.White
            ) {
                Icon(imageVector = Icons.Default.Add, contentDescription = "Agregar")
            }
        }
    ) {
        ContentHomeView(it,navController,viewModel)
    }
}
```

### 10.2 Programar las funciones componibles para agregar un Producto, para ello creamos un archivo AddView en el package views

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun AddView(navController: NavController, viewModel: ProductoViewModel){
    Scaffold(
        topBar = {
            CenterAlignedTopAppBar(title = {
                Text(text = "Agregar Producto", color = Color.White,fontWeight = FontWeight.Bold) },
                colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primary
                ),
                navigationIcon = {
                    IconButton(onClick = { navController.popBackStack()}) {
                        Icon(imageVector = Icons.AutoMirrored.Filled.ArrowBack, contentDescription = "Back", tint = Color.White)
                    }
                }
            )
        }
    ) {
        ContentAddView(it,navController,viewModel)
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ContentAddView(paddingValues: PaddingValues, navController: NavController, viewModel: ProductoViewModel){
    var nombre by remember { mutableStateOf("") }
    var precio by remember { mutableStateOf("") }

    Column(
        modifier = Modifier
            .padding(paddingValues)
            .padding(top = 30.dp)
            .fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally
    )
    {
        OutlinedTextField(
            value = nombre,
            onValueChange = {nombre = it},
            label = { Text(text = "Producto") },
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp)
                .padding(bottom = 12.dp)
        )
        OutlinedTextField(
            value = precio,
            onValueChange = {precio = it},
            label = { Text(text = "Precio") },
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number),
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp)
                .padding(bottom = 12.dp)
        )
        Button(
            onClick = {
                val producto = Producto(nombre=nombre, precio=precio.toFloat())
                viewModel.addProducto(producto)
                navController.popBackStack()
            }) {
            Text(text = "Agregar")
        }
    }
}
```
### 10.3 Programar las funciones componibles para editar un Producto, para ello creamos un archivo EditView en el package views

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun EditView(navController: NavController, viewModel: ProductoViewModel,
               id:Int, nombre: String?, precio: Float?){
    Scaffold(
        topBar = {
            CenterAlignedTopAppBar(title = {
                Text(text = "Editar Producto", color = Color.White,fontWeight = FontWeight.Bold) },
                colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primary
                ),
                navigationIcon = {
                    IconButton(onClick = { navController.popBackStack()}) {
                        Icon(imageVector = Icons.AutoMirrored.Filled.ArrowBack, contentDescription = "Back", tint = Color.White)
                    }
                }
            )
        }
    ) {
        ContentEditView(it,navController,viewModel, id, nombre, precio)
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ContentEditView(paddingValues: PaddingValues, navController: NavController, viewModel: ProductoViewModel,
                      id: Int, nombre: String?, precio: Float?){
    var nombre by remember { mutableStateOf(nombre) }
    var precio by remember { mutableStateOf(precio.toString()) }
    Column(
        modifier = Modifier
            .padding(paddingValues).padding(top = 30.dp).fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally
    )
    {
        OutlinedTextField(
            value = nombre ?: "",
            onValueChange = {nombre = it},
            label = { Text(text = "Producto") },
            modifier = Modifier.fillMaxWidth().padding(horizontal = 16.dp).padding(bottom = 12.dp)
        )
        OutlinedTextField(
            value = precio ?: "0",
            onValueChange = {precio = it},
            label = { Text(text = "Precio") },
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number),
            modifier = Modifier.fillMaxWidth().padding(horizontal = 16.dp).padding(bottom = 12.dp)
        )
        Button(
            onClick = {
                val producto = Producto(id=id, nombre=nombre!!, precio=precio.toFloat())
                viewModel.updateProducto(producto)
                navController.popBackStack()
            }) {
            Text(text = "Actualizar")
        }
    }
}
```
## 11.- Modificar evento onCreate de MainActivity para crear instancias y llamar función  NavManager

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            CRUDRoomTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    //creamos la variable para definir la bd
                    val database = Room.databaseBuilder(this, ProductosDatabase::class.java,"db_productos").build()
                    //definimos dao---para llamar los metodos
                    val dao = database.productoDao()
                    //creando nuestro viewModel
                    val viewModel = ProductoViewModel(dao)
                    NavManager(viewModel = viewModel)
                }
            }
        }
    }
}

```
## 12.- Ejecutar y probar la aplicación
![WhatsApp Image 2024-10-10 at 7 20 22 AM](https://github.com/user-attachments/assets/7e9967d4-633f-4769-98ca-7561e88c7061)

## 13.- Implementación de un TextField como filtro de búsqueda
Debido a que se puede tener una cantidad grande de productos almacenados en la base de datos del dispositivo, se debe implementar un filtro de búsqueda para encontrar productos que coincidan con la cadena de texto que el usuario digite en un TextField, para ello vamos a modificar las funciones componibles del archivo HomeView.kt
### 13.1 crear variable de estado searchQuery y TextField en la función HomeView
```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun HomeView(navController: NavController, viewModel: ProductoViewModel) {
    var searchQuery by remember { mutableStateOf("") }

    Scaffold(
        topBar = {
            Column {
                CenterAlignedTopAppBar(
                    title = {
                        Text(text = "Listado de Productos", color = Color.White, fontWeight = FontWeight.Bold)
                    },
                    colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                        containerColor = MaterialTheme.colorScheme.primary
                    )
                )
                // Barra de búsqueda
                TextField(
                    value = searchQuery,
                    onValueChange = { query -> searchQuery = query },
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(8.dp),
                    placeholder = { Text("Buscar producto...") },
                    leadingIcon = { Icon(Icons.Default.Search, contentDescription = "Buscar") },
                    colors = TextFieldDefaults.textFieldColors(
                        containerColor = MaterialTheme.colorScheme.surface,
                        focusedIndicatorColor = MaterialTheme.colorScheme.primary,
                        unfocusedIndicatorColor = Color.Transparent
                    )
                )
            }
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = { navController.navigate("agregar") },
                containerColor = MaterialTheme.colorScheme.primary,
                contentColor = Color.White
            ) {
                Icon(imageVector = Icons.Default.Add, contentDescription = "Agregar")
            }
        }
    ) {
        ContentHomeView(
            paddingValues = it,
            navController = navController,
            viewModel = viewModel,
            searchQuery = searchQuery
        )
    }
}

```
### 13.2 filtrar la lista de productos con la cadena que el usuario digite en el TextField, en la funcion ContentHomeView

```kotlin
@SuppressLint("DefaultLocale")
@Composable
fun ContentHomeView(
    paddingValues: PaddingValues,
    navController: NavController,
    viewModel: ProductoViewModel,
    searchQuery: String
) {
    val productos = viewModel.state.listProductos

    // Filtrar productos basado en la búsqueda
    val filteredProductos = remember(searchQuery, productos) {
        productos.filter { producto ->
            producto.nombre.contains(searchQuery, ignoreCase = true)
        }
    }

    Column(modifier = Modifier.padding(paddingValues)) {
        LazyColumn {
            items(filteredProductos) { producto ->
                Card(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(4.dp),
                    shape = RoundedCornerShape(8.dp),
                    colors = CardDefaults.cardColors(
                        containerColor = MaterialTheme.colorScheme.primary,
                    )
                ) {
                    Row(
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(16.dp),
                        verticalAlignment = Alignment.CenterVertically
                    ) {
                        Column(
                            modifier = Modifier.weight(1f)
                        ) {
                            Text(
                                text = producto.nombre,
                                style = MaterialTheme.typography.titleMedium,
                                color = MaterialTheme.colorScheme.onPrimary,
                                modifier = Modifier.padding(bottom = 4.dp)
                            )
                            Text(
                                text = String.format("$%.2f", producto.precio),
                                style = MaterialTheme.typography.bodySmall,
                                color = MaterialTheme.colorScheme.onPrimary
                            )
                        }
                        Row(
                            horizontalArrangement = Arrangement.End,
                            modifier = Modifier.wrapContentWidth()
                        ) {
                            IconButton(onClick = {
                                val precioFormat = String.format("%.2f", producto.precio)
                                navController.navigate("editar/${producto.id}/${producto.nombre}/$precioFormat") }) {
                                Icon(imageVector = Icons.Default.Edit, contentDescription = "Editar")
                            }
                            IconButton(onClick = { viewModel.deleteProducto(producto) }) {
                                Icon(imageVector = Icons.Default.Delete, contentDescription = "Borrar")
                            }
                        }
                    }
                }
            }
        }
    }
}
```
Probar aplicación

## Explicación
**Estado de búsqueda local:** En HomeView, se crea un searchQuery con remember que se actualiza cada vez que el usuario escribe en el TextField.

**Filtrado local:** Usando remember y derivedStateOf, filtramos la lista de productos dentro del mismo composable, de modo que el filtro es reactivo y eficiente. Cada vez que searchQuery o la lista de productos cambien, el filtrado se actualiza.

**Mostrar los productos filtrados:** La lista filtrada filteredProductos se muestra dentro del LazyColumn, manteniendo el diseño y funcionalidad original del componente.

Este enfoque es eficiente y adecuado para filtrar en tiempo real en la interfaz.

