# El sistema de vistas de Android (Views + XML)

> **Unidad 1 · Tecnologías para aplicaciones en dispositivos móviles**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Explicar qué es una `View` y un `ViewGroup`, y cómo forman la jerarquía de una pantalla.
- Diseñar un layout sencillo en XML y cargarlo desde una actividad.
- Acceder a las vistas desde Kotlin con **View Binding** y responder a eventos.
- Explicar qué es Jetpack Compose y por qué en este módulo empezamos con el sistema de vistas.

---

## 1. Dos formas de construir interfaces en Android

Android ofrece hoy dos herramientas para construir la interfaz de usuario de una app nativa:

| | **Sistema de vistas (Views)** | **Jetpack Compose** |
|---|---|---|
| Desde | Android 1.0 (2008) | Versión estable en 2021 |
| Cómo se diseña | Ficheros **XML** + código Kotlin/Java | Solo código **Kotlin** |
| Estilo | **Imperativo**: obtienes una vista y le dices qué cambiar | **Declarativo**: describes cómo debe verse la UI según el estado |
| Situación actual | Mantenido; presente en la gran mayoría de apps existentes | **Recomendado por Google** para proyectos nuevos |

En este módulo **empezaremos con el sistema de vistas**. En el apartado 8 explicamos por qué.

---

## 2. Conceptos básicos: `View` y `ViewGroup`

Toda pantalla construida con el sistema de vistas es un **árbol de objetos**:

- **`View`**: un elemento que se dibuja en pantalla y con el que el usuario puede interactuar. Se les suele llamar *widgets*: `TextView`, `Button`, `EditText`, `ImageView`, `CheckBox`…
- **`ViewGroup`**: un contenedor invisible que **organiza** otras vistas (o a otros `ViewGroup`). Se les llama *layouts*: `LinearLayout`, `ConstraintLayout`, `FrameLayout`…

```
ConstraintLayout                 ← ViewGroup (raíz)
├── TextView    "Bienvenido"     ← View
├── EditText    (nombre)         ← View
└── LinearLayout (horizontal)    ← ViewGroup anidado
    ├── Button  "Aceptar"        ← View
    └── Button  "Cancelar"       ← View
```

> 💡 **Conexión con POO:** `ViewGroup` **hereda** de `View`, y un `ViewGroup` **contiene** objetos `View`. Es el patrón de diseño **Composite**: un contenedor se trata igual que un elemento simple, y eso permite anidar estructuras tan profundas como haga falta.

---

## 3. El layout en XML

Los layouts se guardan en la carpeta **`res/layout/`** del proyecto. Cada fichero tiene **un único elemento raíz** (un `View` o un `ViewGroup`) y dentro se van anidando los demás, de forma parecida a como se escribe HTML.

```xml
<!-- res/layout/activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/tvSaludo"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/saludo"
        android:textSize="20sp" />

    <EditText
        android:id="@+id/etNombre"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:hint="@string/hint_nombre"
        android:inputType="textPersonName" />

    <Button
        android:id="@+id/btnSaludar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="@string/saludar" />
</LinearLayout>
```

### Atributos que debes conocer

| Atributo | Significado |
|---|---|
| `android:id="@+id/nombre"` | Identificador para poder acceder a la vista desde el código. El `+` indica que se crea un ID nuevo |
| `layout_width` / `layout_height` | **Obligatorios** en todas las vistas |
| `match_parent` | Ocupa todo el espacio que permite el contenedor padre |
| `wrap_content` | Ocupa solo lo que necesita su contenido |
| `padding` | Espacio **interior**, entre el borde de la vista y su contenido |
| `layout_margin` | Espacio **exterior**, entre la vista y las que la rodean |
| `@string/...` | Referencia a un texto definido en `res/values/strings.xml` |

### Unidades de medida

