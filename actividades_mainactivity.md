# Actividades: la clase `MainActivity` y su layout

> **Unidad 1 · Tecnologías para aplicaciones en dispositivos móviles**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Explicar qué es una actividad y qué papel tiene dentro de una app.
- Entender línea a línea el código que genera Android Studio en `MainActivity.kt` y `activity_main.xml`.
- Distinguir los espacios de nombres `android:`, `app:` y `tools:` de un layout.
- Construir una actividad sencilla con un layout en `ConstraintLayout`, View Binding y eventos de clic.

---

## 1. ¿Qué es una actividad?

Una **actividad** (`Activity`) es **una pantalla** de la aplicación con la que el usuario interactúa. Es uno de los cuatro componentes de Android y el que más usaremos durante el curso.

Cada actividad tiene dos partes:

| Parte | Fichero | Qué contiene |
|---|---|---|
| **Comportamiento** | `MainActivity.kt` | Una clase Kotlin que hereda de una clase del framework y responde a lo que pasa: el usuario pulsa un botón, la pantalla se crea, la app pasa a segundo plano… |
| **Diseño** | `res/layout/activity_main.xml` | El layout XML con las vistas que se muestran en pantalla |

Recuerda tres ideas del apartado anterior:

- **No creamos las actividades con `MainActivity()`**: las crea el sistema cuando hace falta, normalmente a partir de un *intent*.
- **Toda actividad debe estar declarada en el manifiesto.**
- **El sistema decide cuándo se crea, se pausa o se destruye**: eso es el **ciclo de vida**, que veremos en el siguiente apartado.

---

## 2. ¿De qué clase hereda `MainActivity`?

`MainActivity` no hereda directamente de `Activity`, sino de **`AppCompatActivity`**, que está al final de una cadena de herencia:

```
android.app.Activity                      ← la clase base del framework
   └── androidx.activity.ComponentActivity     ← añade ciclo de vida observable, ViewModel, gestión del botón Atrás…
          └── androidx.fragment.app.FragmentActivity  ← añade soporte para fragments
                 └── androidx.appcompat.app.AppCompatActivity  ← añade compatibilidad con versiones antiguas, temas Material, barra de la app
                        └── MainActivity           ← la nuestra
```

¿Por qué `AppCompatActivity` y no `Activity`? Porque viene de **AndroidX** (las librerías de Jetpack) y ofrece las mismas funcionalidades en **todas las versiones de Android** desde nuestro `minSdk`. Con `Activity` a secas, algunas características solo funcionarían en los móviles más modernos.

> 💡 **Conexión con POO:** es **herencia** pura. `MainActivity` recibe todo el comportamiento de sus superclases y **sobrescribe** (`override`) solo los métodos que necesita, como `onCreate()`.

---

## 3. La `MainActivity` que genera Android Studio

Al crear un proyecto con la plantilla **Empty Views Activity**, Android Studio genera algo parecido a esto (puede variar ligeramente según la versión):

```kotlin
package com.ejemplo.miapp                                   // (1)

import android.os.Bundle
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat

class MainActivity : AppCompatActivity() {                  // (2)

    override fun onCreate(savedInstanceState: Bundle?) {    // (3)
        super.onCreate(savedInstanceState)                  // (4)
        enableEdgeToEdge()                                  // (5)
        setContentView(R.layout.activity_main)              // (6)

        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->  // (7)
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }
    }
}
```

### Línea a línea

| Nº | Código | Explicación |
|---|---|---|
| (1) | `package com.ejemplo.miapp` | El paquete coincide con el `namespace` de `build.gradle.kts`. Por eso el manifiesto puede escribir `.MainActivity` |
| (2) | `class MainActivity : AppCompatActivity()` | Declaramos la clase y heredamos de `AppCompatActivity`. Los paréntesis llaman al constructor de la superclase |
| (3) | `override fun onCreate(savedInstanceState: Bundle?)` | Sobrescribimos el **primer método del ciclo de vida**. El sistema lo llama **una sola vez** al crear la actividad. Aquí va la **inicialización** |
| (4) | `super.onCreate(savedInstanceState)` | Llamamos a la versión de la superclase. **Es obligatorio**: si se omite, la app falla con una excepción |
| (5) | `enableEdgeToEdge()` | La app se dibuja **de borde a borde**, también detrás de la barra de estado y la de navegación |
| (6) | `setContentView(R.layout.activity_main)` | **Infla** el layout: convierte el XML en objetos `View` y los muestra como contenido de la pantalla. `R.layout.activity_main` es el identificador del layout en la clase `R` |
| (7) | `ViewCompat.setOnApplyWindowInsetsListener(...)` | Como ahora el contenido se dibuja detrás de las barras del sistema, este código añade un **relleno (*padding*)** del tamaño de esas barras para que nada quede tapado. Busca la vista raíz por su id: `R.id.main` |

### El parámetro `savedInstanceState: Bundle?`

