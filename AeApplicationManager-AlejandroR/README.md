<!--
  README de la actividad AE01_07 - Markdown
  Proyecto: AeApplicationManager (C++ / Qt)
  Autor: Alejandro Rodríguez
  Nota: las fórmulas, la planificación y algunos datos de contexto son inventados
  para cubrir todos los elementos que pide la actividad.
-->

[//]: # (Comentario de Markdown: este bloque no se muestra al visualizar el documento)

<a id="indice"></a>

<div align="center">

<img src="application-desktop/Logo.png" alt="Logo de AeApplicationManager" width="140">

<h1>🗂️ AeApplicationManager</h1>

<p><b>Gestor de aplicaciones de escritorio</b> desarrollado en <b>C++</b> con <b>Qt</b></p>

<img src="https://img.shields.io/badge/C%2B%2B-17-00599C?logo=cplusplus&logoColor=white" alt="C++17">
<img src="https://img.shields.io/badge/Qt-6-41CD52?logo=qt&logoColor=white" alt="Qt 6">
<img src="https://img.shields.io/badge/Build-qmake-orange" alt="qmake">
<img src="https://img.shields.io/badge/Plataforma-Windows-0078D6?logo=windows&logoColor=white" alt="Windows">
<img src="https://img.shields.io/badge/Estado-En%20desarrollo-yellow" alt="Estado">

</div>

---

## 📑 Índice

1. [📌 Introducción](#introduccion)
2. [🧰 Tecnologías y requisitos](#tecnologias)
3. [🏗️ Arquitectura del proyecto](#arquitectura)
4. [✨ Funcionalidades](#funcionalidades)
5. [🚀 Instalación y uso](#instalacion)
6. [📐 Modelo de datos y fórmulas](#modelo-datos)
7. [🗓️ Planificación y tareas](#planificacion)
8. [🖼️ Galería y recursos](#galeria)
9. [🏁 Conclusiones y créditos](#conclusiones)

---

<a id="introduccion"></a>

# 📌 Introducción [⬆️](#indice "Volver al índice")

**AeApplicationManager** es una aplicación de escritorio que permite *descubrir, instalar, desinstalar y marcar como favoritas* un conjunto de aplicaciones, de forma parecida a una pequeña tienda de software.\
El proyecto forma parte del repositorio [AeProyectos_cpp_Qt](https://github.com/AlejandroRodriguez05/AeProyectos_cpp_Qt "Repositorio completo en GitHub"), que agrupa varios proyectos en C++ con Qt.

La aplicación está dividida en dos módulos: una **librería** (`application-core`) con la lógica y los datos, y una **interfaz gráfica** (`application-desktop`) que los muestra al usuario.<br>
Esta separación permite reutilizar el núcleo sin depender de la interfaz.

## 🎯 Objetivos

- Practicar la arquitectura **Model/View** de Qt.
- Separar la lógica de negocio de la interfaz gráfica.
- Simular tareas largas (instalación) sin bloquear la interfaz usando **hilos**.
  - Cada instalación se ejecuta en su propio `QThread`.
  - La interfaz solo recibe señales de progreso.

## 🔤 Formatos de texto utilizados

Este documento utiliza **negrita**, *cursiva*, ***negrita cursiva***, ~~tachado~~ y `código en línea`.

La versión ~~1.0.0~~ de la documentación fue sustituida por esta versión ***definitiva***.

> [!NOTE]
> Este README forma parte de la actividad **AE01_07** de Markdown. Algunos apartados (fórmulas, planificación y roadmap) se han inventado para completar los elementos que exige la tarea.

---

<a id="tecnologias"></a>

# 🧰 Tecnologías y requisitos [⬆️](#indice "Volver al índice")

El proyecto se ha construido con las siguientes herramientas:

| Tecnología |   Versión   | Uso en el proyecto                    |
| :--------- | :---------: | ------------------------------------: |
| C++        |     17      | Lenguaje principal                    |
| Qt Widgets |      6      | Interfaz gráfica de escritorio        |
| qmake      |      Qt     | Sistema de compilación (`.pro`)       |
| JSON       |      —      | Almacenamiento de las aplicaciones    |
| QSS        |      —      | Hoja de estilos de la interfaz        |

## 📋 Requisitos previos

1. **Qt 6** con el módulo *Widgets* instalado.
2. Un compilador compatible con **C++17**:
   1. MSVC (Visual Studio) en Windows.
   2. MinGW o GCC, si se adapta la ruta del JSON.
3. **Qt Creator** (opcional, pero recomendado).

> [!TIP]
> Abre directamente el archivo [`aeapplicationmanager.pro`](aeapplicationmanager.pro "Proyecto raíz de qmake") desde Qt Creator: compilará primero la librería y después la aplicación.

---

<a id="arquitectura"></a>

# 🏗️ Arquitectura del proyecto [⬆️](#indice "Volver al índice")

## 📁 Estructura de carpetas

```text
AeApplicationManager/
├── aeapplicationmanager.pro        # Proyecto raíz (subdirs)
├── application-core/               # Librería con la lógica
│   ├── application.h/.cpp
│   ├── version.h/.cpp
│   ├── applicationdao.h/.cpp
│   ├── applicationmodel.h/.cpp
│   ├── versionmodel.h/.cpp
│   └── installerworker.h/.cpp
└── application-desktop/            # Interfaz gráfica
    ├── main.cpp, mainwindow.*
    ├── applicationmanagerwidget.*
    ├── availableapplicationswidget.*
    ├── installedapplicationswidget.*
    ├── applicationinfodialog.*
    ├── applicationdelegate.*
    ├── filterproxymodel.*
    ├── searchwidget.*
    ├── styles.qss
    └── Resources/applications.json
```

## 🧩 Diagrama de clases (núcleo)

```mermaid
classDiagram
    class Application {
        -int mId
        -QString mName
        -QString mDescription
        -QString mImageUrl
        -QString mExecutableFile
        -QDate mExpirationDate
        -bool mIsLiked
        -bool mIsDownloaded
        -int mProgress
        +name() QString
        +isLiked() bool
        +versions() QList~Version~
    }
    class Version {
        -int mId
        -QString mName
        -double mSize
        -QDate mLastModification
        -QDate mExpirationDate
        -bool mIsInstalled
    }
    class ApplicationDao {
        +applications() vector
        +updateApplication(Application)
    }
    class ApplicationModel {
        +rowCount() int
        +data(index, role) QVariant
        +setData(index, value, role) bool
        +indexForAppId(int) QModelIndex
    }
    class installerWorker {
        +install()
        +progress(QModelIndex, int)
        +finished(QModelIndex)
    }
    Application "1" *-- "many" Version : contiene
    ApplicationDao ..> Application : crea
    ApplicationModel o-- ApplicationDao : usa
    ApplicationModel ..> Application : expone
    installerWorker ..> ApplicationModel : actualiza progreso
```

## 🔄 Flujo de una instalación

```mermaid
flowchart TD
    A([Usuario pulsa Descargar]) --> B{¿Progreso = 0?}
    B -- No --> Z([Ignorar clic])
    B -- Sí --> C[Crear QThread e installerWorker]
    C --> D[moveToThread]
    D --> E[Iniciar hilo]
    E --> F[Bucle de 0 a 100 %]
    F --> G[Emitir señal progress]
    G --> H{¿100 %?}
    H -- No --> F
    H -- Sí --> I[Emitir señal finished]
    I --> J[Actualizar modelo y JSON]
    J --> K([Aplicación instalada])
```

## 📡 Secuencia de señales y slots

```mermaid
sequenceDiagram
    actor U as Usuario
    participant D as ApplicationDelegate
    participant W as AvailableApplicationsWidget
    participant T as QThread
    participant I as installerWorker
    participant M as ApplicationModel
    U->>D: Clic en botón de descarga
    D->>W: isDownloadedButtonClicked(index)
    W->>T: start()
    T->>I: install()
    loop 0 a 100
        I-->>W: progress(index, i)
        W->>M: setData(ProgressRole)
    end
    I-->>W: finished(index)
    W->>M: setData(IsDownloadedRole)
    M-->>U: La vista se repinta
```

## 🔁 Estados de una aplicación

```mermaid
stateDiagram-v2
    [*] --> Disponible
    Disponible --> Instalando : Descargar
    Instalando --> Instalada : Progreso 100 %
    Instalada --> Desinstalando : Papelera
    Desinstalando --> Disponible : Progreso 100 %
    Instalada --> Favorita : Me gusta
    Favorita --> Instalada : Quitar me gusta
```

> [!IMPORTANT]
> Los **tres elementos** clave del patrón Model/View son: el modelo (`ApplicationModel`), la vista (`QListView`) y el delegado (`ApplicationDelegate`), que dibuja cada tarjeta de aplicación.

---

<a id="funcionalidades"></a>

# ✨ Funcionalidades [⬆️](#indice "Volver al índice")

## 🛒 Gestión de aplicaciones

- 📥 **Descargar / instalar** aplicaciones con barra de progreso.
- 🗑️ **Desinstalar** aplicaciones instaladas.
- ❤️ **Marcar favoritas** con el icono del corazón.
- ℹ️ **Ver información** detallada de cada aplicación:
  - Descripción y fecha de caducidad.
  - Lista de versiones con tamaño y fecha de modificación.
  - Instalación o desinstalación por versión.

## 🔍 Búsqueda y filtros

1. Buscar por nombre desde el cuadro de texto superior.
2. Filtrar por **Instaladas** o **Favoritas**.
3. Cambiar entre vista de **cuadrícula** y vista de **menú**.
   - La vista de cuadrícula usa el icono <kbd>grid</kbd>.
   - La vista de menú usa el icono <kbd>menu</kbd>.

## 🔔 Bandeja del sistema

La aplicación no se cierra al pulsar la <kbd>X</kbd>: se **oculta** en el área de notificación.\
Con *doble clic* sobre el icono se restaura la ventana y con el menú contextual (**Salir**) se cierra definitivamente.

<details>
<summary>👀 Ver detalles técnicos de la bandeja del sistema</summary>

<br>

- Se usa `QSystemTrayIcon` con el logo de la aplicación.
- El método `closeEvent` ignora el evento y llama a `hide()`.
- Se desactiva `quitOnLastWindowClosed` para que el proceso siga vivo.

</details>

> [!WARNING]
> Si la ventana está oculta y no se usa el menú **Salir**, el proceso seguirá ejecutándose en segundo plano.

---

<a id="instalacion"></a>

# 🚀 Instalación y uso [⬆️](#indice "Volver al índice")

## ⬇️ Clonar el repositorio

```bash
git clone https://github.com/AlejandroRodriguez05/AeProyectos_cpp_Qt.git
cd AeProyectos_cpp_Qt/AeApplicationManager
```

## 🔨 Compilar

Desde Qt Creator basta con abrir el archivo `.pro` y pulsar **Ejecutar**. Desde la terminal:

```bash
qmake aeapplicationmanager.pro
nmake        # o mingw32-make / make según tu compilador
```

El archivo de proyecto raíz define los dos módulos y la dependencia entre ellos:

```qmake
TEMPLATE = subdirs

SUBDIRS += \
    application-core \
    application-desktop

application-desktop.depends = application-core
```

## 💻 Fragmento de código: simulación de la instalación

El trabajador simula el progreso con un tiempo aleatorio por paso:

```cpp
void installerWorker::install()
{
    quint32 tiempo = QRandomGenerator::global()->bounded(50, 101);

    for (int i = 0; i <= 100; i++) {
        QThread::msleep(tiempo);
        emit progress(mSourceIndex, i);
    }
    emit finished(mSourceIndex);
}
```

## 🧾 Formato de los datos

Las aplicaciones se cargan desde [`applications.json`](application-desktop/Resources/applications.json "Datos de las aplicaciones"):

```json
{
    "id": 1,
    "name": "Cloud",
    "executable_file": "Cloud.exe",
    "expiration_date": "2026-11-11",
    "is_liked": true,
    "is_downloaded": true,
    "versions": [
        { "id": 2, "name": "1.1.0", "size": 1.1, "is_installed": true }
    ]
}
```

## 🎨 Estilos

La apariencia se define en [`styles.qss`](application-desktop/styles.qss "Hoja de estilos de Qt"), por ejemplo el fondo de la lista:

```css
QListView {
    background-color: #454545;
    border: none;
    padding: 10px;
}
```

> [!CAUTION]
> La ruta del JSON (`../Resources/applications.json`) es **relativa** al directorio de ejecución. Si ejecutas la aplicación desde otra carpeta, no encontrará el archivo.

---

<a id="modelo-datos"></a>

# 📐 Modelo de datos y fórmulas [⬆️](#indice "Volver al índice")

## 📊 Aplicaciones incluidas

| Id | Aplicación | Ejecutable  | Caduca     | Versiones |  Tamaño más reciente (MB) |
| -: | :--------- | :---------- | :--------: | :-------: | ------------------------: |
|  1 | Cloud      | `Cloud.exe`  | 2026-11-11 |     2     |                       1.1 |
|  2 | aeMaps     | `Maps.exe`   | 2027-12-15 |     2     |                       1.1 |
|  3 | Scann      | `Scann.exe`  | 2026-12-01 |     2     |                       1.1 |
|  4 | aeSync     | `Sync.exe`   | 2027-03-18 |     2     |                       1.1 |
|  5 | aeTools    | `Tools.exe`  | 2027-05-09 |     2     |                       1.1 |
|  6 | aeViewer   | `Viewer.exe` | 2027-08-21 |     3     |                       3.3 |

## 🧮 Fórmulas matemáticas

### Tiempo de instalación

El bucle del trabajador realiza 101 pasos (de 0 a 100) y cada paso espera $\tau$ milisegundos, con $\tau \in \{50, \dots, 100\}$. El tiempo total es:

$$
T = 101 \cdot \tau \quad \Rightarrow \quad T_{\min} = 5{,}05\ \text{s}, \quad T_{\max} = 10{,}1\ \text{s}
$$

Como $\tau$ se elige de forma uniforme, su valor esperado es $E[\tau] = \frac{50 + 100}{2} = 75\ \text{ms}$ y, por tanto, $E[T] = 7{,}575\ \text{s}$.

### Tamaño total de las versiones instaladas

Si $s_i$ es el tamaño de la versión $i$ y $n$ el número de versiones instaladas:

$$
S_{\text{total}} = \sum_{i=1}^{n} s_i
$$

Con las cinco primeras aplicaciones (una versión instalada de $1{,}1$ MB cada una) se obtiene $S_{\text{total}} = 5 \times 1{,}1 = 5{,}5$ MB.

### Porcentaje de aplicaciones instaladas

$$
P = \frac{N_{\text{instaladas}}}{N_{\text{totales}}} \times 100 = \frac{5}{6} \times 100 \approx 83{,}3\ \%
$$

### Velocidad media de descarga (inventada)

Para una versión de tamaño $s$ (MB) descargada en $T$ segundos, la velocidad es $v = \dfrac{s}{T}$. Con $s = 3{,}3$ MB y $T = E[T] = 7{,}575$ s resulta $v \approx 0{,}44\ \text{MB/s}$.

### Complejidad del filtrado (inventada)

Cada vez que cambia el texto de búsqueda se llama a `filterAcceptsRow` una vez por aplicación, por lo que el coste es lineal: $O(N)$. Si además se ordena la lista, el coste pasa a ser $O(N \log N)$.

## ⏳ Cuenta atrás de caducidad

Los días restantes de una aplicación se calculan como $d = f_{\text{caducidad}} - f_{\text{hoy}}$ y se pueden obtener con `QDate::daysTo()`:

```cpp
int diasRestantes = QDate::currentDate().daysTo(application.expirationDate());
```

---

<a id="planificacion"></a>

# 🗓️ Planificación y tareas [⬆️](#indice "Volver al índice")

## ✅ Lista de tareas

- [x] Separar el proyecto en librería (`application-core`) y GUI (`application-desktop`)
- [x] Cargar las aplicaciones desde un archivo JSON
- [x] Mostrar las aplicaciones con un delegado personalizado
- [x] Instalar y desinstalar con hilos y barra de progreso
- [x] Filtrar por texto, instaladas y favoritas
- [x] Icono en la bandeja del sistema
- [ ] Descargas reales desde un servidor
- [ ] Guardar la configuración del usuario
- [ ] Soporte para Linux y macOS
  - [ ] Ruta del JSON independiente del directorio de ejecución
  - [ ] Pruebas en otros compiladores
- [ ] Añadir pruebas unitarias del modelo

## 📅 Diagrama de Gantt (inventado)

```mermaid
gantt
    title Planificación del proyecto
    dateFormat  YYYY-MM-DD
    section Núcleo
    Clases y DAO              :done, a1, 2026-02-01, 14d
    Modelos Qt                :done, a2, after a1, 10d
    section Interfaz
    Widgets y delegado        :done, b1, after a2, 21d
    Hilos e instalación       :done, b2, after b1, 10d
    section Futuro
    Descargas reales          :active, c1, 2026-12-01, 20d
    Pruebas unitarias         :c2, after c1, 10d
```

---

<a id="galeria"></a>

# 🖼️ Galería y recursos [⬆️](#indice "Volver al índice")


| Qt | C++ |
| :-: | :-: |
| <img src="https://raw.githubusercontent.com/github/explore/main/topics/qt/qt.png" alt="Logo de Qt" width="120"> | <img src="https://raw.githubusercontent.com/github/explore/main/topics/cpp/cpp.png" alt="Logo de C++" width="120"> |

![Badge de Qt](https://img.shields.io/badge/Hecho%20con-Qt-41CD52?logo=qt&logoColor=white "Hecho con Qt")

## 💾 Imágenes locales del proyecto

Iconos de las aplicaciones incluidas (carpeta [`application-desktop/apps`](application-desktop/apps "Carpeta con los iconos")):

<table>
  <tr align="center">
    <td><img src="application-desktop/apps/Cloud.png" alt="Cloud" width="90"><br><b>Cloud</b></td>
    <td><img src="application-desktop/apps/Maps.png" alt="aeMaps" width="90"><br><b>aeMaps</b></td>
    <td><img src="application-desktop/apps/Scann.png" alt="Scann" width="90"><br><b>Scann</b></td>
  </tr>
  <tr align="center">
    <td><img src="application-desktop/apps/Sync.png" alt="aeSync" width="90"><br><b>aeSync</b></td>
    <td><img src="application-desktop/apps/Tools.png" alt="aeTools" width="90"><br><b>aeTools</b></td>
    <td><img src="application-desktop/apps/Viewer.png" alt="aeViewer" width="90"><br><b>aeViewer</b></td>
  </tr>
</table>

Iconos de la interfaz: ![Corazón](application-desktop/Corazon.png "Favorita") ![Corazón seleccionado](application-desktop/CorazonSeleccionado.png "Favorita seleccionada") ![Información](application-desktop/Icon_Info.png "Información") ![Descarga](application-desktop/Icon_Download.png "Descarga")

## 🔗 Enlaces a archivos del proyecto

| Archivo | Descripción |
| :------ | :---------- |
| [`application.h`](application-core/application.h "Clase Application") | Entidad que representa una aplicación |
| [`version.h`](application-core/version.h "Clase Version") | Entidad que representa una versión |
| [`applicationmodel.cpp`](application-core/applicationmodel.cpp "Modelo de lista") | Modelo Qt que expone las aplicaciones |
| [`filterproxymodel.cpp`](application-desktop/filterproxymodel.cpp "Modelo proxy") | Filtros por texto, instaladas y favoritas |
| [`applicationdelegate.cpp`](application-desktop/applicationdelegate.cpp "Delegado") | Dibujo de las tarjetas de la lista |

## 💬 Citas relacionadas

> “Any fool can write code that a computer can understand. Good programmers write code that humans can understand.”
>
> — **Martin Fowler**, *Refactoring*

> “Within C++, there is a much smaller and cleaner language struggling to get out.”
>
> — **Bjarne Stroustrup**, creador de C++

> “Talk is cheap. Show me the code.”
>
> — **Linus Torvalds**

---

<a id="conclusiones"></a>

# 🏁 Conclusiones y créditos [⬆️](#indice "Volver al índice")

Con este proyecto se han aplicado conceptos importantes de **Qt** y **C++ moderno**: arquitectura *Model/View*, delegados personalizados, modelos *proxy*, lectura de JSON y programación con **hilos**.\
Queda como mejora futura conectar la aplicación con descargas reales y ampliar la compatibilidad con otros sistemas operativos.

## 👤 Autor

- **Alejandro Rodríguez** · [@AlejandroRodriguez05](https://github.com/AlejandroRodriguez05 "Perfil de GitHub")

## 📚 Recursos consultados

- [Documentación de Qt](https://doc.qt.io/ "Documentación oficial de Qt")
- [Guía de Markdown de GitHub](https://docs.github.com/es/get-started/writing-on-github "Escribir en GitHub")
- [Mermaid](https://mermaid.js.org/ "Documentación de Mermaid")

<p align="center">
  <sub>AE01_11 · Markdown</sub>
</p>

[⬆️ Volver al índice](#indice "Ir al índice")
