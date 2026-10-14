# Estructura de una aplicación Android

> **Unidad 1 · Tecnologías para aplicaciones en dispositivos móviles**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Nombrar los cuatro componentes de una aplicación Android y explicar para qué sirve cada uno.
- Usar *intents* explícitos e implícitos para comunicar componentes.
- Explicar qué contiene el fichero `AndroidManifest.xml` y por qué es imprescindible.
- Orientarte en la estructura de un proyecto de Android Studio: código, recursos y ficheros de Gradle.
- Distinguir `minSdk`, `targetSdk` y `compileSdk`.

---

## 1. Una app Android no tiene `main()`

En los programas Java o Kotlin que has hecho hasta ahora, la ejecución empieza siempre en el mismo sitio: la función `main()`. **En Android no es así.**

Una aplicación Android está formada por varios **componentes**, y cada uno es un **punto de entrada** por el que el sistema o el usuario pueden "entrar" en la app:

- el usuario pulsa el icono → se abre una pantalla;
- otra app pide compartir una foto → se abre otra pantalla distinta de la misma app;
- el sistema avisa de que la batería está baja → se ejecuta un fragmento de código sin interfaz.

Además, **el sistema** es quien crea, gestiona y destruye esos componentes. Nunca escribirás `MainActivity()` para crear una pantalla: se lo pides a Android y él la crea cuando y como considera.

> 🔗 **Conexión con la arquitectura:** recuerda que cada app se ejecuta en **su propio proceso** y con **su propio usuario Linux** (el *sandbox*). Android aplica el **principio de mínimo privilegio**: por defecto, una app solo accede a lo que necesita. Por eso una app no puede abrir directamente una pantalla de otra app: se lo tiene que **pedir al sistema**, y para eso existen los *intents* (apartado 3).

---

## 2. Los cuatro componentes

| Componente | Qué es | Tiene interfaz | Ejemplo |
|---|---|---|---|
| **Activity** | Una pantalla con la que interactúa el usuario | ✅ Sí | La lista de correos; la pantalla de redactar un correo |
| **Service** | Trabajo que se ejecuta en segundo plano, de larga duración | ❌ No | Reproducir música mientras usas otra app |
| **BroadcastReceiver** | Responde a avisos (*broadcasts*) del sistema o de otras apps | ❌ No | "Batería baja", "el dispositivo ha arrancado" |
| **ContentProvider** | Gestiona datos que se pueden compartir con otras apps de forma controlada | ❌ No | La app de Contactos compartiendo la agenda |

Los cuatro tienen dos cosas en común:

1. **Heredan de una clase del framework** (`Activity`, `Service`, `BroadcastReceiver`, `ContentProvider`).
2. **Tienen un ciclo de vida** que gestiona el sistema: Android decide cuándo se crean y cuándo se destruyen.

### 2.1. Activity

Es el componente **más importante** del curso. Una actividad representa **una pantalla**: carga su layout, responde a lo que hace el usuario y pasa por un ciclo de vida (`onCreate`, `onStart`, `onResume`, `onPause`, `onStop`, `onDestroy`), que estudiaremos en el **punto 5** de la unidad.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
}
```

Una app suele tener **varias actividades**, una por pantalla. Aunque trabajan juntas, cada una es **independiente**: incluso otra app podría abrir una de ellas si se lo permitimos.

### 2.2. Service

Sirve para tareas que deben seguir funcionando aunque el usuario no esté mirando la app. Hay dos tipos:

- **Iniciados (*started*)**: se ejecutan hasta terminar su trabajo, como una descarga o la reproducción de música. Si el usuario es consciente de ellos (música), muestran una notificación y el sistema les da prioridad.
- **Vinculados (*bound*)**: ofrecen una API a otros componentes, que se "conectan" a ellos para usarla.

> ℹ️ Para tareas en segundo plano que deben ejecutarse aunque la app se cierre (sincronizar datos, subir ficheros), hoy se usan herramientas de más alto nivel como **WorkManager** (de Jetpack), que respetan mejor la batería. Conviene que conozcas el concepto de servicio, pero no será lo habitual en nuestras prácticas.

### 2.3. BroadcastReceiver

Permite que la app **reaccione a eventos** aunque no se esté ejecutando: la pantalla se apaga, cambia la conectividad, salta una alarma programada… Su trabajo debe ser **mínimo**. Normalmente solo hace de "portero": recibe el aviso y lanza otra cosa, como una notificación o una tarea.

Es el único componente que se puede registrar **también desde código** (`registerReceiver()`), además de en el manifiesto.

### 2.4. ContentProvider

Publica datos de la app (de una base de datos SQLite, de ficheros…) para que **otras apps** los consulten o modifiquen, si lo permitimos. Cada conjunto de datos se identifica con una **URI** del tipo `content://...`.

