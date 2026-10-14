# Programación multimedia y dispositivos móviles · Android

Apuntes del módulo **Programación multimedia y dispositivos móviles** de **2º DAM** (Desarrollo de Aplicaciones Multiplataforma).

> 🚧 **Material en construcción.** Los apuntes se irán publicando a medida que avance el curso.

---

## 📚 Contenido

### Unidad 1 · Tecnologías para aplicaciones en dispositivos móviles

| Apuntes | Qué aprenderás |
|---|---|
| [Arquitectura de la plataforma Android](arquitectura_android.md) | Las capas de Android (kernel de Linux, HAL, librerías nativas y ART, framework y aplicaciones) y cómo se ejecuta tu código en el dispositivo |
| [El sistema de vistas (Views + XML)](sistema_de_vistas.md) | Cómo se construyen interfaces con `View`, `ViewGroup`, layouts XML y View Binding, y por qué empezamos con Views en lugar de Jetpack Compose |
| [Estructura de una aplicación Android](estructura_app_android.md) | Los cuatro componentes, los intents, el manifiesto, los recursos y la clase `R`, y los ficheros de Gradle |

Se recomienda leerlos **en este orden**: cada documento se apoya en el anterior.

Cada documento incluye objetivos, explicaciones con ejemplos, una práctica, preguntas de autoevaluación y enlaces a la documentación oficial.

---

## 🛠️ Qué necesitas

- **Android Studio** (versión estable más reciente): [descargar](https://developer.android.com/studio?hl=es-419)
- Un **emulador** configurado desde el *Device Manager* de Android Studio, o un móvil Android con la depuración USB activada
- Conocimientos de **Kotlin** y de **programación orientada a objetos**

## 🧭 Decisiones del curso

- **Lenguaje:** Kotlin.
- **Interfaces:** sistema de vistas (**Views + XML**) con **View Binding**. Al crear un proyecto, usa la plantilla **Empty Views Activity**; la plantilla *Empty Activity* crea un proyecto con Jetpack Compose. El motivo se explica en [El sistema de vistas](sistema_de_vistas.md#8-por-qué-empezamos-con-el-sistema-de-vistas).
- **Configuración de Gradle:** Kotlin DSL (`build.gradle.kts`).

---

## 🔗 Documentación de referencia

- [Android Developers](https://developer.android.com/?hl=es-419): documentación oficial para desarrolladores
- [Android Open Source Project (AOSP)](https://source.android.com/): documentación del sistema operativo
- [Documentación de Kotlin](https://kotlinlang.org/docs/home.html)
