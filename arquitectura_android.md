# Arquitectura de la plataforma Android

> **Unidad 1 · Tecnologías para aplicaciones en dispositivos móviles**
> Módulo: Programación multimedia y dispositivos móviles (2º DAM)

## Objetivos

Al terminar este apartado deberías ser capaz de:

- Nombrar las capas que forman la plataforma Android y su orden.
- Explicar qué hace cada capa y qué relación tiene con las demás.
- Describir el recorrido de tu código Kotlin/Java hasta que se ejecuta en el dispositivo.
- Seguir una petición de una app (por ejemplo, hacer una foto) a través de las capas.

---

## 1. Visión general

Android es una **pila de software (software stack) de código abierto basada en Linux**, pensada para funcionar en dispositivos muy distintos: móviles, tablets, relojes, televisores, coches…

Se organiza en **capas**: cada una ofrece servicios a la de arriba y se apoya en la de abajo. Como desarrolladores, trabajaremos casi siempre en las dos capas superiores, pero conocer las inferiores explica muchas cosas que veremos durante el curso (permisos, ciclo de vida, rendimiento, compatibilidad entre fabricantes…).

```
┌──────────────────────────────────────────────────────────┐
│                     APLICACIONES                         │
│     Apps del sistema (Contactos, Cámara…) y las nuestras │
├──────────────────────────────────────────────────────────┤
│                  JAVA API FRAMEWORK                      │
│  Activity Manager · View System · Content Providers ·    │
│  Resource Manager · Notification Manager · ...           │
├────────────────────────────┬─────────────────────────────┤
│   LIBRERÍAS NATIVAS C/C++  │   ANDROID RUNTIME (ART)     │
│  SQLite · OpenGL ES ·      │   Ejecuta bytecode DEX      │
│  libc · Media · ...        │   + librerías core Java     │
├────────────────────────────┴─────────────────────────────┤
│        CAPA DE ABSTRACCIÓN DE HARDWARE (HAL)             │
│     Cámara · Audio · Sensores · Bluetooth · ...          │
├──────────────────────────────────────────────────────────┤
│                    KERNEL DE LINUX                       │
│   Drivers · Memoria · Procesos · Energía · Binder (IPC)  │
└──────────────────────────────────────────────────────────┘
                         HARDWARE
```

Lo estudiaremos **de abajo arriba**, desde el hardware hasta nuestra app.

---

## 2. Kernel de Linux

Es la **base de toda la plataforma**. Android no reinventa el sistema operativo: reutiliza el núcleo Linux para las tareas de bajo nivel:

- Gestión de **procesos e hilos**.
- Gestión de **memoria**.
- **Drivers** de hardware (pantalla, cámara, Wi-Fi, USB…).
- Gestión de **energía** (fundamental en un dispositivo con batería).
- **Seguridad**: usuarios, permisos de ficheros, SELinux.

Usar Linux tiene dos ventajas claras: Android hereda un modelo de seguridad maduro y los fabricantes pueden escribir drivers para un núcleo que ya conocen.

### Lo que Android añade: Binder

Android incorpora al kernel **Binder**, su mecanismo de **comunicación entre procesos (IPC)**. Cada vez que tu app pide algo al sistema (abrir una actividad, leer la ubicación, mostrar una notificación), esa petición viaja de un proceso a otro a través de Binder.

### Idea clave: el sandbox

Cada app se ejecuta **en su propio proceso** y con **su propio usuario Linux**. Por eso una app no puede leer los datos de otra: el kernel lo impide. Este aislamiento es la base del modelo de permisos que veremos más adelante.

---

## 3. Capa de abstracción de hardware (HAL)

Cada fabricante monta hardware distinto: la cámara de un modelo no se parece a la de otro. La **HAL** resuelve este problema definiendo **interfaces estándar** para cada tipo de componente (cámara, audio, sensores, Bluetooth…), que cada fabricante **implementa** para su hardware concreto.

Cuando una API del framework necesita acceder al hardware, el sistema carga el módulo de la HAL correspondiente. Así, el framework puede decir "abre la cámara" sin saber nada del sensor que hay debajo.