- **`dp`** (*density-independent pixels*) para tamaños y márgenes. Se ven igual de grandes en pantallas de distinta densidad.
- **`sp`** (*scale-independent pixels*) para tamaños de texto. Como `dp`, pero además respeta el tamaño de letra que el usuario elige en los ajustes (accesibilidad).
- ❌ **Evita `px`**: un píxel no mide lo mismo en todos los dispositivos.

### Los textos van en recursos, no en el layout

```xml
<!-- res/values/strings.xml -->
<resources>
    <string name="app_name">Saludador</string>
    <string name="saludo">Bienvenido</string>
    <string name="hint_nombre">Escribe tu nombre</string>
    <string name="saludar">Saludar</string>
</resources>
```

Así la app se puede **traducir** añadiendo, por ejemplo, `res/values-gl/strings.xml` sin tocar el diseño. Es el **Resource Manager** del framework el que elige el fichero adecuado según el idioma del dispositivo.

---

## 4. Los layouts más usados

| Layout | Cómo organiza a sus hijos | Cuándo usarlo |
|---|---|---|
| **`LinearLayout`** | En una fila o una columna (`orientation`) | Formularios sencillos, grupos de botones |
| **`ConstraintLayout`** | Cada vista se posiciona con **restricciones** respecto al padre o a otras vistas | **Opción recomendada** para pantallas completas: es flexible y evita anidar layouts |
| **`FrameLayout`** | Apila las vistas unas sobre otras | Contenedor de un solo elemento o de un fragment |
| **`RecyclerView`** | Lista o cuadrícula con desplazamiento, que **recicla** las vistas que salen de pantalla | Cualquier lista de datos de longitud variable |

> ⚠️ **Buena práctica:** mantén la jerarquía lo más **plana** posible. Cada nivel de anidamiento obliga a medir y dibujar más, y la pantalla tarda más en mostrarse. `ConstraintLayout` existe precisamente para evitar el anidamiento.

---

## 5. Del XML al código: View Binding

El layout por sí solo no hace nada. Hay que **cargarlo** en la actividad y **acceder** a sus vistas para programar su comportamiento.

### 5.1. Activar View Binding

En el `build.gradle.kts` del módulo `app`:

```kotlin
android {
    buildFeatures {
        viewBinding = true
    }
}
```

Con esto, Android Studio **genera automáticamente una clase por cada layout**. El nombre se forma pasando el nombre del fichero a *PascalCase* y añadiendo `Binding`:

`activity_main.xml` → `ActivityMainBinding`

Esa clase tiene una propiedad por cada vista que tenga `id` (`tvSaludo`, `etNombre`, `btnSaludar`), además de `root`, que es la vista raíz.

### 5.2. Usarlo en la actividad

```kotlin
class MainActivity : AppCompatActivity() {

    // Referencia a la clase generada para activity_main.xml.
    // lateinit: se inicializa en onCreate, no en la declaración.
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 1. "Inflar" el layout: convertir el XML en objetos View
        binding = ActivityMainBinding.inflate(layoutInflater)

        // 2. Mostrar la vista raíz como contenido de la actividad
        setContentView(binding.root)

        // 3. Programar el comportamiento: escuchar el clic del botón
        binding.btnSaludar.setOnClickListener {
            val nombre = binding.etNombre.text.toString().trim()

            binding.tvSaludo.text = if (nombre.isEmpty()) {
                getString(R.string.saludo)
            } else {
                "Hola, $nombre"
            }
        }
    }
}
```

Fíjate en que todo ocurre en **`onCreate()`**, el primer método del **ciclo de vida** de la actividad. Lo estudiaremos en el punto 5 de la unidad.

### 5.3. ¿Por qué View Binding y no `findViewById()`?

En código antiguo verás esto:

```kotlin
val btnSaludar: Button = findViewById(R.id.btnSaludar)
```

View Binding es mejor porque:

- **Es seguro frente a nulos**: si una vista no existe en el layout, la propiedad no existe y el proyecto **no compila**. Con `findViewById` el error salta en tiempo de ejecución.
- **Es seguro en tipos**: `binding.btnSaludar` ya es un `Button`. No hay que hacer *cast* ni existe riesgo de `ClassCastException`.

> 🔴 **Cuidado con los tutoriales antiguos:** si ves `import kotlinx.android.synthetic...` y vistas usadas directamente por su nombre (`btnSaludar.setOnClickListener`), son los **Kotlin synthetics**. Ese mecanismo está **eliminado** y hay que sustituirlo por View Binding.

---

## 6. Un vistazo a Jetpack Compose

Para que veas la diferencia, esta es **la misma pantalla** escrita con Compose. No hace falta que la entiendas en detalle todavía.

```kotlin
@Composable
fun PantallaSaludo() {
    // El estado: cuando cambia, Compose vuelve a dibujar lo que depende de él
    var nombre by remember { mutableStateOf("") }
    var saludo by remember { mutableStateOf("Bienvenido") }

    Column(modifier = Modifier.padding(16.dp)) {
        Text(text = saludo, fontSize = 20.sp)
        TextField(
            value = nombre,
            onValueChange = { nombre = it },
            label = { Text("Escribe tu nombre") }
        )
        Button(onClick = { saludo = if (nombre.isBlank()) "Bienvenido" else "Hola, $nombre" }) {
            Text("Saludar")
        }
    }
}
```

La diferencia de fondo está en el modelo mental:

- **Views (imperativo):** "busca el `TextView` y **cámbiale** el texto".
- **Compose (declarativo):** "el texto **es** el valor de `saludo`". Cuando `saludo` cambia, la interfaz se actualiza sola.

---

## 7. Ventajas e inconvenientes

### Sistema de vistas

✅ Separa claramente el **diseño (XML)** del **comportamiento (Kotlin)**.
✅ Tiene un **editor visual** de arrastrar y soltar en Android Studio.
✅ Es la base de la mayoría de apps en producción y de muchísima documentación, cursos y ejemplos.
✅ Su funcionamiento está muy ligado al framework (actividades, ciclo de vida, recursos), lo que ayuda a entender Android "por dentro".
❌ Requiere más código repetitivo, sobre todo en las listas (`RecyclerView` + `Adapter` + `ViewHolder`).
❌ Hay que mantener sincronizados el XML y el código.
❌ Es más fácil cometer errores de estado: que la UI no refleje los datos porque se olvidó actualizar una vista.

### Jetpack Compose

✅ Necesita menos código, y todo está en Kotlin.
✅ El modelo declarativo evita muchos errores de sincronización entre datos e interfaz.
✅ Las listas son mucho más sencillas (`LazyColumn`).
✅ Es la dirección que marca Google: la documentación nueva aparece primero, o solo, para Compose.
❌ Exige entender bien conceptos de Kotlin (lambdas, funciones de orden superior) y el **estado** y la **recomposición** antes de poder hacer cosas sencillas.
❌ Oculta parte del funcionamiento del framework, así que es más difícil ver "qué pasa por debajo".

---

## 8. ¿Por qué empezamos con el sistema de vistas?

Lo primero, con honestidad: **Google recomienda Jetpack Compose para las apps nuevas**, y es probable que lo uses en tu vida profesional. No elegimos Views porque Compose sea peor, sino por razones **didácticas y prácticas**:

1. **Encaja con lo que estamos aprendiendo.** El sistema de vistas se apoya directamente en la arquitectura que acabamos de estudiar: el **View System** y el **Resource Manager** del framework, las actividades y su ciclo de vida. Al usarlo, ves esas piezas en acción.

2. **La separación entre diseño y lógica es visible.** Tener el XML por un lado y el código por otro refuerza una idea que ya trabajasteis en 1º con MVC: **separar la presentación del comportamiento**.

