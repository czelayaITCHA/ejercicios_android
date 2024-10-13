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
### 2.3 agregar pluglin katp en el *build.gradle.kts* modulo :app
```kotlin
kotlin("kapt")
id("com.google.dagger.hilt.android")
```
Por problemas de dependencias con dagger y hilt vamos a usar kapt en este ejercicio
### 2.4 agregar dependencias en la sección de dependencies
```kotlin
   // Dagger Hilt
    implementation("com.google.dagger:hilt-android:2.46.1")
    kapt("com.google.dagger:hilt-compiler:2.46.1")

    val nav_version = "2.8.0"
    implementation("androidx.navigation:navigation-compose:$nav_version")
    //retrofit
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    //Coil
    implementation("io.coil-kt:coil-compose:2.5.0")
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
## 6.- Crear clase Constants que contenga un *companion object* para definir constantes en el package *utils* 
```kotlin
class Constants {
    companion object{
        const val BASE_URL = "https://fakestoreapi.com/"
        const val ENDPOINT = "products"
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
## 9.- Crear el modelo *ProductModel* en el package models
Para definir el o los modelos a utilizar debe analizarse la estructura del JSON devuelto por la API en el endpoint principal
```kotlin
data class ProductModel(
    val id : Int,
    val title : String,
    val price : Double,
    val description : String,
    val category : String,
    val image : String,
    val rating : Rating
)

data class Rating(
    val rate: Double,
    val count: Int
)
```
## 10.- Definir métodos para obtener todos los productos y por id de la interface ApiProduct por el momento

```kotlin
interface ApiProduct {

    @GET(ENDPOINT)
    suspend fun getProducts(): Response<List<ProductModel>>

    @GET("$ENDPOINT/{id}")
    suspend fun getProductById(@Path("id") id: Int): Response<ProductModel>

   //faltan métodos para agregar, editar y eliminar registros
}
```
## 11.- Crear el repositorio con el nombre ProductRepository en el package *repository*
```kotlin
class ProductRepository @Inject constructor(private val apiProduct: ApiProduct){

    suspend fun getProducts(): List<ProductModel>? {
        val response = apiProduct.getProducts()
        if(response.isSuccessful){
            return response.body()
        }
        return null
    }

    suspend fun getProductById(id: Int) : ProductModel?{
        val response = apiProduct.getProductById(id)
        if(response.isSuccessful){
            return response.body()
        }
        return null
    }
}
```
## 12.- Crear la clase *ProductState* para gestionar variables de estado
data class ProductState(
    val isLoading: Boolean = false,
    val products: List<ProductModel> = emptyList(),
    val errorMessage: String = ""

    val title: String = "",
    val price: Double = 0.0,
    val description: String = "",
    val category: String = "",
    val image: String = "",
    val rating: Rating = Rating(0.0, 0)
)


## 13.- Crear el ViewModel en el package viewmodels

```kotlin
@HiltViewModel
class ProductViewModel @Inject constructor(private val repository: ProductRepository) : ViewModel() {

    private val _products = MutableStateFlow<List<ProductModel>>(emptyList())
    val products = _products.asStateFlow()

    var state by mutableStateOf(ProductState())
        private set

    init {
        getProducts()
    }

    private fun getProducts() {
        viewModelScope.launch {
            withContext(Dispatchers.IO) {
                val products = repository.getProducts()
                Log.d("ProductViewModel", "Fetched products: $products")
                _products.value = products ?: emptyList()
            }
        }
    }

    fun getProductById(id: Int) {
        viewModelScope.launch {
            withContext(Dispatchers.IO){
                val product = repository.getProductById(id)
                state = state.copy(
                    title = product?.title ?: "",
                    price = product?.price ?: 0.0,
                    description = product?.description ?: "",
                    category = product?.category ?: "",
                    image = product?.image ?: "",
                    rating = product?.rating ?: Rating(0.0, 0)
                )
            }
        }
    }

}

```
## 14.- Crear archivo y funcion composable HomeView para verificar si se obtienen los productos de la API
Hasta este punto la función HomeView será sencilla
```kotlin
@Composable
fun HomeView(productViewModel: ProductViewModel) {
    val products by productViewModel.products.collectAsState()
    if (products.isEmpty()) {
        // Mostrar un mensaje si no hay productos
        Text("No hay productos disponibles.")
    } else {
        LazyColumn(
            modifier = Modifier.fillMaxSize(),
            contentPadding = PaddingValues(horizontal = 16.dp, vertical = 8.dp),
            verticalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            items(products) { product ->
                Text(text = product.title)
            }
        }
    }
}
```

## 15.- Invocar la función HomeView desde el Main Activity
```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    private val productViewModel: ProductViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ProductsApiAppTheme {
               Surface(
                   modifier = Modifier.fillMaxSize(),
                   color = MaterialTheme.colorScheme.background
               )
                {
                    HomeView(productViewModel)
               }
            } 
        }
    }
}
```
Ejecute la aplicación y deberia mostrar los títulos de los productos en los Text del HomeView

## 16.- Crear el archivo BodyComponents, en el package components para definir las funciones composables para los elementos visuales
```kotlin

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MainTopBar(title: String, showBackButton: Boolean = false, onCickBackButton: () -> Unit,
               onCickAction: () -> Unit){
    TopAppBar(
        title = { Text(text = title, color = Color.White, fontWeight = FontWeight.ExtraBold) },
        colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
            containerColor = MaterialTheme.colorScheme.primary
        ),
        navigationIcon = {
            if(showBackButton){
                IconButton(onClick = { onCickBackButton() }) {
                    Icon(imageVector = Icons.AutoMirrored.Filled.ArrowBack, contentDescription = "Back",
                        tint = Color.White)
                }
            }
        },
        actions = {
            if(!showBackButton){
                IconButton(onClick = { onCickAction() }) {
                    Icon(imageVector = Icons.Default.Search, contentDescription = "Back",
                        tint = Color.White)
                }
            }
        }
    )
}

@Composable
fun CardProduct(
    product: ProductModel,
    onClick: () -> Unit,
    onEditClick: () -> Unit,
    onDeleteClick: () -> Unit,
    function: () -> Unit

) {
    Card(
        shape = RoundedCornerShape(8.dp),
        modifier = Modifier
            .fillMaxWidth()
            .padding(10.dp)
            .shadow(elevation = 10.dp)
            .clickable { onClick() }
        ) {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(8.dp)
        ) {
            // Imagen del producto
            ProductImage(imageUrl = product.image)
            // Detalles del producto
            Spacer(modifier = Modifier.height(8.dp))
            Text(
                text = product.title,
                style = MaterialTheme.typography.titleLarge,
                modifier = Modifier.padding(horizontal = 8.dp)
            )

            Text(
                text = "$${product.price}",
                fontWeight = FontWeight.ExtraBold,
                style = MaterialTheme.typography.titleMedium.copy(color = Color.Black),
                modifier = Modifier.padding(horizontal = 8.dp),

            )
            RatingBar(rating = product.rating.rate)
            Spacer(modifier = Modifier.height(8.dp))

            // Botones de Editar y Eliminar
            Row(
                horizontalArrangement = Arrangement.SpaceBetween,
                modifier = Modifier.fillMaxWidth().padding(horizontal = 8.dp)
            ) {

                IconButton(onClick = { onEditClick() }) {
                    Icon(
                        imageVector = Icons.Default.Edit,
                        contentDescription = "Editar",
                        tint = Color.DarkGray
                    )
                }

                IconButton(onClick = { onDeleteClick() }) {
                    Icon(
                        imageVector = Icons.Default.Delete,
                        contentDescription = "Eliminar",
                        tint = Color.Red
                    )
                }

            }
        }
    }
}

@Composable
fun ProductImage(imageUrl: String) {
    val painter = rememberAsyncImagePainter(model = imageUrl)
    Image(
        painter = painter,
        contentDescription = null,
        contentScale = ContentScale.Fit, // Para evitar que la imagen se corte
        modifier = Modifier
            .fillMaxWidth()
            .height(250.dp)
            .padding(8.dp)
    )
}

@Composable
fun RatingBar(rating: Double) {
    Row(
        verticalAlignment = Alignment.CenterVertically,
        horizontalArrangement = Arrangement.Start,
        modifier = Modifier.padding(horizontal = 8.dp)
    ) {
        // Mostrar estrellas según el rating (entre 0 y 5)
        repeat(5) { index ->
            Icon(
                imageVector = Icons.Default.Star,
                contentDescription = null,
                tint = if (index < rating.toInt()) Color.Blue else Color.Gray,
                modifier = Modifier.size(20.dp)
            )
        }
        Spacer(modifier = Modifier.width(8.dp))
        Text(text = String.format("%.1f", rating))
    }
}
```

## 17.- Hacer cambios en la HomeView
```kotlin
@Composable
fun HomeView(viewModel: ProductViewModel, navController: NavController){
    Scaffold(
        topBar = {
            MainTopBar(title = "API Products", onCickBackButton = { }) {
                navController.navigate("SearchProductView")
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
        ContentHomeView(viewModel,it,navController)
    }
}

@Composable
fun ContentHomeView(viewModel: ProductViewModel, paddingValues: PaddingValues, navController: NavController){
    val products by viewModel.products.collectAsState()

    Column(
        modifier = Modifier.padding(paddingValues),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        LazyColumn(
            modifier = Modifier
                .background(Color(0xFF2B2626))
        ){
            items(products){product ->
                CardProduct(
                    product = product,
                    onClick = {},
                    onEditClick = {},onDeleteClick = {}) {
                    //enviar a la siguiente vista
                    
                }
                Text(text = product.title, fontWeight = FontWeight.ExtraBold, color = Color.White,
                    modifier = Modifier.padding(start = 10.dp))
            }
        }
    }
}

```
## 18.- Crear archivo *NavManager* para gestionar la navegación entre las UI

```kotlin
@Composable
fun NavManager(viewModel: ProductViewModel){
    val navController = rememberNavController()
    NavHost(navController = navController, startDestination = "Home"){
        composable("Home"){
            HomeView(viewModel, navController)
        }
        composable("DetailView/{id}", arguments = listOf(
            navArgument("id") { type = NavType.IntType }
        )  ){
            //val id = it.arguments?.getInt("id") ?: 0
            //DetailView(viewModel, navController, id)
        }
        
    }
}
```
## 19.- Llamar NavManager desde la actividad principal

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    private val productViewModel: ProductViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ProductsApiAppTheme {
               Surface(
                   modifier = Modifier.fillMaxSize(),
                   color = MaterialTheme.colorScheme.background
               )
                {
                    NavManager(productViewModel)
               }
            }
        }
    }
}
```

Ejecutar aplicación y observe como se cargan los productos de la API