> 💡 **Conexión con POO:** la HAL funciona igual que una **interfaz** en Java/Kotlin. Android define el contrato (`interface Camara`) y cada fabricante aporta su clase que la implementa. El código de arriba programa contra la interfaz, no contra la implementación: es **polimorfismo** aplicado a un sistema operativo.

---

## 4. Librerías nativas y Android Runtime

Estas dos piezas están **al mismo nivel** y se complementan.

### 4.1. Librerías nativas C/C++

Muchos componentes del sistema (incluidos ART y la HAL) están escritos en **C/C++** por rendimiento. Algunos ejemplos:

| Librería | Para qué sirve |
|---|---|
| **SQLite** | Base de datos relacional ligera (la usaremos para persistencia local) |
| **OpenGL ES / Vulkan** | Gráficos 2D y 3D acelerados por hardware |
| **Media Framework** | Reproducción y grabación de audio y vídeo |
| **libc** | Librería estándar de C adaptada a Android |

Normalmente **no las llamamos directamente**: usamos las APIs Java/Kotlin del framework, que por debajo las invocan. Si una app necesita código nativo propio (por ejemplo, un motor de juego), se usa el **NDK** (*Native Development Kit*).

### 4.2. Android Runtime (ART)

**ART es el entorno que ejecuta el código de nuestras apps.** Desde Android 5.0 (API 21), cada app se ejecuta en su propio proceso con **su propia instancia de ART**.

#### Del código fuente al dispositivo

```
 Kotlin / Java  ──►  bytecode JVM (.class)  ──►  bytecode DEX (.dex)  ──►  código máquina
   (tu código)        (compilador)                (herramienta d8)          (ART en el móvil)
                                                  └── va dentro del APK
```

El formato **DEX** (*Dalvik Executable*) es un bytecode diseñado específicamente para Android, optimizado para ocupar poca memoria.

#### Características principales de ART

- **Compilación AOT y JIT**: combina la compilación anticipada (*Ahead-Of-Time*, antes de ejecutar) con la compilación en tiempo de ejecución (*Just-In-Time*), para equilibrar velocidad de arranque, rendimiento y espacio.
- **Recolector de basura (GC) optimizado**: menos pausas, apps más fluidas.
- **Mejor soporte de depuración**: profiler, excepciones más detalladas, informes de fallos.

> 📜 **Un poco de historia:** antes de Android 5.0 el runtime era **Dalvik**. ART lo sustituyó, aunque mantiene el formato DEX (de ahí su nombre).

ART incluye además un conjunto de **librerías core** que proporcionan la mayor parte de la funcionalidad del lenguaje Java (colecciones, E/S, concurrencia…), que es lo que usa el framework.

---

## 5. Java API Framework

Es la capa con la que **trabajaremos a diario**. Todas las funcionalidades del sistema se exponen mediante APIs escritas en Java (que usamos exactamente igual desde Kotlin). Son bloques reutilizables que nos ahorran programarlo todo desde cero:

| Componente | Qué hace | Dónde lo veremos |
|---|---|---|
| **Activity Manager** | Gestiona el ciclo de vida de las apps y la pila de navegación (*back stack*) | Punto 5: actividades y ciclo de vida |
| **View System** | Componentes para construir la interfaz: botones, textos, listas, WebView… | Interfaces de usuario |
| **Resource Manager** | Acceso a recursos que no son código: textos traducidos, imágenes, layouts (carpeta `res/`) | Estructura de una app |
| **Notification Manager** | Mostrar avisos en la barra de estado | Notificaciones |
| **Content Providers** | Acceder a datos de otras apps (p. ej., Contactos) o compartir los nuestros | Persistencia y datos compartidos |
| **Package Manager** | Información sobre las apps instaladas | — |
| **Location Manager** | Acceso a la ubicación | Sensores y localización |

> ⭐ **Importante:** los desarrolladores tenemos acceso a **las mismas APIs** que usan las apps del sistema.

---

## 6. Aplicaciones

En la cima están las apps: las que vienen de serie (correo, SMS, calendario, navegador, contactos…) y las que instalamos o programamos nosotros.

