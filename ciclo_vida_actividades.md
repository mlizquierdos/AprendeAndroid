# El ciclo de vida de las actividades

> **Unidad 1 · Tecnologías para aplicaciones en dispositivos móviles**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Nombrar los estados de una actividad y los métodos (*callbacks*) que los conectan.
- Explicar qué conviene hacer, y qué no, en cada método.
- Predecir qué métodos se ejecutan en situaciones habituales: abrir la app, pulsar Inicio, girar la pantalla, navegar entre actividades…
- Guardar y recuperar el estado de una actividad con `onSaveInstanceState()`.

---

## 1. ¿Por qué existe un ciclo de vida?

En un programa de escritorio, el usuario abre la aplicación, la usa y la cierra. En un móvil todo es más complicado:

- El usuario **cambia de app** constantemente.
- Entra una **llamada** en mitad de lo que estaba haciendo.
- **Gira la pantalla**, o cambia el idioma o el tema oscuro.
- El sistema anda **justo de memoria** y necesita cerrar apps que no se están usando.

Android gestiona todo esto por nosotros: **crea, pausa, detiene y destruye** las actividades según lo necesita. Lo que sí debemos hacer es **reaccionar** a esos cambios, y para eso el sistema nos avisa llamando a una serie de métodos: los **callbacks del ciclo de vida**.

> 🔗 **Conexión con la arquitectura:** quien gestiona el ciclo de vida es el **Activity Manager** del framework. Nosotros nunca llamamos a `onCreate()` ni a `onPause()`: **el sistema los llama**, y nosotros los sobrescribimos para añadir nuestro código.

---

## 2. Los estados y los callbacks

```
                 ┌─────────────────┐
                 │ Actividad lanzada│
                 └────────┬────────┘
                          ▼
                     onCreate()
                          │
                ┌─────────▼─────────┐
                │      CREADA       │
                └─────────┬─────────┘
                          ▼
     onRestart() ───►  onStart()
         ▲                │
         │      ┌─────────▼─────────┐
         │      │      VISIBLE      │  (iniciada)
         │      └─────────┬─────────┘
         │                ▼
         │           onResume()  ◄─────────────┐
         │                │                    │
         │      ┌─────────▼─────────┐          │
         │      │   EN PRIMER PLANO │  (reanudada: el usuario interactúa)
         │      └─────────┬─────────┘          │
         │                ▼                    │ el usuario vuelve
         │           onPause()                 │
         │                │                    │
         │      ┌─────────▼─────────┐          │
         │      │      PAUSADA      │ ─────────┘
         │      └─────────┬─────────┘
         │                ▼  (deja de verse)
         │            onStop()
         │                │
         │      ┌─────────▼─────────┐
         └───── │     DETENIDA      │
  el usuario    └─────────┬─────────┘
   vuelve                 ▼  (termina o el sistema la destruye)
                     onDestroy()
                          │
                ┌─────────▼─────────┐
                │    DESTRUIDA      │
                └───────────────────┘
```

Los callbacks van **en parejas**, y cada pareja delimita un periodo:

| Pareja | Periodo | ¿Cuántas veces? |
|---|---|---|
| `onCreate()` ↔ `onDestroy()` | La actividad **existe** | Una vez por instancia |
| `onStart()` ↔ `onStop()` | La actividad **es visible** | Varias veces |
| `onResume()` ↔ `onPause()` | La actividad está **en primer plano** y recibe la interacción del usuario | Muchas veces |

---

## 3. Qué hacer en cada método

