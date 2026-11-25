# Implementación de KDE Plasma en Fedora Server

## 1. Introducción

Esta guía explica el proceso de instalación y configuración del entorno gráfico KDE Plasma en Fedora Server.

El objetivo es implementar una interfaz gráfica que permita utilizar el servidor mediante un escritorio visual, manteniendo como configuración predeterminada el inicio en modo consola.

De esta manera, el usuario puede decidir cuándo necesita utilizar el entorno gráfico y cuándo prefiere trabajar directamente desde la terminal, evitando cargar recursos gráficos cuando no sean necesarios.

Durante esta implementación se utilizarán herramientas propias de Fedora y comandos de administración de servicios de Linux.

## 2. Objetivos

- Instalar el entorno gráfico KDE Plasma en Fedora Server.
- Configurar el gestor de inicio de sesión gráfico.
- Mantener el arranque predeterminado del servidor en modo consola.
- Crear comandos cortos para alternar entre la consola y el entorno gráfico.
- Comprobar el funcionamiento de la interfaz.
- Conocer una alternativa para solucionar problemas al cerrar el entorno gráfico.

## 3. Requisitos previos

Antes de comenzar, es necesario contar con:

- Fedora Server instalado y funcionando.
- Acceso a una cuenta de usuario con permisos de administración mediante `sudo`.
- Conexión a Internet para descargar los paquetes necesarios.
- Acceso a la terminal del servidor.
- Espacio disponible en el almacenamiento para instalar KDE Plasma.

**Nota:** La instalación del entorno KDE Plasma puede requerir una descarga considerable de paquetes. El tamaño aproximado dependerá de los componentes que necesite instalar el sistema.

---

# 4. Instalación de KDE Plasma

KDE Plasma es un entorno de escritorio que proporciona una interfaz gráfica para interactuar con Linux mediante ventanas, menús, paneles y aplicaciones.

En Fedora Server, normalmente se trabaja desde la consola. Por eso, primero es necesario instalar los componentes que permitirán utilizar el escritorio gráfico.

### Paso 1. Actualizar el sistema

Antes de instalar nuevos paquetes, se recomienda actualizar el sistema para disponer de las versiones más recientes disponibles en los repositorios configurados.

Ejecuta:

```bash
sudo dnf upgrade --refresh -y
```

**¿Qué hace este comando?**

- `sudo`: ejecuta la operación con permisos de administrador.
- `dnf`: es el administrador de paquetes utilizado por Fedora.
- `upgrade`: actualiza los paquetes instalados.
- `--refresh`: actualiza la información de los repositorios antes de realizar la operación.
- `-y`: acepta automáticamente las confirmaciones.

### Paso 2. Instalar KDE Plasma

Una vez actualizado el sistema, instala el entorno de escritorio:

```bash
sudo dnf install @kde-desktop-environment -y
```

Este comando utiliza DNF para instalar el grupo de paquetes correspondiente al entorno KDE Plasma.

Durante el proceso se descargarán e instalarán los componentes necesarios para disponer de la interfaz gráfica.

**Resultado esperado:** Una vez finalizada la instalación, el sistema contará con los componentes necesarios para iniciar el entorno de escritorio KDE Plasma.

---

# 5. Configuración del inicio y gestor de sesión

Después de instalar KDE Plasma, es necesario configurar cómo se iniciará el entorno gráfico y cuál será el comportamiento predeterminado del servidor.

En esta implementación se utilizará un gestor de inicio gráfico y se mantendrá el arranque del sistema en modo consola.

### ¿Qué es un gestor de inicio de sesión?

Es el componente que presenta una pantalla gráfica para que el usuario pueda autenticarse antes de acceder al escritorio.

En este caso se utilizará Plasma Login Manager, encargado de gestionar el acceso a la sesión gráfica de KDE Plasma.

### Paso 1. Habilitar el gestor de inicio gráfico

Ejecuta:

```bash
sudo systemctl enable plasmalogin.service
```

Este comando configura el servicio para que pueda iniciarse automáticamente cuando corresponda durante el arranque del sistema.

### Paso 2. Establecer el arranque predeterminado en modo consola

Ejecuta:

```bash
sudo systemctl set-default multi-user.target
```

Este comando establece `multi-user.target` como objetivo predeterminado de inicio.

**¿Qué significa esto?**

Linux utiliza diferentes objetivos de systemd para definir los servicios y componentes que deben iniciarse.

En este caso:

- `multi-user.target`: inicia el sistema en modo multiusuario sin cargar automáticamente el entorno gráfico.
- `graphical.target`: permite iniciar los servicios necesarios para utilizar un entorno gráfico.

Al establecer `multi-user.target`, Fedora Server iniciará directamente en la consola.

**Importante:** Esto no desinstala KDE Plasma ni impide utilizarlo. Simplemente permite decidir cuándo iniciar la interfaz gráfica.

---

# 6. Crear comandos cortos para entrar y salir (Opcional)

Para facilitar el uso diario del servidor, se pueden crear dos alias que permitan cambiar entre los modos de funcionamiento sin escribir comandos largos.

Un alias es un nombre corto que representa un comando o conjunto de comandos.

En esta implementación se crearán:

- `gui`: permite cambiar al entorno gráfico.
- `cli`: permite regresar al modo consola.

### Paso 1. Crear el alias para entrar al entorno gráfico

```bash
echo "alias gui='sudo systemctl isolate graphical.target'" >> ~/.bashrc
```

Este comando agrega una nueva configuración al archivo `.bashrc` del usuario.

Cuando se ejecute `gui`, el sistema cambiará al objetivo gráfico mediante systemd.

### Paso 2. Crear el alias para regresar a la consola

```bash
echo "alias cli='sudo systemctl isolate multi-user.target'" >> ~/.bashrc
```

Este comando agrega el alias `cli`, que permite regresar al modo consola.

### Paso 3. Aplicar los cambios

```bash
source ~/.bashrc
```

Este comando vuelve a cargar el archivo `.bashrc` para que los alias estén disponibles sin tener que cerrar y abrir la sesión.

**Resultado esperado:** El usuario podrá escribir `gui` o `cli` directamente en la terminal para cambiar entre los dos modos.

---

# 7. Cómo entrar y salir en el día a día

Una vez completada la instalación y configuración, el funcionamiento será el siguiente:

### 7.1 Al encender o reiniciar el servidor

El sistema iniciará directamente en la consola mediante `multi-user.target`.

Esto evita que KDE Plasma se cargue automáticamente cada vez que se inicia el servidor.

### 7.2 Para entrar al modo gráfico

Inicia sesión en la consola y ejecuta:

```bash
gui
```

También puedes utilizar el comando completo:

```bash
sudo systemctl isolate graphical.target
```

El sistema cambiará al objetivo gráfico y, si los servicios están correctamente configurados, aparecerá la pantalla de inicio de sesión de KDE Plasma.

Desde allí podrás acceder al escritorio utilizando tu usuario.

### 7.3 Para salir y volver al modo consola

Abre la aplicación Konsole dentro del escritorio KDE Plasma y ejecuta:

```bash
cli
```

También puedes utilizar el comando completo:

```bash
sudo systemctl isolate multi-user.target
```

Esto cambiará el sistema al modo consola y cerrará los servicios gráficos que dependan del objetivo anterior.

**Nota:** Al cambiar de objetivo se puede cerrar la sesión gráfica y las aplicaciones abiertas. Es recomendable guardar el trabajo antes de ejecutar el comando.

### 7.4 Acceso a una consola de emergencia

Si al entrar al entorno gráfico aparece una pantalla sin imagen o no puedes interactuar normalmente con el escritorio, puedes intentar acceder a otra terminal virtual mediante:

`Ctrl + Alt + F3`

Esto permite abrir una consola TTY desde la cual puedes iniciar sesión y ejecutar comandos de administración.

---

# 8. Solución de errores al cerrar la interfaz gráfica

Durante la implementación pueden presentarse situaciones en las que el comando `cli` no regrese correctamente a la consola.

En caso de que el alias original presente problemas en el entorno utilizado, se puede probar una alternativa que detiene el gestor gráfico, termina la sesión del asiento `seat0` y devuelve el control a la consola TTY1.

**Importante:** Esta alternativa es específica para el escenario donde se presentó el problema. No es necesario utilizarla si el comando `cli` original funciona correctamente.

### Paso 1. Eliminar el alias anterior

```bash
sed -i '/alias cli=/d' ~/.bashrc
```

Este comando busca y elimina de `.bashrc` la línea donde se encuentra definido el alias `cli`.

### Paso 2. Crear el nuevo alias

```bash
echo "alias cli='sudo systemctl stop plasmalogin.service && sudo loginctl terminate-seat seat0 2>/dev/null; sudo systemctl restart getty@tty1.service && sudo chvt 1'" >> ~/.bashrc
```