Las otras apps no hablan con el *provider* directamente, sino a través de un `ContentResolver`, que añade una capa de seguridad. Es lo que usan apps como WhatsApp para leer tus contactos (con tu permiso).

---

## 3. Intents: el pegamento entre componentes

Un **Intent** es un **mensaje** con el que pides al sistema que haga algo: abrir una pantalla, iniciar un servicio, enviar un aviso. Activa tres de los cuatro componentes (actividades, servicios y receptores; los *content providers* se usan con `ContentResolver`).

### 3.1. Intent explícito

Indica **exactamente qué componente** quieres abrir. Se usa para moverse **dentro de tu propia app**.

```kotlin
// Desde MainActivity: abrir DetalleActivity y pasarle un dato
val intent = Intent(this, DetalleActivity::class.java)
intent.putExtra("NOMBRE", "Ada Lovelace")   // dato extra: clave → valor
startActivity(intent)
```

```kotlin
// En DetalleActivity: recoger el dato
val nombre = intent.getStringExtra("NOMBRE") ?: "Desconocido"
```

### 3.2. Intent implícito

No dice qué componente, sino **qué acción** quieres realizar. El sistema busca qué apps pueden hacerla y, si hay varias, deja elegir al usuario.

```kotlin
// Compartir un texto con cualquier app que sepa hacerlo
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "¡Mira qué app he hecho en 2º DAM!")
}

try {
    startActivity(Intent.createChooser(intent, "Compartir con…"))
} catch (e: ActivityNotFoundException) {
    // Ninguna app instalada puede realizar la acción
    Toast.makeText(this, "No hay ninguna app para compartir", Toast.LENGTH_SHORT).show()
}
```

Otros ejemplos de acciones habituales: `ACTION_VIEW` (abrir una web o un mapa), `ACTION_DIAL` (abrir el marcador con un número), `ACTION_SENDTO` (enviar un correo).

> ⚠️ Con un intent implícito **no puedes saber si habrá alguna app** capaz de atenderlo. Por eso hay que capturar `ActivityNotFoundException`: si no, la app se cierra.

### 3.3. Intent filters

¿Cómo sabe el sistema qué apps pueden atender un intent implícito? Porque cada app declara en su manifiesto qué intents acepta cada componente, mediante **`<intent-filter>`**. El sistema compara el intent con los filtros de **todas las apps instaladas**: acción, categoría y tipo de datos.

Así es, por ejemplo, como aparece tu app en el menú "Compartir" de otras apps.

---

## 4. Otras piezas que conviene conocer

No son componentes en sentido estricto, pero aparecerán continuamente:

| Pieza | Qué es |
|---|---|
| **`Context`** | El "contexto" de la app: da acceso a recursos, servicios del sistema, preferencias… `Activity` y `Application` **son** un `Context`. Por eso en `Intent(this, ...)` el `this` es la actividad. |
| **Fragment** | Una porción reutilizable de interfaz, con su propio ciclo de vida, que vive **dentro** de una actividad. |
| **Application** | Un objeto único que existe mientras vive el proceso de la app. Sirve para inicializar cosas globales. |
| **Recursos** | Todo lo que no es código: textos, imágenes, layouts, colores… (apartado 7). |