| Método | Cuándo se llama | Qué hacer aquí | Qué **no** hacer |
|---|---|---|---|
| **`onCreate()`** | Al crear la actividad. **Obligatorio** sobrescribirlo | Inicializaciones que se hacen **una sola vez**: inflar el layout, preparar los *listeners*, recuperar el estado guardado | Tareas lentas: la pantalla no aparece hasta que termina |
| **`onStart()`** | La actividad pasa a ser **visible** | Preparar lo que mantiene la interfaz actualizada mientras se ve | — |
| **`onResume()`** | La actividad pasa a **primer plano** | Iniciar lo que solo tiene sentido mientras el usuario interactúa: vista previa de la cámara, sensores, animaciones | — |
| **`onPause()`** | **Primera señal** de que el usuario se va (aunque puede que la actividad siga visible) | Pausar o liberar lo que se inició en `onResume()`: sensores, cámara… | **Guardar datos, llamadas de red o accesos a base de datos**: es un método muy breve y puede no dar tiempo |
| **`onStop()`** | La actividad **deja de verse** | Liberar recursos que no se necesitan sin verla; operaciones de cierre algo más pesadas, como **guardar un borrador** | — |
| **`onRestart()`** | Una actividad detenida **va a volver** a mostrarse (justo antes de `onStart()`) | Poco habitual | — |
| **`onDestroy()`** | Antes de destruir la actividad | Liberar lo que no se haya liberado antes | Confiar en que **siempre** se ejecute (ver más abajo) |

> ⚠️ **`onDestroy()` no está garantizado.** Si el sistema necesita memoria, puede **matar el proceso** de una app en segundo plano sin llamar a `onDestroy()`. Por eso **los datos importantes se guardan en `onStop()`** o, mejor aún, en cuanto el usuario los introduce.

### Saber por qué se destruye

`onDestroy()` se llama por dos motivos muy distintos:

- La actividad **termina de verdad**: el usuario la cierra o el código llama a `finish()`.
- El sistema la destruye **temporalmente** para recrearla, por un **cambio de configuración** (giro de pantalla, cambio de idioma, tema oscuro…).

La propiedad **`isFinishing`** permite distinguirlos: vale `true` solo en el primer caso.

### Plantilla para sobrescribir los callbacks

```kotlin
override fun onStart() {
    super.onStart()         // ← siempre llamar a la superclase
    // tu código
}
```

En **todos** los callbacks del ciclo de vida hay que llamar a `super`. Si no, la app falla con una excepción `SuperNotCalledException`.

---

## 4. Ver el ciclo de vida en acción

La mejor forma de entenderlo es **verlo**. Añade este código a la `MainActivity` del contador: cada método escribe un mensaje en el **Logcat** (la consola de Android Studio; la veremos a fondo en el apartado de depuración).

```kotlin
import android.util.Log

private const val TAG = "CicloVida"

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d(TAG, "onCreate (savedInstanceState es ${if (savedInstanceState == null) "null" else "no null"})")
        // ... resto del código del contador ...
    }

    override fun onStart() {
        super.onStart()
        Log.d(TAG, "onStart")
    }

    override fun onResume() {
        super.onResume()
        Log.d(TAG, "onResume")
    }

    override fun onPause() {
        super.onPause()
        Log.d(TAG, "onPause")
    }

    override fun onStop() {
        super.onStop()
        Log.d(TAG, "onStop")
    }

    override fun onRestart() {
        super.onRestart()
        Log.d(TAG, "onRestart")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d(TAG, "onDestroy (isFinishing = $isFinishing)")
    }
}
```

Ejecuta la app, abre la pestaña **Logcat** (abajo en Android Studio) y escribe en el filtro:

```
package:mine tag:CicloVida
```

---

## 5. Escenarios habituales

Estos son los métodos que verás en el Logcat en cada situación:

| Situación | Métodos que se ejecutan |
|---|---|
| **Abrir la app** | `onCreate` → `onStart` → `onResume` |
| **Pulsar Inicio** (o cambiar de app) | `onPause` → `onStop` |
| **Volver a la app** | `onRestart` → `onStart` → `onResume` |
| **Girar la pantalla** | `onPause` → `onStop` → `onDestroy` → `onCreate` → `onStart` → `onResume` |
| **Pulsar Atrás en la actividad principal** (Android 12 o superior) | `onPause` → `onStop` (la app pasa a segundo plano, **no** se destruye) |
| **Pulsar Atrás en la actividad principal** (Android 11 o inferior) | `onPause` → `onStop` → `onDestroy` |
| **Llamar a `finish()`** | `onPause` → `onStop` → `onDestroy` (`isFinishing = true`) |