Dos ideas importantes:

1. **Las apps del sistema no tienen un estatus especial** (con alguna excepción, como Ajustes). Por eso una app de terceros puede convertirse en el navegador, la app de SMS o el teclado por defecto.
2. **Las apps se pueden reutilizar entre sí.** Si tu app necesita enviar un SMS o hacer una foto, no hace falta programarlo: puedes pedírselo a la app que ya lo hace (lo veremos con los *intents*).

---

## 7. Juntándolo todo: ¿qué pasa cuando una app hace una foto?

```
App "MiCámara"            ──  pulsas el botón "Hacer foto"
   │
   ▼
Java API Framework        ──  la app usa CameraManager
   │  (Binder: IPC)
   ▼
Servicio de cámara        ──  proceso del sistema que coordina el acceso
   │
   ▼
HAL de la cámara          ──  implementación del fabricante
   │
   ▼
Kernel de Linux           ──  driver de la cámara
   │
   ▼
Hardware                  ──  el sensor captura la imagen
```

La imagen hace el camino inverso hasta llegar a tu app. Fíjate en que **tu código solo "ve" la capa del framework**: todo lo demás queda oculto gracias a las capas de abstracción.

---

## 8. Practica: mira dentro de un APK

1. En Android Studio, compila cualquier proyecto (*Build → Build APK(s)*).
2. Abre el APK con **Build → Analyze APK…** (*APK Analyzer*).
3. Localiza:
   - El fichero **`classes.dex`** → el bytecode DEX que ejecutará ART.
   - La carpeta **`res/`** y el fichero **`resources.arsc`** → los recursos que gestiona el Resource Manager.
   - El **`AndroidManifest.xml`** → la "ficha" de la app que lee el sistema.

---

## 9. No confundir: arquitectura de la plataforma vs. arquitectura de una app

En la documentación también encontrarás **"arquitectura de apps"** (*app architecture*). Es otro tema: trata de **cómo organizar el código de nuestra app** (capa de interfaz, capa de datos, patrones como MVVM o Repository). Lo trabajaremos más adelante en el curso.

---

## 10. Autoevaluación

1. Ordena las capas de abajo arriba.
2. ¿Por qué una app no puede leer los ficheros privados de otra? ¿Qué capa lo garantiza?
3. ¿Qué problema resuelve la HAL? Relaciónalo con el concepto de interfaz en POO.
4. Describe el recorrido desde un fichero `.kt` hasta el código máquina que ejecuta el móvil.
5. ¿Qué diferencia hay entre Dalvik y ART?
6. ¿Qué componente del framework gestiona el ciclo de vida de las actividades?
7. ¿Para qué sirve Binder?

---

## 📚 Para saber más (documentación oficial)

**Visión general**

- [Arquitectura de la plataforma — Android Developers](https://developer.android.com/guide/platform?hl=es-419) · La página de referencia para desarrolladores, con el diagrama oficial de capas (disponible en español).
- [Architecture overview — Android Open Source Project](https://source.android.com/docs/core/architecture) · Visión desde dentro del sistema (AOSP), más técnica: añade capas como los *system services* y distingue tipos de apps.

**Profundizar en cada capa** (AOSP, en inglés)

- [Kernel](https://source.android.com/docs/core/architecture/kernel)
- [Binder (comunicación entre procesos)](https://source.android.com/docs/core/architecture/ipc/binder-overview)
- [Hardware Abstraction Layer (HAL)](https://source.android.com/docs/core/architecture/hal)
- [Android Runtime (ART) y Dalvik](https://source.android.com/docs/core/runtime)
- [Zygote: cómo se arrancan los procesos de las apps](https://source.android.com/docs/core/runtime/zygote)

**Herramientas mencionadas**

- [d8: compilador a bytecode DEX](https://developer.android.com/tools/d8)
- [Android NDK](https://developer.android.com/ndk)
- [Analizar la compilación con el APK Analyzer](https://developer.android.com/studio/debug/apk-analyzer?hl=es-419)

**Para más adelante**

- [Guía de arquitectura de apps](https://developer.android.com/topic/architecture/intro?hl=es-419)