---

## 5. El manifiesto: `AndroidManifest.xml`

Es el **documento de identidad** de la app. Antes de poder arrancar cualquier componente, el sistema tiene que saber que existe, y lo sabe leyendo este fichero. Lo leen tres actores distintos: las **herramientas de compilación**, el **sistema operativo** y **Google Play**.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Permisos que necesita la app -->
    <uses-permission android:name="android.permission.INTERNET" />

    <!-- Hardware que usa (Google Play filtra los dispositivos que no lo tienen) -->
    <uses-feature android:name="android.hardware.camera.any" android:required="false" />

    <application
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.MiApp">

        <!-- Pantalla principal: se abre al pulsar el icono -->
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- Pantalla secundaria: solo se abre desde nuestra app -->
        <activity
            android:name=".DetalleActivity"
            android:exported="false" />

    </application>
</manifest>
```

### Qué declara el manifiesto

- **Los componentes**: `<activity>`, `<service>`, `<receiver>` y `<provider>`. Si una actividad, servicio o *provider* no está declarado, **el sistema no puede iniciarlo**.
- **Los *intent filters*** de cada componente.
- **Los permisos** que necesita la app (`<uses-permission>`).
- **Las características de hardware y software** que usa (`<uses-feature>`).
- El **icono**, el **nombre** visible y el **tema** de la app.

### Detalles importantes

- **`android:name=".MainActivity"`**: el punto inicial significa "dentro del *namespace* de la app", que se define en el `build.gradle.kts` (apartado 8).
- **`MAIN` + `LAUNCHER`**: esta pareja de acción y categoría hace que la actividad aparezca en el lanzador de apps del móvil.
- **`android:exported`**: indica si **otras apps** pueden abrir ese componente. Desde Android 12 es **obligatorio** indicarlo en todo componente con `<intent-filter>`; si no, la app no se instala. Por seguridad, pon `false` salvo que realmente lo necesites.

> 🔴 **Error típico:** crear una actividad copiando un fichero `.kt` y olvidarse de declararla. La app **compila**, pero al intentar abrirla falla con `ActivityNotFoundException`. Si creas las actividades con ***File → New → Activity***, Android Studio la declara por ti.

---

## 6. Estructura de un proyecto en Android Studio

Un **proyecto** contiene uno o varios **módulos**. El módulo principal se llama **`app`** y contiene el código, los recursos y la configuración de la aplicación.

### 6.1. Vista *Android* (la que aparece por defecto)

Android Studio no muestra los ficheros tal y como están en disco, sino **agrupados por tipo** para facilitar la navegación:

```
app/
├── manifests/
│   └── AndroidManifest.xml          ← el manifiesto
├── kotlin+java/
│   └── com.ejemplo.miapp/
│       ├── MainActivity.kt          ← código fuente
│       └── DetalleActivity.kt
│   └── com.ejemplo.miapp (test)     ← pruebas unitarias
│   └── com.ejemplo.miapp (androidTest) ← pruebas en dispositivo
└── res/                             ← recursos
    ├── drawable/
    ├── layout/
    ├── mipmap/
    ├── values/
    └── xml/

