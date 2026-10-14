# Depuración y logs en Android

> **Unidad 1 · Tecnologías para aplicaciones en dispositivos móviles**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Escribir mensajes de log con la clase `Log` y elegir el nivel adecuado.
- Usar la ventana **Logcat** de Android Studio y filtrar sus mensajes.
- Leer la traza de error (*stack trace*) de un cierre inesperado y localizar la línea que lo provoca.
- Usar el **depurador**: puntos de interrupción, ejecución paso a paso e inspección de variables.

---

## 1. Dos herramientas complementarias

Cuando una app no hace lo que esperamos, tenemos dos formas de averiguar qué ocurre:

| Herramienta | Cómo funciona | Cuándo usarla |
|---|---|---|
| **Logs** | El código escribe mensajes en una consola mientras la app se ejecuta con normalidad | Seguir el **orden** en que pasan las cosas (como el ciclo de vida), registrar valores, investigar errores que ya han ocurrido |
| **Depurador** | La app se **detiene** en las líneas que elijamos y podemos examinar el estado en ese instante | Entender **por qué** una variable tiene un valor incorrecto, recorrer el código paso a paso |

En la práctica se combinan: los logs ayudan a localizar **dónde** está el problema y el depurador a entender **qué** pasa exactamente allí.

---

## 2. La clase `Log`

En Android, `println()` no es la forma adecuada de mostrar mensajes. Se usa la clase **`android.util.Log`**, que escribe en el **registro del sistema**, donde Android Studio puede leerlo y filtrarlo.

```kotlin
import android.util.Log

Log.d("MainActivity", "El contador vale $contador")
```

Cada mensaje tiene tres partes:

- **Nivel (prioridad):** indica la importancia del mensaje. Lo marca el método usado (`d` = *debug*).
- **Etiqueta (*tag*):** un texto para identificar de dónde viene el mensaje. Sirve para filtrar.
- **Mensaje:** el texto en sí.

### 2.1. Niveles

| Método | Nivel | Para qué usarlo |
|---|---|---|
| `Log.v()` | **VERBOSE** | Detalle máximo; casi nunca hace falta |
| `Log.d()` | **DEBUG** | Mensajes de depuración durante el desarrollo. **El más habitual** |
| `Log.i()` | **INFO** | Información relevante del funcionamiento normal ("sincronización completada") |
| `Log.w()` | **WARN** | Algo inesperado pero no grave ("no hay conexión, se usan datos guardados") |
| `Log.e()` | **ERROR** | Un error. Se suele pasar también la excepción |
| `Log.wtf()` | **ASSERT** | *What a Terrible Failure*: algo que **nunca** debería ocurrir |

Para registrar una excepción, pásala como tercer parámetro. Así Logcat muestra también su traza:

```kotlin
try {
    val numero = texto.toInt()
} catch (e: NumberFormatException) {
    Log.e(TAG, "El texto '$texto' no es un número", e)
}
```

### 2.2. La etiqueta: una constante por clase

Lo habitual es definir la etiqueta **una vez** como constante y reutilizarla:

```kotlin
private const val TAG = "MainActivity"

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d(TAG, "onCreate")
    }
}
```

> 💡 Usar como etiqueta el **nombre de la clase** permite saber de un vistazo de dónde viene cada mensaje. Para un experimento concreto también vale una etiqueta temática, como `"CicloVida"`.

### 2.3. Buenas prácticas

- **Nunca escribas en el log datos sensibles**: contraseñas, tokens, datos personales. El registro del sistema lo pueden leer otras herramientas conectadas al dispositivo.
- **No llenes el código de logs** "por si acaso": elimina los de depuración cuando ya no los necesites.
- Si usas `println()`, el mensaje también llega a Logcat (con la etiqueta `System.out`), pero sin nivel ni etiqueta propia: **usa `Log`**.

---

## 3. La ventana Logcat

Ábrela con **View → Tool Windows → Logcat** o desde la pestaña **Logcat** de la parte inferior de Android Studio. Muestra en tiempo real los mensajes del **dispositivo o emulador** seleccionado: los nuestros y los de **todo el sistema**, que son muchísimos.

Cada línea muestra, de izquierda a derecha: la fecha y hora, el proceso, la **etiqueta**, el **paquete**, el **nivel** (una letra de color: `D`, `I`, `W`, `E`…) y el **mensaje**.

### 3.1. Filtrar: la barra de búsqueda

La barra superior de Logcat admite **consultas** con el formato `clave:valor`. Al abrirla, suele venir escrito `package:mine`.

| Consulta | Qué muestra |
|---|---|
| `package:mine` | Solo los mensajes de **nuestra app** (los paquetes del proyecto abierto) |
| `tag:CicloVida` | Solo los mensajes con esa etiqueta |
| `level:WARN` | Mensajes de nivel **WARN o superior** (WARN, ERROR, ASSERT) |
| `message:contador` | Mensajes que contienen ese texto |
| `is:crash` | Solo los **cierres inesperados** de apps |
| `age:5m` | Solo los mensajes de los **últimos 5 minutos** |
| `-tag:Choreographer` | **Excluye** esa etiqueta (el `-` niega) |