3. **Es el estilo imperativo que ya conocéis.** Venís de programar así ("obtengo el objeto, le cambio una propiedad"). El salto al modelo declarativo de Compose es más fácil cuando ya dominas el clásico y entiendes **qué problemas resuelve**.

4. **Lo vais a encontrar en el trabajo.** La mayoría de apps existentes están hechas, total o parcialmente, con Views. En una empresa es muy habitual **mantener** código XML o **migrarlo** a Compose, y para migrar hay que entender ambos.

5. **Hay muchísimo material.** Documentación oficial, ejemplos, preguntas resueltas y libros. La mayor parte del contenido acumulado durante más de quince años está escrito para Views.

6. **No es una elección cerrada.** Views y Compose **conviven**: Google ofrece APIs para meter Compose dentro de un layout XML (`ComposeView`) y vistas clásicas dentro de Compose (`AndroidView`). Lo que aprendas ahora (actividades, recursos, ciclo de vida, permisos…) sirve igual con Compose.

> 🎯 **Resumen:** aprendemos primero el sistema de vistas para **entender cómo funciona Android**, y con esa base Compose será un paso natural, no un salto al vacío.

---

## 9. Practica

1. Crea un proyecto nuevo en Android Studio con la plantilla **"Empty Views Activity"**. La plantilla "Empty Activity" crea un proyecto con Compose.
2. Activa **View Binding** en `build.gradle.kts` y sincroniza el proyecto.
3. Reproduce el layout y la actividad del apartado 5 y ejecútalos en el emulador.
4. Mueve todos los textos a `strings.xml` (incluido el `"Hola, $nombre"`, usando un *string* con parámetro `%1$s`).
5. **Reto:** rehaz el layout con `ConstraintLayout` usando el editor visual, sin anidar ningún layout.

---

## 10. Autoevaluación

1. ¿Qué diferencia hay entre una `View` y un `ViewGroup`? Pon dos ejemplos de cada.
2. ¿Qué diferencia hay entre `match_parent` y `wrap_content`? ¿Y entre `padding` y `margin`?
3. ¿Por qué se usa `sp` para el tamaño del texto y no `dp`?
4. ¿Qué clase genera View Binding para el layout `item_producto.xml`?
5. Indica dos ventajas de View Binding frente a `findViewById()`.
6. Explica con tus palabras la diferencia entre el estilo imperativo (Views) y el declarativo (Compose).
7. ¿Se pueden usar Views y Compose en la misma app? ¿Cómo?

---

## 📚 Para saber más (documentación oficial)

**Sistema de vistas**

- [Diseños en vistas (Layouts in views)](https://developer.android.com/develop/ui/views/layout/declaring-layout?hl=es-419) · Introducción oficial a `View`, `ViewGroup`, XML y atributos.
- [Crear una IU adaptable con ConstraintLayout](https://developer.android.com/develop/ui/views/layout/constraint-layout?hl=es-419)
- [Crear listas dinámicas con RecyclerView](https://developer.android.com/develop/ui/views/layout/recyclerview?hl=es-419)
- [View Binding](https://developer.android.com/topic/libraries/view-binding?hl=es-419) · Configuración, uso en actividades y fragments, y comparación con `findViewById`.
- [Migrar de Kotlin synthetics a View Binding](https://developer.android.com/topic/libraries/view-binding/migration?hl=es-419)
- [Recursos de tipo string](https://developer.android.com/guide/topics/resources/string-resource?hl=es-419)

**Jetpack Compose**

- [Jetpack Compose](https://developer.android.com/compose?hl=es-419) · Página principal del toolkit.
- [Cómo pensar en Compose](https://developer.android.com/develop/ui/compose/mental-model?hl=es-419) · Explica el modelo declarativo frente al imperativo.
- [APIs de interoperabilidad entre Compose y Views](https://developer.android.com/develop/ui/compose/migrate/interoperability-apis?hl=es-419)