Gradle Scripts/                      ← configuración de compilación
├── build.gradle.kts (Project: MiApp)
├── build.gradle.kts (Module :app)
├── libs.versions.toml
├── gradle.properties
├── settings.gradle.kts
└── local.properties
```

### 6.2. Vista *Project* (la estructura real en disco)

Si cambias a la vista **Project** en el desplegable del panel, verás la estructura real:

```
MiApp/
├── app/
│   ├── build/                 ← resultados de la compilación (no se toca)
│   ├── src/
│   │   ├── main/              ← código y recursos de la app
│   │   │   ├── AndroidManifest.xml
│   │   │   ├── java/          ← código Kotlin y/o Java
│   │   │   ├── res/           ← recursos
│   │   │   └── assets/        ← ficheros que se copian tal cual (opcional)
│   │   ├── test/              ← pruebas unitarias (se ejecutan en el PC)
│   │   └── androidTest/       ← pruebas instrumentadas (en emulador o dispositivo)
│   └── build.gradle.kts       ← configuración del módulo
├── gradle/
│   └── libs.versions.toml     ← catálogo de versiones
├── build.gradle.kts           ← configuración del proyecto
└── settings.gradle.kts
```

> 💡 La carpeta **`assets/`** guarda ficheros que se incluyen en la app **sin procesar** y se leen como un flujo de bytes (por ejemplo, un JSON de datos o recursos de un juego). A diferencia de `res/`, **no generan identificadores** en la clase `R`.

---

## 7. Los recursos (`res/`) y la clase `R`

Una app Android es mucho más que código. Todo lo relacionado con la presentación se guarda **separado del código** en la carpeta `res/`. Así se puede cambiar la apariencia, traducir la app o adaptarla a otras pantallas **sin tocar el código**.

### 7.1. Tipos de recursos

| Carpeta | Contenido | Ejemplo |
|---|---|---|
| `layout/` | Diseños de pantalla en XML | `activity_main.xml` |
| `values/` | Textos, colores, dimensiones, estilos y temas | `strings.xml`, `colors.xml`, `themes.xml` |
| `drawable/` | Imágenes y gráficos vectoriales | `logo.png`, `ic_add.xml` |
| `mipmap/` | El icono de la app en varias densidades | `ic_launcher.webp` |
| `menu/` | Menús de la barra de la app | `menu_principal.xml` |
| `xml/` | Otros ficheros XML de configuración | `backup_rules.xml` |

### 7.2. La clase `R`

Al compilar, se genera automáticamente la clase **`R`**, con un **identificador numérico** para cada recurso. Así se accede a ellos:

| Desde Kotlin | Desde XML |
|---|---|
| `R.layout.activity_main` | — |
| `R.string.app_name` | `@string/app_name` |
| `R.drawable.logo` | `@drawable/logo` |
| `R.color.primario` | `@color/primario` |
| `R.id.btnAceptar` | `@+id/btnAceptar` (al definirlo) |

```kotlin
val titulo = getString(R.string.app_name)
binding.ivLogo.setImageResource(R.drawable.logo)
```

### 7.3. Recursos alternativos: los calificadores

Se pueden crear **versiones alternativas** de un recurso añadiendo un **calificador** al nombre de la carpeta. El sistema elige automáticamente la más adecuada para cada dispositivo:

| Carpeta | Se usa cuando… |
|---|---|
| `values-gl/strings.xml` | el idioma del dispositivo es gallego |
| `values-en/strings.xml` | el idioma es inglés |
| `layout-land/` | el dispositivo está en horizontal |
| `values-night/` | el tema oscuro está activado |
| `drawable-hdpi/`, `drawable-xxhdpi/`… | según la densidad de la pantalla |

Si no existe una versión específica, se usa la carpeta **sin calificador** (`values/`, `layout/`…), que actúa como valor **por defecto**. Por eso **siempre** debe existir.

> 🔗 Quien hace esta elección es el **Resource Manager** del framework, que vimos en la arquitectura.

---

## 8. Gradle: la configuración de compilación

Android usa **Gradle** como sistema de compilación: descarga las librerías, compila el código, procesa los recursos y genera la app. Sus ficheros están en el grupo **Gradle Scripts**.

| Fichero | Para qué sirve |
|---|---|
| `settings.gradle.kts` | Qué módulos forman el proyecto y de qué repositorios se descargan las librerías |
| `build.gradle.kts` (*Project*) | Configuración común a todos los módulos |
| `build.gradle.kts` (*Module :app*) | **Configuración de la app**. Es el que más tocarás |
| `libs.versions.toml` | **Catálogo de versiones**: reúne en un solo sitio las versiones de las librerías |
| `gradle.properties` | Opciones de Gradle (memoria, opciones de AndroidX…) |
| `local.properties` | Ruta del SDK en tu ordenador. **No se sube a Git** |

> ℹ️ La extensión `.kts` indica que los ficheros están escritos en **Kotlin**. En proyectos antiguos verás `build.gradle` (sin `.kts`), escritos en Groovy: hacen lo mismo con otra sintaxis.

### 8.1. El `build.gradle.kts` del módulo

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
}

android {
    namespace = "com.ejemplo.miapp"       // paquete del código (clase R, nombres del manifiesto)
    compileSdk = 36                       // versión de la API con la que se compila

    defaultConfig {
        applicationId = "com.ejemplo.miapp"   // identificador ÚNICO de la app en el mundo
        minSdk = 24                       // versión mínima en la que se puede instalar
        targetSdk = 36                    // versión para la que está diseñada y probada
        versionCode = 1                   // número interno; se incrementa en cada publicación
        versionName = "1.0"               // versión que ve el usuario
    }

    buildFeatures {
        viewBinding = true                // generar las clases de View Binding
    }
}

dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)
    implementation(libs.androidx.constraintlayout)
    testImplementation(libs.junit)
}
```

