## Introducción 
El patrón MVVM (Model-View-ViewModel) es un patrón de diseño arquitectónico que ayuda a separar la lógica de negocio, la lógica de presentación y la interfaz de usuario en las aplicaciones Android. Es muy utilizado cuando se emplean frameworks como Jetpack ViewModel y LiveData, que facilitan la implementación del patrón.

**Model (M):**
Contiene la lógica de negocio y la gestión de datos. Es responsable de acceder a los datos, ya sea desde una base de datos, una API remota o cualquier otra fuente.
Es completamente independiente de la vista y del ViewModel. El modelo no tiene conocimiento de las capas superiores.

**View (V):**
Es la interfaz de usuario. En Android, generalmente son las actividades, fragmentos, o vistas que interactúan con los usuarios.
Se encarga de observar los datos expuestos por el ViewModel y reflejar los cambios en la interfaz.
Debe ser lo más sencilla posible y no contener lógica de negocio.

**ViewModel (VM):**
Actúa como intermediario entre el modelo y la vista.
Contiene la lógica de presentación. No tiene referencias directas a la vista, lo que facilita el testeo.
Almacena y gestiona los datos necesarios para la interfaz de usuario y maneja la lógica de la vista.
Utiliza observables (como LiveData o StateFlow) para notificar a la vista de los cambios en los datos.

    ![image](https://github.com/user-attachments/assets/212e4aef-fc84-4f16-9185-a6022f94c59e)