Este nuevo alias realiza varias operaciones:

1. Detiene el servicio `plasmalogin.service`.
2. Intenta terminar la sesión gráfica asociada a `seat0`.
3. Reinicia el servicio `getty@tty1.service`, encargado de proporcionar el inicio de sesión en la primera terminal virtual.
4. Cambia la pantalla activa a TTY1.

El operador `&&` permite ejecutar el siguiente comando únicamente cuando el anterior termina correctamente. El punto y coma separa dos partes de la secuencia.

### Paso 3. Aplicar los cambios

```bash
source ~/.bashrc
```

Una vez aplicado el cambio, el alias `cli` utilizará la nueva secuencia de comandos.

---

# 9. Verificación de la implementación

Después de completar los pasos anteriores, es recomendable comprobar que el sistema quedó configurado correctamente.

Estas verificaciones permiten confirmar que KDE Plasma está instalado, que el gestor gráfico está disponible y que el servidor mantiene el arranque predeterminado en consola.

### 9.1 Comprobar el objetivo de inicio predeterminado

```bash
systemctl get-default
```

**Resultado esperado:**

```text
multi-user.target
```

Esto confirma que el sistema está configurado para iniciar en modo consola.

### 9.2 Comprobar el estado del gestor gráfico

```bash
systemctl status plasmalogin.service
```

Este comando muestra información sobre el servicio Plasma Login Manager.

El estado puede variar dependiendo de si el entorno gráfico está iniciado o detenido. Lo importante es comprobar que el servicio existe y que su configuración corresponde a la instalación realizada.

### 9.3 Comprobar los alias

```bash
alias gui
alias cli
```

Estos comandos permiten verificar que los alias fueron creados correctamente y muestran las instrucciones que ejecutará cada uno.

### 9.4 Probar el cambio entre los modos

Para comprobar el funcionamiento completo:

1. Reinicia el servidor y verifica que aparezca la consola.
2. Inicia sesión con tu usuario.
3. Ejecuta `gui` para entrar a KDE Plasma.
4. Comprueba que aparezca la pantalla de inicio de sesión.
5. Accede al escritorio.
6. Abre Konsole y ejecuta `cli`.
7. Comprueba que el sistema regrese al modo consola.

### 9.5 Cambiar entre la interfaz gráfica y la consola mediante el teclado

También es posible alternar entre la interfaz gráfica y la consola utilizando combinaciones de teclas, sin necesidad de ejecutar comandos:

- **Ctrl + Alt + F3:** permite cambiar a una terminal de texto (TTY), donde se puede iniciar sesión y ejecutar comandos desde la consola.
- **Ctrl + Alt + F2:** permite regresar a la sesión gráfica de KDE Plasma.

Estas combinaciones son útiles cuando se necesita acceder rápidamente a la consola o regresar al escritorio. La tecla de función correspondiente puede variar según la configuración del sistema.

---

# 10. Resultados de la implementación

Al finalizar el procedimiento se obtiene un servidor Fedora con KDE Plasma instalado y disponible para su utilización.

La configuración permite:

- Disponer de un entorno gráfico completo.
- Mantener el inicio predeterminado del servidor en modo consola.
- Iniciar KDE Plasma solamente cuando sea necesario.
- Regresar a la consola mediante un comando corto.
- Administrar el uso de los recursos del servidor según las necesidades del usuario.

De esta manera, se combinan las ventajas de trabajar desde una terminal con la facilidad de utilizar una interfaz gráfica cuando se requiera.

---

# 11. Conclusión

La implementación de KDE Plasma en Fedora Server permite ampliar las formas de interacción con el sistema operativo, incorporando un entorno gráfico sin perder la posibilidad de trabajar directamente desde la consola.

La configuración del objetivo predeterminado de systemd permite controlar cuándo se inicia la interfaz gráfica, mientras que los alias simplifican el cambio entre los diferentes modos de funcionamiento.

Esta configuración resulta útil para usuarios que desean disponer de una interfaz gráfica sin mantenerla activa permanentemente.

---

# 12. Referencias

- [Fedora Documentation](https://docs.fedoraproject.org/)
- [KDE Plasma](https://kde.org/plasma-desktop/)
- [KDE Documentation](https://docs.kde.org/)
- [systemd Documentation](https://www.freedesktop.org/software/systemd/man/latest/systemd.special.html)