Se pueden **combinar**: claves distintas se unen con "y", y la misma clave repetida se une con "o".

```
package:mine tag:CicloVida
package:mine level:ERROR
package:mine tag:MainActivity tag:DetalleActivity
is:crash package:mine
```

> 💡 Pulsa **Ctrl + Espacio** en la barra de búsqueda para ver sugerencias de claves y valores.

Otros controles útiles de la ventana:

- **Borrar Logcat** (icono de papelera): limpia los mensajes anteriores para empezar una prueba desde cero.
- **Desplazamiento automático** al final: activado, siempre ves lo último.
- El historial de consultas (desplegable de la barra) permite **reutilizar filtros** anteriores.

---

## 4. Cuando la app se cierra: leer la traza de error

Cuando una app se cierra de forma inesperada, Android escribe en el Logcat una **traza de la pila** (*stack trace*). Aprender a leerla es **la habilidad de depuración más importante**.

### 4.1. Un error de ejemplo

Supongamos que añadimos al contador un `EditText` (`etValor`) y un botón **Establecer** que pone el contador al valor escrito:

```kotlin
binding.btnEstablecer.setOnClickListener {
    contador = binding.etValor.text.toString().toInt()   // ← línea 41
    mostrarContador()
}
```

Si el usuario pulsa **Establecer** con el campo **vacío**, la app se cierra. En el Logcat (filtra con `is:crash package:mine`) aparece algo parecido a esto:

```
FATAL EXCEPTION: main
Process: com.ejemplo.contador, PID: 12345
java.lang.NumberFormatException: For input string: ""
    at java.lang.Integer.parseInt(Integer.java:...)
    at java.lang.Integer.parseInt(Integer.java:...)
    at com.ejemplo.contador.MainActivity.onCreate$lambda$2(MainActivity.kt:41)
    at android.view.View.performClick(View.java:...)
    at ...
```

### 4.2. Cómo leerla

1. **`FATAL EXCEPTION: main`**: la app se ha cerrado por una excepción no capturada en el **hilo principal** (`main`, el de la interfaz).
2. **La línea de la excepción**: `java.lang.NumberFormatException: For input string: ""`. Nos dice **qué** ha pasado: se intentó convertir un texto vacío en número.
3. **Las líneas `at ...`**: la pila de llamadas, de la **más reciente** (arriba) a la más antigua (abajo).
4. **Busca la primera línea con tu paquete** (`com.ejemplo.contador`): `MainActivity.kt:41`. Ahí está el problema. Las líneas de `java.*` y `android.*` son código del sistema que llamó o fue llamado por el nuestro.
5. **Haz clic en el enlace** `MainActivity.kt:41`: Android Studio abre el fichero en esa línea.

> ℹ️ Si la traza incluye líneas **`Caused by:`**, la excepción original es la del **último** `Caused by:`. Las de arriba son excepciones que la "envolvieron".

### 4.3. La solución

```kotlin
binding.btnEstablecer.setOnClickListener {
    val valor = binding.etValor.text.toString().toIntOrNull()
    if (valor == null) {
        binding.etValor.error = getString(R.string.error_numero)
    } else {
        contador = valor
        mostrarContador()
    }
}
```

`toIntOrNull()` devuelve `null` en lugar de lanzar una excepción, y `EditText.error` muestra un aviso junto al campo.

---

## 5. El depurador

El depurador permite **detener la app** en una línea concreta y examinar su estado: el valor de cada variable, qué método llamó a cuál…

### 5.1. Puntos de interrupción (*breakpoints*)

Un **punto de interrupción** marca una línea donde la ejecución se detendrá. Para ponerlo, **haz clic en el margen** izquierdo del editor, junto al número de línea: aparece un **círculo rojo**. Otro clic lo quita.

### 5.2. Ejecutar en modo depuración

En lugar de **Run** (▶), usa **Debug** (el icono del insecto 🐞, o **Shift + F9**). La app se instala y se ejecuta con normalidad hasta que llega a una línea con punto de interrupción. Entonces:

- La app se **congela** en el emulador.
- En Android Studio, la línea queda resaltada y se abre la ventana **Debug**.

> ℹ️ Si la app ya se está ejecutando, puedes conectarle el depurador sin reiniciarla con **Attach Debugger to Android Process** (icono junto a Debug).

### 5.3. La ventana Debug

| Panel | Qué muestra |
|---|---|
| **Frames** | La pila de llamadas: qué método llamó al actual. Igual que una traza de error, pero en vivo |
| **Variables** | El valor de las variables en ese momento: las locales, los parámetros y `this` (la actividad, con todas sus propiedades). Se puede desplegar cada objeto |

### 5.4. Controlar la ejecución

Los atajos son los de **Windows/Linux**; en macOS cambian (se pueden consultar en *Help → Keyboard Shortcuts PDF*).