> 📌 Los valores concretos de `compileSdk` y `targetSdk` los pone Android Studio al crear el proyecto según la versión más reciente disponible. Los de este ejemplo son orientativos.

### 8.2. Las tres versiones del SDK

Es una pregunta clásica, así que conviene tenerla clara:

| Propiedad | Significado | Qué pasa si… |
|---|---|---|
| **`minSdk`** | La versión **más antigua** de Android en la que se puede instalar la app | Los dispositivos con una versión inferior **no pueden instalarla** |
| **`targetSdk`** | La versión para la que la app está **diseñada y probada** | El sistema aplica los comportamientos (permisos, restricciones) de esa versión |
| **`compileSdk`** | La versión de la API con la que se **compila** | Solo puedes usar en el código las APIs que existen en esa versión |

Normalmente `minSdk ≤ targetSdk ≤ compileSdk`. Elegir el `minSdk` es un **compromiso**: cuanto más bajo, a más dispositivos llegas, pero menos APIs modernas puedes usar sin comprobaciones adicionales. Al crear el proyecto, Android Studio muestra qué porcentaje de dispositivos cubre cada versión.

> ⚠️ Estas versiones se configuran en **Gradle**, no en el manifiesto. Si aparecen en ambos sitios, **Gradle sobrescribe** al manifiesto.

### 8.3. `applicationId` frente a `namespace`

Suelen coincidir, pero no son lo mismo:

- **`applicationId`**: identifica la app **en el dispositivo y en Google Play**. Una vez publicada, **no se puede cambiar**: sería otra app distinta.
- **`namespace`**: es el paquete del **código** (donde se genera la clase `R`) y el que se usa para resolver los nombres abreviados del manifiesto (`.MainActivity`).

---

## 9. Del proyecto al dispositivo

Al compilar, Gradle empaqueta todo en un único fichero:

```
 Código Kotlin ──► bytecode DEX (classes.dex) ──┐
 Recursos (res/) ──► recursos compilados ───────┼──► APK  o  AAB
 AndroidManifest.xml ───────────────────────────┤
 assets/, librerías... ─────────────────────────┘
```

| Formato | Qué es | Para qué se usa |
|---|---|---|
| **APK** (`.apk`) | *Android Package*: el fichero que **se instala** en el dispositivo | Pruebas, emulador, distribución directa |
| **AAB** (`.aab`) | *Android App Bundle*: formato de **publicación**. **No se puede instalar** directamente | Subir a Google Play, que genera APKs optimizados para cada dispositivo |

Al instalar la app, el sistema **lee el manifiesto**, registra sus componentes y le asigna su **propio usuario Linux**. A partir de ahí, cuando se necesita alguno de sus componentes, Android crea el proceso de la app y lo arranca.