Un `Bundle` es un contenedor de pares **clave → valor**. Si la actividad se crea por primera vez, vale **`null`** (por eso el tipo es nullable, `Bundle?`). Si el sistema la está **recreando** (por ejemplo, al girar la pantalla), contiene el estado que se guardó. Lo usaremos en el apartado del ciclo de vida.

---

## 4. El layout que genera Android Studio

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### El elemento raíz

- **`androidx.constraintlayout.widget.ConstraintLayout`**: el layout raíz. Se escribe con el **nombre completo** porque viene de una librería de AndroidX, no del framework.
- **`android:id="@+id/main"`**: es el id que usa el código de la plantilla (paso 7). **No lo borres**, o `findViewById(R.id.main)` no compilará.

### Los tres espacios de nombres

| Prefijo | URI | Para qué sirve |
|---|---|---|
| `android:` | `.../apk/res/android` | Atributos **del framework**: `layout_width`, `text`, `id`… |
| `app:` | `.../apk/res-auto` | Atributos **de librerías** (AndroidX, Material). Las restricciones de `ConstraintLayout` van aquí |
| `tools:` | `.../tools` | Atributos **solo para el editor** de Android Studio. **No llegan a la app** compilada |

`tools:context=".MainActivity"` le dice al editor qué actividad usa el layout, para mostrar la vista previa con el tema correcto. Otro atributo muy útil es `tools:text`, que muestra un texto de ejemplo **solo en la vista previa**.

### Las restricciones de `ConstraintLayout`

En `ConstraintLayout`, cada vista se posiciona con **restricciones** que la "atan" al padre o a otras vistas. El patrón es:

```
app:layout_constraint<LadoDeEstaVista>_to<LadoDelOtro>Of="<id o parent>"
```

El `TextView` de la plantilla está atado por sus **cuatro lados** al padre (`parent`): como tira de él en las cuatro direcciones a la vez, queda **centrado** en la pantalla.

> ⚠️ Cada vista necesita, como mínimo, **una restricción horizontal y una vertical**. Si falta alguna, el editor muestra un aviso y, al ejecutar, la vista aparece pegada a la esquina superior izquierda.

### Las tres vistas del editor de layouts

En la esquina superior derecha del editor puedes cambiar entre:

- **Code**: el XML.
- **Split**: el XML y la vista previa a la vez. Es la más práctica para aprender.
- **Design**: el editor visual de arrastrar y soltar, con el panel de **Attributes** y los "tiradores" para crear restricciones.

---

## 5. Ejemplo: una app contador

Vamos a construir una actividad sencilla: un contador con un botón para **sumar** y otro para **reiniciar**.

```
┌──────────────────────────────┐
│          Contador            │
│                              │
│             7                │
│                              │
│   [ Reiniciar ]  [ Sumar ]   │
└──────────────────────────────┘
```

### 5.1. Preparación

1. Crea un proyecto con **Empty Views Activity** llamado `Contador`.
2. Activa **View Binding** en `build.gradle.kts (Module :app)` y sincroniza:

```kotlin
android {
    buildFeatures {
        viewBinding = true
    }
}
```

### 5.2. Los textos (`res/values/strings.xml`)

```xml
<resources>
    <string name="app_name">Contador</string>
    <string name="titulo">Contador</string>
    <string name="btn_sumar">Sumar</string>
    <string name="btn_reiniciar">Reiniciar</string>
</resources>
```

### 5.3. El layout (`res/layout/activity_main.xml`)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <!-- Título: arriba y centrado horizontalmente -->
    <TextView
        android:id="@+id/tvTitulo"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="48dp"
        android:text="@string/titulo"
        android:textSize="28sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <!-- Número: centrado en la pantalla -->
    <TextView
        android:id="@+id/tvContador"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="96sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="7" />

    <!-- Botón Sumar: debajo del número, a la derecha del centro -->
    <Button
        android:id="@+id/btnSumar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_marginTop="32dp"
        android:text="@string/btn_sumar"
        app:layout_constraintStart_toEndOf="@id/guiaCentro"
        app:layout_constraintTop_toBottomOf="@id/tvContador" />

    <!-- Botón Reiniciar: debajo del número, a la izquierda del centro -->
    <Button
        android:id="@+id/btnReiniciar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:layout_marginTop="32dp"
        android:text="@string/btn_reiniciar"
        app:layout_constraintEnd_toStartOf="@id/guiaCentro"
        app:layout_constraintTop_toBottomOf="@id/tvContador" />

    <!-- Guía vertical invisible en el centro de la pantalla -->
    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guiaCentro"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintGuide_percent="0.5" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

Fíjate en tres detalles:

- **`tools:text="7"`** muestra un número en la vista previa del editor, pero en la app el texto lo pone el código.
- Los botones se atan a **otras vistas** (`@id/tvContador`), no solo al padre. Al referirse a un id que ya existe se escribe `@id/`, sin el `+`.
- Una **`Guideline`** es una línea invisible que sirve de referencia. Aquí está al 50 % del ancho y los botones se colocan a cada lado.