| Acción | Atajo | Qué hace |
|---|---|---|
| **Step Over** | F8 | Ejecuta la línea actual y se detiene en la **siguiente**. Si la línea llama a un método, lo ejecuta entero sin entrar |
| **Step Into** | F7 | **Entra** dentro del método que se llama en la línea actual |
| **Step Out** | Shift + F8 | Termina el método actual y vuelve al que lo llamó |
| **Resume Program** | F9 | Continúa la ejecución normal hasta el siguiente punto de interrupción |
| **Run to Cursor** | Alt + F9 | Continúa hasta la línea donde está el cursor |
| **Evaluate Expression** | Alt + F8 | Calcula cualquier expresión con los valores actuales (por ejemplo, `contador * 2` o `binding.etValor.text`) |
| **Stop** | Ctrl + F2 | Detiene la app |

### 5.5. Puntos de interrupción avanzados

Haz **clic derecho** sobre el círculo rojo para configurarlo:

- **Condition**: solo se detiene si se cumple una condición, por ejemplo `contador == 5`. Muy útil dentro de bucles o en eventos que se repiten.
- **Suspend** desactivado + **Log/Evaluate and log**: no detiene la app, solo escribe un mensaje en la consola del depurador. Es como añadir un `Log.d` **sin tocar el código**.

---

## 6. ¿Logs o depurador?

| Situación | Mejor opción |
|---|---|
| Ver el **orden** en que se ejecutan los callbacks del ciclo de vida | **Logs**: detener la app con el depurador alteraría el propio ciclo de vida |
| La app se **cierra** | **Logcat** (`is:crash package:mine`) para localizar la línea; después, **depurador** si no queda claro el motivo |
| Una variable tiene un **valor incorrecto** y no sabes por qué | **Depurador**: punto de interrupción y paso a paso |
| Un error que solo ocurre **a veces** | **Logs** para registrar las circunstancias, o un punto de interrupción **condicional** |
| Saber si **se ejecuta** un bloque de código | Cualquiera de los dos: un `Log.d` o un punto de interrupción |

---

## 7. Practica

Trabaja sobre la app contador.

1. **Logs y niveles.** Registra con `Log.d` cada pulsación de **Sumar** y con `Log.i` cada **Reiniciar**, mostrando el valor del contador. Filtra en Logcat solo los mensajes de tu app y después solo los de nivel INFO o superior.
2. **Provocar un error.** Añade el `EditText` y el botón **Establecer** del apartado 4 **sin** controlar el campo vacío. Provoca el cierre, filtra con `is:crash package:mine`, localiza la línea en la traza y haz clic en el enlace.
3. **Corregir.** Aplica la solución con `toIntOrNull()` y comprueba que la app ya no se cierra. Define el texto `error_numero` en `strings.xml` (y en `values-gl`, si tu proyecto tiene traducción).
4. **Depurador.** Pon un punto de interrupción en `contador++`. Ejecuta en modo **Debug**, pulsa **Sumar** y observa el valor de `contador` en el panel **Variables**. Avanza con **Step Over** y comprueba cómo cambia.
5. **Evaluate Expression.** Con la app detenida, evalúa `contador + 100` y `binding.tvContador.text`.
6. **Reto:** convierte el punto de interrupción en **condicional** para que solo se detenga cuando `contador == 5`.

---

## 8. Autoevaluación

1. ¿Por qué se usa la clase `Log` en lugar de `println()`?
2. ¿Qué tres partes tiene un mensaje de log? ¿Qué nivel usarías para registrar una excepción capturada?
3. ¿Qué consulta escribirías en Logcat para ver solo los errores de tu app?
4. En una traza de error, ¿cómo localizas la línea de **tu** código que ha provocado el cierre?
5. ¿Qué indica `FATAL EXCEPTION: main`?
6. ¿Qué diferencia hay entre **Step Over** y **Step Into**?
7. ¿Para qué sirve un punto de interrupción condicional?
8. ¿Por qué para estudiar el ciclo de vida son mejores los logs que el depurador?
9. ¿Qué datos no deben escribirse nunca en el log?

---

## 📚 Para saber más (documentación oficial)

- [Escribir y ver registros con Logcat](https://developer.android.com/studio/debug/logcat?hl=es-419) · Ventana Logcat y sintaxis completa de las consultas.
- [Clase `Log` (referencia)](https://developer.android.com/reference/android/util/Log)
- [Depurar tu app](https://developer.android.com/studio/debug?hl=es-419) · Puntos de interrupción, ventana Debug y ejecución paso a paso.
- [Analizar un seguimiento de pila](https://developer.android.com/studio/debug/stacktraces?hl=es-419)
- [Configurar las opciones para desarrolladores](https://developer.android.com/studio/debug/dev-options?hl=es-419)
- [Atajos de teclado de Android Studio](https://developer.android.com/studio/intro/keyboard-shortcuts?hl=es-419)