---

## 10. Practica

Crea un proyecto nuevo con la plantilla **"Empty Views Activity"** y activa **View Binding**.

1. **Explora el proyecto.** Localiza el manifiesto, `MainActivity.kt`, `activity_main.xml` y `strings.xml`. Cambia a la vista *Project* y encuentra la carpeta `src/main`.
2. **Segunda actividad.** Crea `DetalleActivity` con ***File → New → Activity → Empty Views Activity***. Comprueba qué ha añadido Android Studio al manifiesto.
3. **Intent explícito.** Añade a `MainActivity` un `EditText` y un botón que abra `DetalleActivity` pasándole el texto escrito con `putExtra()`. `DetalleActivity` debe mostrarlo en un `TextView`.
4. **Intent implícito.** Añade un segundo botón que abra `https://developer.android.com` en el navegador (`ACTION_VIEW`) y un tercero que comparta un texto (`ACTION_SEND`). Captura `ActivityNotFoundException`.
5. **Recursos alternativos.** Crea `values-gl/strings.xml` con la traducción al gallego, cambia el idioma del emulador y comprueba el resultado.
6. **Reto.** Elimina la declaración de `DetalleActivity` del manifiesto, ejecuta la app y pulsa el botón. ¿Qué error aparece en *Logcat*? Vuelve a declararla.

---

## 11. Autoevaluación

1. ¿Por qué se dice que una app Android no tiene un único punto de entrada?
2. Indica los cuatro componentes de una app y pon un ejemplo de uso de cada uno.
3. ¿Qué diferencia hay entre un intent explícito y uno implícito? ¿Cuándo usarías cada uno?
4. ¿Por qué hay que capturar `ActivityNotFoundException` al lanzar un intent implícito?
5. ¿Qué ocurre si creas una actividad pero no la declaras en el manifiesto?
6. ¿Qué indica el atributo `android:exported`? ¿Qué valor es más seguro por defecto?
7. ¿Qué es la clase `R`? ¿Cómo accederías desde Kotlin a un texto llamado `bienvenida`?
8. Tu app tiene `values/strings.xml` y `values-en/strings.xml`. ¿Qué textos verá un usuario con el móvil en francés?
9. Explica la diferencia entre `minSdk`, `targetSdk` y `compileSdk`.
10. ¿Qué diferencia hay entre un APK y un AAB?

---

## 📚 Para saber más (documentación oficial)

**Fundamentos**

- [Fundamentos de las aplicaciones](https://developer.android.com/guide/components/fundamentals?hl=es-419) · La referencia principal de este apartado: componentes, manifiesto y recursos.
- [Introducción a las actividades](https://developer.android.com/guide/components/activities/intro-activities?hl=es-419)
- [Intents y filtros de intents](https://developer.android.com/guide/components/intents-filters?hl=es-419)
- [Intents comunes](https://developer.android.com/guide/components/intents-common?hl=es-419) · Catálogo de acciones listas para usar: alarmas, cámara, mapas, teléfono…

**Manifiesto y recursos**

- [Descripción general del manifiesto de la app](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=es-419)
- [Descripción general de los recursos de la app](https://developer.android.com/guide/topics/resources/providing-resources?hl=es-419) · Tipos de recursos y lista completa de calificadores.
- [Localizar tu app](https://developer.android.com/guide/topics/resources/localization?hl=es-419)

**Proyecto y compilación**

- [Descripción general de los proyectos](https://developer.android.com/studio/projects?hl=es-419) · Estructura del proyecto en Android Studio, vistas *Android* y *Project*, módulos.
- [Configurar la compilación](https://developer.android.com/build?hl=es-419) · Ficheros de Gradle en detalle.
- [Control de versiones de la app](https://developer.android.com/studio/publish/versioning?hl=es-419) · `versionCode`, `versionName`, `minSdk` y `targetSdk`.