### 5.4. La actividad (`MainActivity.kt`)

```kotlin
package com.ejemplo.contador

import android.os.Bundle
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import com.ejemplo.contador.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {

    // Clase generada por View Binding a partir de activity_main.xml
    private lateinit var binding: ActivityMainBinding

    // Estado de la pantalla: el valor actual del contador
    private var contador = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()

        // Inflar el layout con View Binding en lugar de setContentView(R.layout...)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Código de la plantilla, adaptado a View Binding (binding.root es la vista "main")
        ViewCompat.setOnApplyWindowInsetsListener(binding.root) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }

        // Mostrar el valor inicial
        mostrarContador()

        // Eventos: qué hacer cuando se pulsa cada botón
        binding.btnSumar.setOnClickListener {
            contador++
            mostrarContador()
        }

        binding.btnReiniciar.setOnClickListener {
            contador = 0
            mostrarContador()
        }
    }

    // Un único sitio que actualiza la interfaz a partir del estado
    private fun mostrarContador() {
        binding.tvContador.text = contador.toString()
    }
}
```

### 5.5. Qué está pasando

1. El sistema crea `MainActivity` y llama a **`onCreate()`**.
2. Inflamos el layout con View Binding y lo mostramos con `setContentView(binding.root)`.
3. Mostramos el valor inicial del contador.
4. **Registramos los *listeners***: le decimos a cada botón qué código ejecutar cuando se pulse. Ese código **no se ejecuta ahora**, sino cada vez que el usuario pulse.
5. `onCreate()` termina y la actividad queda esperando a que el usuario haga algo.

> 💡 **Buena práctica:** la interfaz se actualiza siempre desde **un único método** (`mostrarContador()`) a partir del **estado** (`contador`). Así nunca se desincronizan. Es la misma idea de separar el **modelo** de la **vista** que trabajasteis con MVC.

### 5.6. Un problema por descubrir 🔍

Ejecuta la app, pulsa **Sumar** varias veces y después **gira el emulador** (botones de rotación de la barra lateral del emulador).

¿Qué le pasa al contador? **Vuelve a 0.** Al girar la pantalla, Android **destruye la actividad y la vuelve a crear**, así que `contador` se inicializa de nuevo. Lo entenderemos y lo solucionaremos en el apartado del **ciclo de vida**.

---

## 6. Practica

1. Reproduce la app contador y ejecútala en el emulador.
2. Abre `activity_main.xml` en la vista **Design**. Selecciona el botón **Sumar** y observa sus restricciones en el panel **Attributes**.
3. Añade un botón **Restar** que reste 1 al contador. Colócalo debajo de los otros dos, centrado.
4. Haz que el contador **no pueda bajar de 0**.
5. **Reto:** cambia el color del número a rojo cuando el contador llegue a 10 o más. Define el color en `res/values/colors.xml` y usa `getColor(R.color.rojo)`.

---

## 7. Autoevaluación

1. ¿Qué dos ficheros forman una actividad y qué contiene cada uno?
2. ¿Por qué `MainActivity` hereda de `AppCompatActivity` y no directamente de `Activity`?
3. ¿Qué ocurre si en `onCreate()` olvidas llamar a `super.onCreate(savedInstanceState)`?
4. ¿Qué hace `setContentView()`? ¿Qué se le pasa cuando usamos View Binding?
5. ¿Cuándo vale `null` el parámetro `savedInstanceState`?
6. ¿Para qué sirven los prefijos `android:`, `app:` y `tools:`? ¿Cuál de ellos no llega a la app compilada?
7. ¿Qué restricciones necesita una vista para quedar centrada en un `ConstraintLayout`?
8. ¿Qué diferencia hay entre `@+id/btnSumar` y `@id/btnSumar`?
9. ¿Cuándo se ejecuta el código que hay dentro de `setOnClickListener { ... }`?

---

## 📚 Para saber más (documentación oficial)

- [Introducción a las actividades](https://developer.android.com/guide/components/activities/intro-activities?hl=es-419)
- [Diseños en vistas (Layouts)](https://developer.android.com/develop/ui/views/layout/declaring-layout?hl=es-419)
- [Crear una IU adaptable con ConstraintLayout](https://developer.android.com/develop/ui/views/layout/constraint-layout?hl=es-419)
- [Crear una IU con el editor de diseño](https://developer.android.com/studio/write/layout-editor?hl=es-419)
- [Referencia de los atributos `tools`](https://developer.android.com/studio/write/tool-attributes?hl=es-419)
- [Mostrar contenido de borde a borde (*edge-to-edge*) en vistas](https://developer.android.com/develop/ui/views/layout/edge-to-edge?hl=es-419)
- [View Binding](https://developer.android.com/topic/libraries/view-binding?hl=es-419)
