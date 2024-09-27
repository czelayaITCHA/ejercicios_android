# Ejercicio desarrollo de aplicación DiscountsApp


## 1. Crear nuevo proyecto con el nombre discountsApp
### 1.1 Seleccionar Empty Activity de Jetpack compose
![image](https://github.com/user-attachments/assets/28c074cf-4b69-44e4-a2c9-7aac6231d34b)
### 1.2 Definir nombre del proyecto 
Asigne el nombre de DiscountsApp y haga click en finish tal como se muestra en la siguiente ventana, para que Android Studio cree estructura de la aplicación 
![image](https://github.com/user-attachments/assets/680b7cc2-11ed-4add-999e-55f9ab6ee2f8)
## 2. Eliminar funcion composable Greeting y su llamada en el evento onCreate de MainActivity.kt y agregamos un Surface de material3
El código del evento onCreate de MainActivity quedaría como se muestra a continuación 

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            DiscountsAppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ){

                }
            }
        }
    }
}
```
En Jetpack Compose, Surface es un contenedor que sirve como un bloque de construcción para los elementos visuales, similar a una tarjeta o una pantalla. Proporciona características como personalización de color, elevación y forma.
## 3. Crear package components dentro del package principal del proyecto
![image](https://github.com/user-attachments/assets/345469d2-fac7-4e93-8e11-a27c1325dad0)
## 4. Ahora vamos a crear unos archivos dentro del package components, que contendrán las funciones componibles para construir pequeñas piezas que al unirlas obtendremos la UI
### 4.1 primero creamos el archivo BodyComponents
![image](https://github.com/user-attachments/assets/61fc21d7-2f6a-45ab-8d1f-b7cd2573afbc)
### 4.2 Haga el mismo proceso para crear los archivos CardComponents y HomeView
Debe quedar la estructura del proyecto y archivos como se muestra en la siguiente imagen
![image](https://github.com/user-attachments/assets/ed973ca8-1f70-4623-95db-5315f85d14fc)

## 5. Programar funciones composables en cada archivo
### 5.1 Programar funciones en BodyComponents.kt
Primero vamos crear funciones para definir spacio para alto y ancho
```kotlin
@Composable
fun SpaceH(size: Dp = 4.dp){
    Spacer(modifier = Modifier.height(size))
}

@Composable
fun SpaceW(size: Dp = 4.dp){
    Spacer(modifier = Modifier.width(size))
}
```
Las funciones anteriores simplemente definen el alto y ancho respectivamente del espacio vertical y horizontal entre elementos visuales, son funciones que reciben como parámetro el tamaño (size) en pixeles del espacio a definir, en caso de no recibir el valor, se asignará 4dp.

Como siguiente paso vamos a crear una función para definir características o propiedades de los TextField a utilizar en la UI, se completará código en la explicación del docente
```kotlin
@Composable
fun MainTextField(value: String, onValueChange: (String) -> Unit, label: String){
    //definir propiedades
}
```
Ahora vamos a programar una función para definir los botones a utilizar en la UI
```kotlin
@Composable
fun MainButton(text: String, color: Color = MaterialTheme.colorScheme.primary, onClick:() -> Unit){
    OutlinedButton(onClick = onClick, colors = ButtonDefaults.outlinedButtonColors(
        contentColor = color,
        containerColor = Color.Transparent
    ),
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 30.dp)
    ) {
        Text(text = text)
    }
}
```
Programar función para mostrar alerta cuando el usuario no digite un dato en los TextField
```kotlin
@Composable
fun Alert(
    title: String,
    message: String,
    confirmText: String,
    onConfirmClick: () -> Unit,
    onDismissClick: ()-> Unit
){
    AlertDialog(
        onDismissRequest = onDismissClick,
        title = { Text(text = title)},
        text = { Text(text = message)},
        confirmButton = {
            Button(onClick = { onConfirmClick() }) {
                Text(text = confirmText)
            }
        }
        )
}
```
### 5.2 Programar funciones para CardComponents.kt
#### 5.2.1 creamos una función que contenga un elemento visual card para mostrar resultado de total y descuento
```kotlin
@Composable
fun MainCard(title: String, number: Double, modifier: Modifier = Modifier){
    Card(
        modifier = modifier,
        colors = CardDefaults.cardColors(
            containerColor = Color.DarkGray
        )
    ) {
        Column(
            verticalArrangement = Arrangement.Center,
            horizontalAlignment = Alignment.CenterHorizontally,
            modifier = Modifier.padding(16.dp)
        ) {
            Text(text = title, color = Color.White, fontSize = 20.sp)
            Text(text = "$$number", color = Color.White, fontSize = 20.sp)
        }
    }
}
```
#### 5.2.2 creamos una función que permita reutilizar la función MainCard y mostrar dos cards a la par utilizando Row
```kotlin
@Composable
fun TwoCards(title1: String, number1: Double, title2: String, number2: Double){
    Row(modifier = Modifier.fillMaxWidth(),
        horizontalArrangement = Arrangement.SpaceEvenly
    ) {
        MainCard(title = title1, number = number1,
            modifier = Modifier
                .padding(start = 28.dp)
                .weight(1f))
        SpaceW(10.dp)
        MainCard(title = title2, number = number2,
            modifier = Modifier
                .padding(end = 28.dp)
                .weight(1f)
        )
    }
}
```

### 5.3 Programar funciones en HomeView.kt
#### 5.3.1 programar funciones no componibles para calcular descuento y precio o total a pagar
```kotlin
fun calcPrecio(precio: Double, descuento: Double): Double{
    val res = precio - calcDescuento(precio, descuento)
    return kotlin.math.round(res * 100) / 100.0
}

