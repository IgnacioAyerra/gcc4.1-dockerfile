# Entornos Docker: `gcc41-env` y `slowbuntu-env`

Este repo contiene dos `Dockerfile` que arman entornos de desarrollo "de laboratorio" para poder compilar y correr programas en condiciones controladas (un compilador viejo, o una máquina limitada en CPU), sin tener que instalar nada raro en tu computadora ni ensuciarla con paquetes antiguos.

Esta guía está pensada para alguien que **nunca usó Docker**. Si ya lo conocés, andá directo a la sección [Uso rápido](#uso-rápido).

---

## 1. ¿Qué es Docker?

Docker es una herramienta que permite empaquetar un programa junto con **todo lo que necesita para funcionar** (sistema operativo base, librerías, herramientas, variables de entorno) en una especie de "caja" llamada **contenedor**.

Pensalo así:

- Tu computadora (Windows, Mac o Linux) es la "casa".
- Docker te permite levantar **habitaciones aisladas** dentro de esa casa, cada una con su propio piso, muebles y reglas, sin que una habitación afecte a la otra ni a la casa en sí.
- Esas "habitaciones" son los **contenedores**.

Con esto conseguimos algo muy útil: si necesitás compilar código con un compilador de hace 20 años (como GCC 4.1), no hace falta que ensucies tu sistema operativo actual instalando versiones viejas y potencialmente incompatibles. En cambio, corrés ese compilador **adentro de un contenedor**, usás tu carpeta de código como si fuera una carpeta compartida, y cuando termina el trabajo, el contenedor desaparece sin dejar rastro en tu máquina.

### Conceptos clave

| Término | Qué es |
|---|---|
| **Imagen (image)** | La "receta" ya cocinada: un sistema de archivos con el sistema operativo, herramientas y configuración lista para usar. Es de solo lectura. |
| **Dockerfile** | El "recetario": un archivo de texto con instrucciones paso a paso para construir una imagen (qué sistema operativo base usar, qué paquetes instalar, etc). |
| **Contenedor (container)** | Una **instancia en ejecución** de una imagen. Es como cuando de una receta (imagen) hacés una torta real (contenedor) que podés usar y después tirar. |
| **`docker build`** | Comando que lee un Dockerfile y genera una imagen. |
| **`docker run`** | Comando que toma una imagen y levanta un contenedor a partir de ella, ejecutando un comando adentro. |
| **Volumen / bind mount (`-v`)** | Forma de "compartir" una carpeta de tu computadora con el contenedor, para que el contenedor pueda leer y escribir tus archivos reales. |

Una vez que instalás Docker una sola vez, no volvés a tocar nada del sistema operativo de tu máquina: todo lo "raro" (compiladores viejos, dependencias antiguas, etc.) queda encerrado dentro de las imágenes.

---

## 2. ¿Qué son estos dos Dockerfiles?

En este repo hay dos entornos distintos, pensados para usarse desde la línea de comandos como si fueran "programas" instalados en tu máquina, aunque en realidad corren dentro de Docker.

### 🔹 `gcc41-env`: compilador GCC 4.1

Este Dockerfile arma una imagen que trae instalado **GCC 4.1**, una versión muy antigua del compilador de C/C++. Se usa para poder compilar código pensado para ese compilador específico (por ejemplo, para reproducir bugs, mantener compatibilidad con código legado, o para materias/ejercicios que piden esa versión puntual), sin tener que instalar un compilador obsoleto directamente en tu sistema operativo actual (lo cual hoy en día es difícil o directamente imposible en distros modernas: ni Ubuntu ni Debian actuales tienen `gcc-4.1` en sus repositorios).

```dockerfile
FROM debian/eol:etch

RUN apt-get update && apt-get install -y gcc-4.1 build-essential
```

- **`FROM debian/eol:etch`**: en vez de partir de un Debian actual, esta imagen parte de **Debian Etch** (versión de 2007), tomada del repositorio `debian/eol` ("eol" = *end of life*, es decir, versiones de Debian que ya no reciben soporte oficial pero se mantienen archivadas para poder seguir usándolas). Se usa esta base tan vieja porque es el sistema donde todavía existe el paquete `gcc-4.1` disponible para instalar; en un Debian o Ubuntu moderno ese paquete ya no existe.
- **`RUN apt-get update && apt-get install -y gcc-4.1 build-essential`**: actualiza la lista de paquetes disponibles e instala `gcc-4.1` (el compilador en sí) junto con `build-essential` (un paquete que trae herramientas básicas para compilar en C/C++, como `make` y las librerías de desarrollo estándar).

Como esta imagen está basada en un sistema operativo archivado (EOL), es normal que sea más lenta para instalar paquetes o que no tenga herramientas modernas como `bash` completo, `curl`, `git`, etc. Para eso está pensada esta imagen: **únicamente** para compilar con GCC 4.1, no como entorno de trabajo general (para eso está `slowbuntu-env`).

### 🔹 `slowbuntu-env`: Ubuntu moderno, pero "lento" a propósito

Este Dockerfile arma una imagen basada en **Ubuntu 22.04**, con las herramientas básicas de cualquier sistema Linux moderno: `bash`, `git`, `curl`, `wget`, `vim`, `nano`, herramientas de red, compresión, etc. Nada raro ni antiguo, es un Ubuntu normal y actualizado.

Lo particular no está en la imagen en sí, sino en **cómo se ejecuta**: se corre limitando la cantidad de CPU disponible (`--cpus="1"`, o sea, usa como máximo el equivalente a 1 núcleo de procesador). Por eso le decimos "slowbuntu" (Ubuntu + lento): sirve para simular una máquina con recursos limitados, por ejemplo para:

- Probar cómo se comporta un programa con poca potencia de cálculo.
- Medir tiempos de ejecución en condiciones "peores" a propósito.
- Simular un entorno más modesto que el de tu computadora de desarrollo.

---

## 3. Instalar Docker

Antes de poder usar cualquiera de los dos entornos, necesitás tener Docker instalado. Se instala **una sola vez** por computadora.

### Windows

1. Instalá **Docker Desktop** desde [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/).
2. Docker Desktop en Windows requiere **WSL2** (Windows Subsystem for Linux). El instalador te va a guiar para activarlo si no lo tenés; si te pide reiniciar, reiniciá.
3. Abrí Docker Desktop y esperá a que el ícono de la ballena en la barra de tareas diga que está corriendo ("Docker Desktop is running").
4. Vas a poder usar los comandos `docker` tanto desde **PowerShell**, **CMD**, como desde **WSL/Git Bash**.

### macOS

1. Instalá **Docker Desktop** desde [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/) (elegí la versión para chip Apple Silicon o Intel según corresponda).
2. Abrí Docker Desktop y esperá a que la ballena en la barra de menú indique que está corriendo.
3. Vas a usar los comandos `docker` desde la Terminal normal.

### Linux (incluye Fedora, como en la captura)

En Fedora, por ejemplo:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

(Cerrá sesión y volvé a entrar para que el usuario quede en el grupo `docker` y no tengas que usar `sudo` en cada comando.)

En Ubuntu/Debian sería con `apt`, siguiendo la [guía oficial](https://docs.docker.com/engine/install/). El nombre del paquete y los repos cambian según la distro, pero el resultado es el mismo: tener el comando `docker` disponible en la terminal.

**Para verificar que quedó bien instalado**, en cualquier sistema operativo corré:

```bash
docker --version
docker run hello-world
```

Si ves un mensaje de bienvenida, Docker está funcionando correctamente.

---

## 4. Construir (build) las imágenes

Antes de poder usar los entornos, hay que "construir" la imagen una vez a partir del Dockerfile correspondiente. Esto descarga la base (Ubuntu) e instala todo lo que dice el Dockerfile. Puede tardar unos minutos la primera vez; las siguientes veces Docker reutiliza lo que ya bajó y es mucho más rápido.

Parados en la carpeta donde está el `Dockerfile` de `slowbuntu-env`:

```bash
docker build -t slowbuntu-env .
```

Y parados en la carpeta donde está el `Dockerfile` de `gcc41-env`:

```bash
docker build -t gcc41-env .
```

- `-t slowbuntu-env` / `-t gcc41-env`: le pone un **nombre** (tag) a la imagen, para poder referirte a ella después sin memorizar un ID largo.
- `.` : le dice a Docker "el Dockerfile está en la carpeta actual".

Estos comandos son **idénticos en Windows, Mac y Linux** (siempre y cuando estés en la carpeta correcta y tengas el `Dockerfile` ahí).

Podés confirmar que las imágenes quedaron creadas con:

```bash
docker images
```

---

## 5. Uso rápido

Una vez construidas las imágenes, hay dos formas de usarlas: **con alias/función** (más cómodo, un solo comando corto) o **escribiendo el comando completo de Docker cada vez** (no requiere configuración previa).

La idea general en ambos casos es la misma: le decimos a Docker "montá mi carpeta actual dentro del contenedor, y ejecutá tal comando ahí adentro". Así, el contenedor ve y modifica tus archivos reales, pero el compilador/entorno que usa es el de la imagen.

### 5.1 Compilar con GCC 4.1 (`gcc41-env`)

**Sin alias**, el comando completo se ve así:

<details open>
<summary><b>Linux / macOS (Terminal o Git Bash en Windows)</b></summary>

```bash
docker run --rm -v $(pwd):/src:z -w /src gcc41-env gcc-4.1 archivo.c -o archivo
```
</details>

<details>
<summary><b>Windows (PowerShell)</b></summary>

```powershell
docker run --rm -v ${PWD}:/src -w /src gcc41-env gcc-4.1 archivo.c -o archivo
```

> En PowerShell no hace falta el sufijo `:z` (eso es específico de Linux con SELinux, como Fedora).
</details>

<details>
<summary><b>Windows (CMD clásica)</b></summary>

```cmd
docker run --rm -v %cd%:/src -w /src gcc41-env gcc-4.1 archivo.c -o archivo
```
</details>

**Qué hace cada parte:**

- `docker run` : levanta un contenedor nuevo a partir de una imagen.
- `--rm` : borra el contenedor automáticamente al terminar (no deja "basura" acumulada).
- `-v $(pwd):/src:z` (o `${PWD}:/src` en Windows) : monta tu carpeta actual dentro del contenedor, en la ruta `/src`. Todo lo que el contenedor escriba en `/src` en realidad lo está escribiendo en tu carpeta real.
- `-w /src` : le dice al contenedor que empiece a trabajar (working directory) dentro de esa carpeta montada.
- `gcc41-env` : el nombre de la imagen que construiste en el paso anterior.
- `gcc-4.1 archivo.c -o archivo` : el comando que se ejecuta *adentro* del contenedor, en este caso, compilar `archivo.c` con GCC 4.1.

**Con el alias configurado** (ver sección 6), el mismo comando se reduce a:

```bash
gcc-4.1 archivo.c -o archivo
```

El resultado (`archivo`, el binario compilado) aparece en tu carpeta real, como si lo hubieras compilado de forma local.

### 5.2 Ejecutar un archivo en `slowbuntu-env`

**Sin alias**, el comando completo:

<details open>
<summary><b>Linux / macOS (Terminal o Git Bash en Windows)</b></summary>

```bash
docker run --rm --cpus="1" -v "$(pwd)":/src:z -w /src slowbuntu-env ./mi_programa arg1 arg2
```
</details>

<details>
<summary><b>Windows (PowerShell)</b></summary>

```powershell
docker run --rm --cpus="1" -v "${PWD}:/src" -w /src slowbuntu-env ./mi_programa arg1 arg2
```
</details>

<details>
<summary><b>Windows (CMD clásica)</b></summary>

```cmd
docker run --rm --cpus="1" -v "%cd%:/src" -w /src slowbuntu-env ./mi_programa arg1 arg2
```
</details>

**Qué hace cada parte (lo que es distinto respecto al ejemplo anterior):**

- `--cpus="1"` : limita el contenedor a usar como máximo el equivalente a 1 núcleo de CPU. Es lo que hace que este entorno sea "lento" a propósito.
- `./mi_programa arg1 arg2` : el programa a ejecutar (tiene que estar dentro de tu carpeta actual) y sus argumentos, si los tiene.

**Con la función configurada** (ver sección 6), el mismo comando se reduce a:

```bash
slowbuntu mi_programa arg1 arg2
```

---

## 6. Configurar el alias / función para no escribir el comando completo cada vez

Un **alias** (o función, cuando necesita lógica extra) es un atajo que le enseñás a tu terminal: cuando escribís una palabra corta, en realidad se ejecuta el comando largo de Docker por detrás.

### Linux (bash) y macOS (con bash o zsh)

Editá el archivo de configuración de tu shell:

- Si usás **bash**: `~/.bashrc` (Linux) o `~/.bash_profile` (Mac, en algunas versiones).
- Si usás **zsh** (default en Mac desde Catalina): `~/.zshrc`.

Se puede abrir así, por ejemplo con `nano`:

```bash
nano ~/.bashrc
```

Y agregar al final:

```bash
alias gcc-4.1='docker run --rm -v $(pwd):/src:z -w /src gcc41-env gcc-4.1'

slowbuntu() {
    local prog="$1"
    shift
    docker run --rm --cpus="1" -v "$(pwd)":/src:z -w /src slowbuntu-env "./$prog" "$@"
}
```

> Nota: `:z` es específico de sistemas con SELinux (como Fedora/RHEL). En Ubuntu o Debian normalmente no hace falta, pero no molesta si lo dejás. En macOS tampoco es necesario, podés sacarlo si preferís (`-v $(pwd):/src`).

Guardá el archivo (en `nano`: `Ctrl+O`, `Enter`, `Ctrl+X`) y aplicá los cambios sin reiniciar la terminal:

```bash
source ~/.bashrc   # o ~/.zshrc si usás zsh
```

A partir de ahora, en cualquier terminal nueva (o en la actual después del `source`), podés usar directamente `gcc-4.1 archivo.c -o archivo` y `slowbuntu mi_programa`.

### Windows (PowerShell)

En PowerShell el equivalente a un alias con lógica se hace con una **función**, guardada en tu "perfil" de PowerShell (un script que se carga automáticamente cada vez que abrís una consola).

1. Comprobá si ya tenés un perfil, y si no, creálo:

```powershell
if (!(Test-Path -Path $PROFILE)) { New-Item -ItemType File -Path $PROFILE -Force }
```

2. Abrilo para editarlo (con notepad, por ejemplo):

```powershell
notepad $PROFILE
```

3. Agregá al final del archivo:

```powershell
function gcc-4.1 {
    docker run --rm -v ${PWD}:/src -w /src gcc41-env gcc-4.1 @args
}

function slowbuntu {
    param([Parameter(Mandatory=$true)][string]$prog)
    docker run --rm --cpus="1" -v "${PWD}:/src" -w /src slowbuntu-env "./$prog" @args
}
```

4. Guardá el archivo y recargá el perfil (o simplemente abrí una consola nueva de PowerShell):

```powershell
. $PROFILE
```

Ahora podés usar `gcc-4.1 archivo.c -o archivo` y `slowbuntu mi_programa` directamente en PowerShell.

> **Alternativa más simple en Windows**: instalar **Git Bash** (viene con [Git for Windows](https://git-scm.com/download/win)) y usar directamente la configuración de bash de la sección de Linux/Mac de arriba (`~/.bashrc`). Es la opción más parecida a lo que ya tenés en Fedora y evita lidiar con la sintaxis de PowerShell.

### Windows (CMD clásica)

CMD no tiene un sistema de alias persistente tan cómodo como bash o PowerShell. Lo más práctico es crear pequeños archivos `.bat` y ponerlos en una carpeta que esté en tu `PATH`. Por ejemplo, `gcc-4.1.bat`:

```bat
@echo off
docker run --rm -v %cd%:/src -w /src gcc41-env gcc-4.1 %*
```

Y `slowbuntu.bat`:

```bat
@echo off
docker run --rm --cpus="1" -v "%cd%:/src" -w /src slowbuntu-env "./%1" %2 %3 %4 %5
```

Guardá ambos archivos en una carpeta (por ejemplo `C:\herramientas`) y agregá esa carpeta a tu variable de entorno `PATH` (Panel de Control → Sistema → Configuración avanzada → Variables de entorno). En general, para CMD se recomienda directamente usar PowerShell o Git Bash en su lugar, ya que es mucho más simple de configurar.

---

## 7. Resumen de comandos

| Acción | Comando |
|---|---|
| Instalar Docker | Ver sección 3 |
| Construir imagen de `gcc41-env` | `docker build -t gcc41-env .` (parado en su carpeta) |
| Construir imagen de `slowbuntu-env` | `docker build -t slowbuntu-env .` (parado en su carpeta) |
| Compilar con GCC 4.1 (con alias) | `gcc-4.1 archivo.c -o archivo` |
| Compilar con GCC 4.1 (sin alias) | `docker run --rm -v $(pwd):/src:z -w /src gcc41-env gcc-4.1 archivo.c -o archivo` |
| Ejecutar programa en slowbuntu (con alias) | `slowbuntu mi_programa arg1 arg2` |
| Ejecutar programa en slowbuntu (sin alias) | `docker run --rm --cpus="1" -v "$(pwd)":/src:z -w /src slowbuntu-env ./mi_programa arg1 arg2` |
| Ver imágenes construidas | `docker images` |
| Borrar una imagen | `docker rmi nombre-de-la-imagen` |

---

## 8. Problemas comunes

- **"docker: command not found" / "no se reconoce como comando"**: Docker no está instalado o no arrancó Docker Desktop (Windows/Mac). Abrí Docker Desktop y esperá a que la ballena indique que está corriendo.
- **Permission denied en Linux**: tu usuario no está en el grupo `docker`. Corré `sudo usermod -aG docker $USER` y volvé a iniciar sesión.
- **El contenedor no ve mis archivos / carpeta vacía**: revisá que estés parado (`cd`) en la carpeta correcta antes de correr el comando, ya que `$(pwd)`, `${PWD}` y `%cd%` toman la carpeta *actual* de la terminal.
- **En Windows, Docker Desktop pide activar WSL2 o virtualización**: activalo desde el instalador o desde "Activar o desactivar las características de Windows" → "Plataforma de máquina virtual" y "Subsistema de Windows para Linux".