> ℹ️ **Novedad de Android 12:** al pulsar Atrás en la actividad que se abre desde el icono, el sistema ya no la destruye: la manda a segundo plano, igual que el botón Inicio. Así, al volver, la app se recupera más rápido. Tenlo en cuenta si tus pruebas no coinciden con tutoriales antiguos.

### Navegar entre dos actividades

Si desde `MainActivity` (A) abrimos `DetalleActivity` (B) con un intent, los métodos de las dos se **intercalan**:

```
A.onPause
             B.onCreate
             B.onStart
             B.onResume
A.onStop                    ← A deja de verse cuando B ya está en pantalla
```

Y al pulsar Atrás desde B:

```
             B.onPause
A.onRestart
A.onStart
A.onResume
             B.onStop
             B.onDestroy     ← B se destruye: Atrás la cierra
```

Fíjate en el orden: **la nueva actividad se muestra antes de que la anterior se detenga**. Por eso no conviene dejar tareas pesadas en `onPause()`: retrasarían la aparición de la siguiente pantalla.

---

## 6. El problema del giro de pantalla

En el contador comprobamos que **al girar la pantalla el valor vuelve a 0**. Ahora sabemos por qué:

1. Girar es un **cambio de configuración**.
2. Android **destruye** la actividad (`onDestroy`) y crea **una instancia nueva** (`onCreate`).
3. La nueva instancia empieza con `contador = 0`.

Android lo hace así para que la actividad pueda cargar **recursos distintos** según la nueva configuración: por ejemplo, un layout de `res/layout-land/` en horizontal, o los textos de otro idioma.

> 💡 Algunas vistas **sí conservan su contenido solas**: un `EditText` con `android:id` recupera el texto escrito, porque el framework guarda su estado automáticamente. Lo que **no** se guarda son **nuestras variables**, como `contador`.

---

## 7. Guardar y recuperar el estado: `onSaveInstanceState()`

Antes de destruir una actividad que podría volver a necesitarse, el sistema llama a **`onSaveInstanceState(outState: Bundle)`**. Ahí guardamos en el `Bundle` los datos que queremos conservar. Ese mismo `Bundle` llega después al `onCreate()` de la nueva instancia como `savedInstanceState`.

```kotlin
class MainActivity : AppCompatActivity() {

    companion object {
        // Clave para guardar el contador en el Bundle
        private const val CLAVE_CONTADOR = "contador"
    }

    private lateinit var binding: ActivityMainBinding
    private var contador = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        // (Se omite aquí el código edge-to-edge de la plantilla para abreviar; mantenlo en tu proyecto)

        // Si la actividad se está RECREANDO, recuperamos el valor guardado.
        // Si se crea por primera vez, savedInstanceState es null y nos quedamos con 0.
        contador = savedInstanceState?.getInt(CLAVE_CONTADOR) ?: 0
        mostrarContador()

        binding.btnSumar.setOnClickListener {
            contador++
            mostrarContador()
        }
        binding.btnReiniciar.setOnClickListener {
            contador = 0
            mostrarContador()
        }
    }

    // El sistema lo llama antes de destruir la actividad
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)   // guarda el estado de las vistas (EditText…)
        outState.putInt(CLAVE_CONTADOR, contador)
    }

    private fun mostrarContador() {
        binding.tvContador.text = contador.toString()
    }
}
```

Ahora el contador **sobrevive al giro**. En el Logcat verás que `onSaveInstanceState` aparece **después de `onStop`** (desde Android 9).

### Qué guardar en el `Bundle`

| ✅ Adecuado | ❌ No adecuado |
|---|---|
| Datos **pequeños y simples** del estado de la pantalla: un número, un texto, la pestaña seleccionada, la posición de una lista | Datos grandes (imágenes, listas largas) o datos que deben **persistir** al cerrar la app |

Para lo segundo existen otras herramientas que veremos más adelante:

- **`ViewModel`**: conserva los datos de la pantalla durante los cambios de configuración, sin límite de tamaño.
- **Persistencia** (preferencias, base de datos, ficheros): para datos que deben sobrevivir aunque la app se cierre.

> ℹ️ El estado guardado con `onSaveInstanceState()` **también sobrevive** cuando el sistema mata el proceso de una app en segundo plano y el usuario vuelve a ella. **No sobrevive** si el usuario cierra la app de verdad (por ejemplo, deslizándola desde la pantalla de apps recientes).

---

## 8. Probar situaciones difíciles de provocar

En las **Opciones para desarrolladores** del emulador (*Ajustes → Sistema → Opciones para desarrolladores*; si no aparecen, pulsa siete veces en *Ajustes → Información del teléfono → Número de compilación*) hay una opción muy útil:

- **No mantener actividades** (*Don't keep activities*): destruye cada actividad en cuanto deja de verse. Sirve para comprobar que la app **recupera bien su estado** sin tener que esperar a que el sistema se quede sin memoria.

> ⚠️ Desactívala al terminar las pruebas: con ella activada, el emulador se comporta de forma muy distinta a un móvil normal.

---

## 9. Practica

1. Añade los `Log.d` de todos los callbacks a la app contador (apartado 4).
2. Reproduce **cada situación** de la tabla del apartado 5 y anota en tu cuaderno la secuencia que muestra el Logcat. ¿Coincide con la tabla?
3. Comprueba el valor de `isFinishing` al girar la pantalla y al llamar a `finish()`.
4. Implementa `onSaveInstanceState()` para que el contador sobreviva al giro (apartado 7).
5. Añade a la pantalla un `EditText` **con** id y otro **sin** id. Escribe en los dos y gira la pantalla. ¿Qué ocurre con cada uno? ¿Por qué?
6. **Reto:** añade una segunda actividad y un botón para abrirla. Registra los callbacks de las dos y comprueba el orden intercalado del apartado 5.

---

## 10. Autoevaluación

1. ¿Quién llama a los métodos del ciclo de vida? ¿Puedes llamarlos tú?
2. Indica las tres parejas de callbacks y qué periodo delimita cada una.
3. ¿Qué métodos se ejecutan al girar la pantalla? ¿Por qué Android destruye la actividad en ese caso?
4. ¿Por qué no se deben guardar datos en `onPause()`? ¿Dónde se guardan?
5. ¿Se ejecuta siempre `onDestroy()`? Razona la respuesta.
6. ¿Para qué sirve la propiedad `isFinishing`?
7. ¿Qué cambió en Android 12 respecto al botón Atrás en la actividad principal?
8. Si desde A se abre B, ¿qué se ejecuta antes: `A.onStop()` o `B.onResume()`?
9. ¿Qué tipo de datos es adecuado guardar en `onSaveInstanceState()`? ¿Y cuáles no?
10. ¿Por qué un `EditText` con id conserva su texto al girar la pantalla, pero nuestra variable `contador` no?

---

## 📚 Para saber más (documentación oficial)

- [El ciclo de vida de una actividad](https://developer.android.com/guide/components/activities/activity-lifecycle?hl=es-419)
- [Cambios de estado de una actividad](https://developer.android.com/guide/components/activities/state-changes?hl=es-419) · Qué ocurre en cada situación: cambios de configuración, multiventana, diálogos, botón Atrás…
- [Guardar estados de la IU](https://developer.android.com/topic/libraries/architecture/saving-states?hl=es-419) · Comparación entre `onSaveInstanceState()`, `ViewModel` y persistencia.
- [Controlar los cambios de configuración](https://developer.android.com/guide/topics/resources/runtime-changes?hl=es-419)
- [Tareas y la pila de actividades](https://developer.android.com/guide/components/activities/tasks-and-back-stack?hl=es-419) · Incluye el cambio de comportamiento del botón Atrás en Android 12.
- [Procesos y ciclo de vida de la app](https://developer.android.com/guide/components/activities/process-lifecycle?hl=es-419)