fun calcDescuento(precio: Double, descuento: Double): Double{
    val res = precio * (1 - descuento / 100)
    return kotlin.math.round(res * 100) / 100.0
}
```
#### 5.3.2 crear función para mostrar el diseño de UI principal y programar funcionalidad básica
```kotlin
@Composable
fun ContentHomeView(paddingValues: PaddingValues){
    Column(modifier= Modifier
        .padding(paddingValues)
        .padding(10.dp)
        .fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        var precio by remember { mutableStateOf("") }
        var descuento by remember { mutableStateOf("") }
        var precioDescuento by remember { mutableStateOf(0.0) }
        var totalDescuento by remember { mutableStateOf(0.0) }
        //variable para mostrar la alerta
        var showAlert by remember { mutableStateOf(false) }
        //colocando las dos card
        TwoCards(title1 = "Total", number1 = totalDescuento, title2 ="Descuento" , number2 = precioDescuento)
        MainTextField(value = precio, onValueChange = {precio=it}, label = "Precio")
        SpaceH()
        MainTextField(value = descuento, onValueChange = {descuento=it}, label = "Descuento %")
        SpaceH(10.dp)
        MainButton(text = "Generar Descuento") {
            if(precio != "" && descuento !=""){
                precioDescuento = calcPrecio(precio.toDouble(), descuento.toDouble())
                totalDescuento = calcDescuento(precio.toDouble(), descuento.toDouble())
            }else{
                showAlert = true
            }
        }
        //evaluando si se muestra la alerta
        if(showAlert){
            Alert(
                title = "Alerta",
                message = "Digita el precio y el descuento",
                confirmText = "Aceptar",
                onConfirmClick = { showAlert = false }) {}
        }
        SpaceH()
        MainButton(text = "Limpiar", color = Color.Red) {
            precio = ""
            descuento = ""
            precioDescuento = 0.0
            totalDescuento = 0.0
        }
    }
}
```
#### 5.3.3 Programar función que contenga un Scaffold para definir un topBar y colocar un título a la aplicación 
```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun HomeView(){
    Scaffold(topBar = {
        CenterAlignedTopAppBar(
            title = { Text(text = "App Descuentos", color = Color.White) },
            colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                containerColor = MaterialTheme.colorScheme.primary
            )
            )
    }) {
        ContentHomeView(it)
    }
}
```
Este código crea una pantalla en **Jetpack Compose** con una barra superior (`TopAppBar`) y un contenido debajo usando el componente `Scaffold`.

### Explicación:

1. **`@OptIn(ExperimentalMaterial3Api::class)`**: Marca que se está usando una API experimental de Material 3.

2. **`HomeView()`**: Es una función composable que define la estructura de la vista.

3. **`Scaffold(...)`**: Estructura básica de una pantalla que incluye una barra superior (`topBar`) y el contenido principal.

4. **`CenterAlignedTopAppBar(...)`**: Es la barra superior que contiene un título centrado ("App Descuentos") y usa un color de fondo (`primary` del tema).

5. **`ContentHomeView(it)`**: Función que define el contenido principal de la pantalla. `it` es el **padding** generado por el `Scaffold` para evitar que el contenido quede oculto debajo de la barra superior.

## 6. Llamar la funcion HomeView() en el cuerpo de Surface() en el evento onCreate de MainActivity.kt
![image](https://github.com/user-attachments/assets/8fc4ebb9-1252-46f9-a880-b5a48f705969)
## 7. Ejecutar aplicación y probar funcionalidad
En la siguiente imágen vemos el resultado de los calculos del descuento y el total a pagar, calculados a partir del precio(puede ser de un producto), y el porcentaje de descuento.

![image](https://github.com/user-attachments/assets/3235b69d-0182-491d-8596-ce123e6c4041)

## 8. Habilitar teclado numérico cuando el usuario digite precio y descuento, ejercicio para usted.
![image](https://github.com/user-attachments/assets/4a615518-c90d-4954-952e-167e5c5f6e56)

