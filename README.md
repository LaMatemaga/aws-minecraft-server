# Guía Completa: Cómo Levantar tu Propio Servidor de Minecraft en AWS

Esta guía te enseñará paso a paso cómo crear y configurar un servidor de Minecraft utilizando Amazon Web Services (AWS), específicamente con instancias EC2. Aprenderás a configurar todo lo necesario desde cero, incluyendo consideraciones de costos y optimización.

## Tabla de Contenidos

- [Introducción](#introducción)
- [Análisis de Costos](#análisis-de-costos)
- [Requisitos Previos](#requisitos-previos)
- [Paso 1: Crear una Instancia EC2](#paso-1-crear-una-instancia-ec2)
- [Paso 2: Conectarse a la Instancia](#paso-2-conectarse-a-la-instancia)
- [Paso 3: Instalar Java](#paso-3-instalar-java)
- [Paso 4: Instalar y Configurar Minecraft Server](#paso-4-instalar-y-configurar-minecraft-server)
- [Paso 5: Configurar Elastic IP](#paso-5-configurar-elastic-ip)
- [Paso 6: Conectarse al Servidor desde Minecraft](#paso-6-conectarse-al-servidor-desde-minecraft)
- [Optimizaciones y Mejoras](#optimizaciones-y-mejoras)
- [Mantenimiento y Troubleshooting](#mantenimiento-y-troubleshooting)
- [Recursos Adicionales](#recursos-adicionales)

---

## Introducción

Minecraft es uno de los juegos más populares del mundo, y tener tu propio servidor te permite jugar con amigos en un mundo completamente personalizado. Esta guía está diseñada para ayudarte a crear un servidor profesional usando la infraestructura de AWS, lo que te brinda flexibilidad, escalabilidad y control total.

### ¿Por qué AWS?

- **Escalabilidad**: Puedes ajustar los recursos según tus necesidades
- **Confiabilidad**: Infraestructura de clase mundial
- **Flexibilidad**: Múltiples opciones de configuración y precios
- **Control**: Acceso completo al servidor y su configuración

---

## Análisis de Costos

Antes de comenzar, es importante entender las opciones de precios disponibles en AWS y compararlas con servicios de hosting dedicados.

### Comparación con Servicios de Hosting Tradicionales

Tomando como referencia los precios de **Apex Minecraft Hosting** (apexminecrafthosting.com/pricing):

| RAM | Precio Mensual (Hosting Tradicional) |
|-----|--------------------------------------|
| 2GB | $8 USD |
| 4GB | $15 USD |
| 6GB | $23 USD |
| 8GB | $28 USD |

### Opciones de Instancias EC2

#### 1. **On-Demand Instances** (Bajo Demanda)
- Pagas por hora de uso sin compromisos a largo plazo
- Ideal para: Pruebas, uso irregular, máxima flexibilidad
- Costo aproximado para 8GB RAM: **~$61 USD/mes**

#### 2. **Spot Instances** (Instancias Spot)
- Aprovecha capacidad no utilizada de AWS
- **Ahorro: Hasta 90%** respecto a On-Demand
- Costo aproximado para 8GB RAM: **$18-30 USD/mes**
- **Desventaja**: AWS puede terminar la instancia con 2 minutos de aviso cuando necesite la capacidad
- Más información: [aws.amazon.com/ec2/spot](https://aws.amazon.com/ec2/spot)

**Mitigación de riesgos con Spot**:
- Implementar scripts de respaldo automático
- Usar múltiples zonas de disponibilidad
- Configurar políticas de interrupción y reinicio automático

#### 3. **Reserved Instances** (Instancias Reservadas)
- Compromiso de 1 o 3 años
- **Ahorro: Hasta 72%** respecto a On-Demand
- Ideal para: Servidores permanentes con uso predecible
- Más información: [aws.amazon.com/ec2/pricing/reserved-instances](https://aws.amazon.com/ec2/pricing/reserved-instances)

### Recomendación de Costos

Para un servidor personal con amigos, las mejores opciones son:
1. **Spot Instances** si puedes tolerar interrupciones ocasionales (más económico)
2. **Reserved Instances** si planeas tener el servidor activo 24/7 por largo tiempo
3. **On-Demand** para pruebas o uso muy esporádico

---

## Requisitos Previos

Antes de comenzar, asegúrate de tener:

1. **Cuenta de AWS**: 
   - Regístrate en [aws.amazon.com](https://aws.amazon.com/)
   - Ten a la mano una tarjeta de crédito/débito válida
   - Completa la verificación de identidad

2. **Conocimientos básicos**:
   - Navegación en la consola de AWS
   - Comandos básicos de Linux (deseable pero no esencial)
   - Cliente de Minecraft Java Edition instalado

3. **Herramientas opcionales**:
   - Cliente SSH (PuTTY en Windows, terminal nativa en Mac/Linux)
   - Editor de texto para configuraciones

---

## Paso 1: Crear una Instancia EC2

EC2 (Elastic Compute Cloud) es el servicio de servidores virtuales de AWS. Aquí crearemos nuestra máquina virtual que alojará Minecraft.

### 1.1 Acceder al Servicio EC2

1. Inicia sesión en la **Consola de AWS**: [console.aws.amazon.com](https://console.aws.amazon.com/)
2. En la barra de búsqueda superior, escribe **"EC2"**
3. Haz clic en **"EC2"** para abrir el panel de control

### 1.2 Lanzar una Nueva Instancia

1. Haz clic en el botón naranja **"Launch Instance"** (Lanzar Instancia)
2. En el campo **"Name"**, escribe un nombre descriptivo:
   ```
   Servidor Minecraft FCFM UANL
   ```
   O elige el nombre que prefieras (ej: "Mi Servidor Minecraft", "Minecraft-Amigos", etc.)

### 1.3 Seleccionar el Sistema Operativo

1. En la sección **"Application and OS Images"**:
   - Busca y selecciona **"Ubuntu"**
   - Versión recomendada: **Ubuntu Server 22.04 LTS** o la más reciente disponible
   - Asegúrate de seleccionar la arquitectura: **64-bit (x86)**

**¿Por qué Ubuntu?**
- Es gratuito (elegible para el Free Tier de AWS)
- Amplia comunidad y documentación
- Compatible con todas las versiones de Minecraft Server
- Fácil de mantener y actualizar

### 1.4 Elegir el Tipo de Instancia

El tipo de instancia determina la capacidad de tu servidor (CPU, RAM, red).

**Recomendaciones según número de jugadores**:

| Jugadores | Tipo de Instancia | RAM | vCPUs | Precio aprox. On-Demand/mes |
|-----------|-------------------|-----|-------|------------------------------|
| 1-5 | t2.small | 2 GB | 1 | ~$17 USD |
| 5-10 | t2.medium | 4 GB | 2 | ~$34 USD |
| 10-20 | t2.large | 8 GB | 2 | ~$68 USD |
| 20-50 | t2.xlarge | 16 GB | 4 | ~$135 USD |

**Para esta guía**: Selecciona **t2.large** o **t2.medium** (ambos funcionan bien para grupos pequeños)

### 1.5 Crear y Descargar Key Pair (Par de Claves)

Las Key Pairs son credenciales SSH que te permiten conectarte de forma segura a tu servidor.

1. En la sección **"Key pair (login)"**, haz clic en **"Create new key pair"**
2. Configura el Key Pair:
   - **Key pair name**: `minecraft-fcfm-uanl` (o el nombre que prefieras)
   - **Key pair type**: Selecciona **RSA**
   - **Private key file format**: Selecciona **.pem**
3. Haz clic en **"Create key pair"**
4. **IMPORTANTE**: El archivo `.pem` se descargará automáticamente. **Guárdalo en un lugar seguro** - lo necesitarás para conectarte y no podrás descargarlo nuevamente.

**Nota de seguridad**: Nunca compartas este archivo. Quien lo tenga puede acceder a tu servidor.

### 1.6 Configurar Network Settings (Configuración de Red)

Esta configuración determina quién puede acceder a tu servidor y por qué puertos.

1. En la sección **"Network settings"**, haz clic en **"Edit"**
2. Asegúrate de que **"Create security group"** esté seleccionado
3. Verifica que esté habilitada la regla **"Allow SSH traffic from"**: Anywhere (0.0.0.0/0)
   - Esto es necesario para administrar el servidor

#### Agregar Regla para Minecraft

4. Haz clic en **"Add security group rule"**
5. Configura la nueva regla:
   - **Type**: Custom TCP
   - **Port range**: `25565` (puerto predeterminado de Minecraft)
   - **Source type**: Custom
   - **Source**: `0.0.0.0/0` (permite que cualquiera se conecte)
   
   **Nota de seguridad**: Si quieres restringir el acceso solo a ciertas IPs (por ejemplo, solo tú y tus amigos), puedes especificar las direcciones IP permitidas en lugar de `0.0.0.0/0`.

**Ejemplo de restricción por IP**:
```
123.45.67.89/32  # Solo permite esta IP específica
```

### 1.7 Configurar Almacenamiento

El almacenamiento determina cuánto espacio tendrá tu servidor para el mundo de Minecraft, backups, etc.

1. En la sección **"Configure storage"**:
   - **Size (GiB)**: Cambia el valor a **15 GiB** (o más si planeas tener mundos muy grandes)
   - **Volume type**: gp3 (General Purpose SSD) - valor predeterminado, funciona perfectamente

**Recomendaciones de almacenamiento**:
- **8-15 GB**: Servidor pequeño, 1 mundo
- **20-30 GB**: Servidor mediano, múltiples mundos o mods
- **50+ GB**: Servidor grande con muchos mods, múltiples backups

### 1.8 Lanzar la Instancia

1. Revisa todas las configuraciones en el panel derecho **"Summary"**
2. Haz clic en el botón naranja **"Launch instance"**
3. Espera unos segundos y verás el mensaje **"Successfully launched instance"**
4. Haz clic en el ID de la instancia (comienza con `i-`) para ver los detalles

### 1.9 Verificar el Estado de la Instancia

1. En la página de detalles de la instancia:
   - **Instance state**: Debe decir "Running" (tarda 1-2 minutos)
   - **Status checks**: Espera a que muestre "2/2 checks passed"
2. Toma nota de la **Public IPv4 address** (la necesitarás para conectarte)

---

## Paso 2: Conectarse a la Instancia

Ahora que tu servidor está funcionando, necesitas conectarte para instalar y configurar Minecraft.

### 2.1 Conectarse Usando EC2 Instance Connect (Método Más Fácil)

AWS ofrece una forma sencilla de conectarse directamente desde el navegador.

1. En la página de detalles de tu instancia, haz clic en el botón **"Connect"**
2. En la nueva página, selecciona la pestaña **"EC2 Instance Connect"**
3. Verifica que el **User name** sea `ubuntu`
4. Haz clic en **"Connect"**
5. Se abrirá una nueva ventana con una terminal de Linux

**¡Felicidades!** Ahora estás dentro de tu servidor.

### 2.2 Conectarse Usando SSH (Método Alternativo)

Si prefieres usar tu propia terminal o el método del navegador no funciona:

#### En Mac/Linux:

1. Abre la Terminal
2. Navega a la carpeta donde guardaste el archivo `.pem`:
   ```bash
   cd ~/Downloads  # o la ruta donde lo guardaste
   ```
3. Cambia los permisos del archivo (requerido):
   ```bash
   chmod 400 minecraft-fcfm-uanl.pem
   ```
4. Conéctate al servidor:
   ```bash
   ssh -i "minecraft-fcfm-uanl.pem" ubuntu@TU_IP_PUBLICA
   ```
   Reemplaza `TU_IP_PUBLICA` con la IPv4 pública de tu instancia.

#### En Windows:

**Opción 1: PowerShell (Windows 10/11)**
```powershell
ssh -i "ruta\a\minecraft-fcfm-uanl.pem" ubuntu@TU_IP_PUBLICA
```

**Opción 2: PuTTY**
1. Descarga e instala [PuTTY](https://www.putty.org/)
2. Usa PuTTYgen para convertir el archivo `.pem` a `.ppk`
3. Configura PuTTY con tu IP y el archivo `.ppk`

### 2.3 Verificar la Conexión

Una vez conectado, verifica que estás en el servidor:

```bash
whoami
```

Deberías ver:
```
ubuntu
```

---

## Paso 3: Instalar Java

Minecraft Server requiere Java para funcionar. Instalaremos OpenJDK, una implementación de código abierto de Java.

### 3.1 Actualizar el Sistema

Antes de instalar cualquier software, es importante actualizar el sistema:

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

**Explicación del comando**:
- `sudo`: Ejecuta el comando con privilegios de administrador
- `apt-get update`: Actualiza la lista de paquetes disponibles
- `apt-get upgrade -y`: Instala las actualizaciones (`-y` confirma automáticamente)
- `&&`: Ejecuta el segundo comando solo si el primero tiene éxito

Este proceso puede tardar 2-5 minutos.

### 3.2 Instalar Java 17 (Recomendado para Minecraft 1.18+)

Minecraft 1.18 y versiones posteriores requieren Java 17 o superior.

```bash
sudo apt-get install openjdk-17-jdk -y
```

**Nota**: Si deseas usar versiones más antiguas de Minecraft (1.17 o anteriores), instala Java 11:
```bash
sudo apt-get install openjdk-11-jdk -y
```

### 3.3 (Opcional) Instalar Múltiples Versiones de Java

Si quieres tener flexibilidad para cambiar entre versiones:

```bash
sudo apt-get install openjdk-21-jdk -y
```

Luego puedes alternar entre versiones:

```bash
sudo update-alternatives --config java
```

Verás algo como:
```
Selection    Path                                         Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-21-openjdk-amd64/bin/java   2111      auto mode
  1            /usr/lib/jvm/java-17-openjdk-amd64/bin/java   1711      manual mode
  2            /usr/lib/jvm/java-21-openjdk-amd64/bin/java   2111      manual mode
```

Escribe el número de la versión que desees y presiona Enter.

### 3.4 Verificar la Instalación

Confirma que Java se instaló correctamente:

```bash
java -version
```

Deberías ver algo como:
```
openjdk version "17.0.x" 2024-xx-xx
OpenJDK Runtime Environment (build 17.0.x)
OpenJDK 64-Bit Server VM (build 17.0.x, mixed mode, sharing)
```

---

## Paso 4: Instalar y Configurar Minecraft Server

Ahora viene la parte emocionante: instalar el servidor de Minecraft.

### 4.1 Crear el Directorio del Servidor

Organiza tus archivos creando un directorio dedicado:

```bash
mkdir minecraft
cd minecraft
```

Verifica que estás en el directorio correcto:
```bash
pwd
```

Deberías ver:
```
/home/ubuntu/minecraft
```

Confirma que el directorio está vacío:
```bash
ls
```

No debería mostrar nada.

### 4.2 Descargar el Servidor de Minecraft

Necesitas la versión más reciente del servidor oficial de Minecraft.

1. Visita [minecraft.net/en-us/download/server](https://www.minecraft.net/en-us/download/server)
2. Haz clic derecho en el enlace de descarga y selecciona **"Copy Link Address"**
3. En tu terminal, descarga el archivo:

```bash
wget https://piston-data.mojang.com/v1/objects/95495a7f485eedd84ce928cef5e223b757d2f764/server.jar -O minecraft_server.jar
```

**Nota**: La URL del comando anterior puede estar desactualizada. Usa la URL que copiaste del sitio oficial de Minecraft.

**Explicación del comando**:
- `wget`: Herramienta para descargar archivos
- `-O minecraft_server.jar`: Guarda el archivo con este nombre

### 4.3 Verificar la Descarga

Confirma que el archivo se descargó correctamente:

```bash
ls -lh
```

Deberías ver algo como:
```
total 48M
-rw-rw-r-- 1 ubuntu ubuntu 48M Nov  4 12:00 minecraft_server.jar
```

### 4.4 Primera Ejecución del Servidor

Ejecuta el servidor por primera vez para generar los archivos de configuración:

```bash
java -Xmx2048M -Xms2048M -jar minecraft_server.jar nogui
```

**Explicación de los parámetros**:
- `-Xmx2048M`: Memoria RAM máxima (2048 MB = 2 GB)
- `-Xms2048M`: Memoria RAM inicial (2048 MB = 2 GB)
- `-jar minecraft_server.jar`: Archivo JAR a ejecutar
- `nogui`: No usar interfaz gráfica (no disponible en servidor)

**Ajusta la memoria según tu instancia**:
- **t2.small** (2 GB RAM): `-Xmx1536M -Xms1536M`
- **t2.medium** (4 GB RAM): `-Xmx3072M -Xms3072M`
- **t2.large** (8 GB RAM): `-Xmx6144M -Xms6144M`

El servidor se detendrá automáticamente con el mensaje:
```
You need to agree to the EULA in order to run the server.
```

### 4.5 Aceptar el EULA (Acuerdo de Licencia)

Para usar el servidor, debes aceptar el EULA de Minecraft:

```bash
echo "eula=true" > eula.txt
```

Este comando crea el archivo `eula.txt` y escribe `eula=true` en él.

También puedes hacerlo manualmente:
```bash
nano eula.txt
```

Cambia `eula=false` a `eula=true`, guarda (Ctrl+O) y sal (Ctrl+X).

### 4.6 Iniciar el Servidor de Minecraft

Ahora puedes iniciar el servidor correctamente:

```bash
java -Xmx2048M -Xms2048M -jar minecraft_server.jar nogui
```

**Primera ejecución completa**: Este proceso puede tardar 2-5 minutos mientras genera el mundo. Verás muchos mensajes en pantalla. Espera hasta ver:

```
[Server thread/INFO]: Done (XXs)! For help, type "help"
```

**¡Tu servidor de Minecraft está funcionando!**

### 4.7 (Opcional) Configurar el Servidor

Puedes personalizar muchos aspectos del servidor editando `server.properties`:

```bash
nano server.properties
```

**Configuraciones importantes**:

```properties
# Configuración básica
server-port=25565              # Puerto del servidor
max-players=20                 # Máximo de jugadores
difficulty=normal              # Dificultad: peaceful, easy, normal, hard
gamemode=survival              # Modo de juego: survival, creative, adventure
level-name=world               # Nombre del mundo
motd=Servidor Minecraft AWS    # Mensaje que aparece en el listado

# Configuración de seguridad
white-list=false               # Activa lista blanca (true para restringir acceso)
enforce-whitelist=false        # Fuerza la lista blanca
pvp=true                       # Permite PvP

# Configuración de rendimiento
view-distance=10               # Distancia de visión (menor = mejor rendimiento)
simulation-distance=10         # Distancia de simulación
spawn-protection=16            # Protección del spawn
```

Después de editar, guarda (Ctrl+O) y sal (Ctrl+X).

### 4.8 Reiniciar el Servidor

Si hiciste cambios en la configuración, reinicia el servidor:

1. En la terminal donde corre el servidor, escribe:
   ```
   stop
   ```
2. Espera a que el servidor se detenga completamente
3. Vuelve a iniciarlo:
   ```bash
   java -Xmx2048M -Xms2048M -jar minecraft_server.jar nogui
   ```

### 4.9 Mantener el Servidor Activo (Screen o Systemd)

**Problema**: Si cierras la terminal SSH, el servidor se detendrá.

**Solución 1: Usar Screen** (recomendado para principiantes)

```bash
# Instalar screen
sudo apt-get install screen -y

# Crear una sesión de screen
screen -S minecraft

# Iniciar el servidor dentro de screen
java -Xmx2048M -Xms2048M -jar minecraft_server.jar nogui

# Presiona Ctrl+A, luego D para "separarte" de la sesión
# El servidor seguirá corriendo en segundo plano

# Para volver a conectarte:
screen -r minecraft
```

**Solución 2: Crear un Servicio Systemd** (recomendado para producción)

1. Crea el archivo de servicio:
```bash
sudo nano /etc/systemd/system/minecraft.service
```

2. Pega el siguiente contenido:
```ini
[Unit]
Description=Minecraft Server
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/minecraft
ExecStart=/usr/bin/java -Xmx2048M -Xms2048M -jar minecraft_server.jar nogui
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

3. Guarda y cierra (Ctrl+O, Enter, Ctrl+X)

4. Habilita e inicia el servicio:
```bash
sudo systemctl daemon-reload
sudo systemctl enable minecraft
sudo systemctl start minecraft
```

5. Comandos útiles:
```bash
# Ver estado
sudo systemctl status minecraft

# Detener
sudo systemctl stop minecraft

# Reiniciar
sudo systemctl restart minecraft

# Ver logs
sudo journalctl -u minecraft -f
```

---

## Paso 5: Configurar Elastic IP

Una Elastic IP es una dirección IP estática que puedes asociar con tu instancia. Sin ella, cada vez que detengas y reinicies tu instancia, obtendrá una IP diferente.

### 5.1 ¿Por Qué Necesitas una Elastic IP?

**Sin Elastic IP**:
- IP cambia cada vez que reinicias la instancia
- Los jugadores deben actualizar la dirección del servidor constantemente

**Con Elastic IP**:
- La IP nunca cambia
- Los jugadores pueden guardar el servidor y conectarse siempre

**Costo**: Las Elastic IPs son **gratuitas** mientras estén asociadas a una instancia en ejecución. Te cobran ~$0.005/hora si la tienes reservada pero no la usas.

### 5.2 Asignar una Elastic IP

1. En la consola de EC2, ve al menú lateral izquierdo
2. En **"Network & Security"**, haz clic en **"Elastic IPs"**
3. Haz clic en **"Allocate Elastic IP address"**
4. Deja las opciones predeterminadas
5. Haz clic en **"Allocate"**
6. Verás tu nueva Elastic IP en la lista (ej: `54.160.25.24`)

### 5.3 Asociar la Elastic IP con tu Instancia

1. Selecciona la Elastic IP que acabas de crear (marca la casilla)
2. Haz clic en **"Actions"** → **"Associate Elastic IP address"**
3. Configura la asociación:
   - **Resource type**: Instance
   - **Instance**: Selecciona tu instancia de Minecraft
   - **Private IP address**: Se selecciona automáticamente
4. Haz clic en **"Associate"**

### 5.4 Verificar la Elastic IP

1. Ve de nuevo a **"Instances"** en el menú de EC2
2. Selecciona tu instancia de Minecraft
3. En los detalles, verás que la **Public IPv4 address** ahora es tu Elastic IP

**Guarda esta IP** - la necesitarás para conectarte al servidor de Minecraft.

### 5.5 Probar la Conexión al Puerto de Minecraft

Antes de intentar conectarte desde el juego, verifica que el puerto 25565 esté abierto:

**En Windows (PowerShell)**:
```powershell
Test-NetConnection -ComputerName 54.160.25.24 -Port 25565
```

**En Mac/Linux**:
```bash
nc -zv 54.160.25.24 25565
```

**En ambos casos, deberías ver**:
```
Connection successful / succeeded
```

Si la conexión falla:
- Verifica que el servidor de Minecraft esté corriendo
- Revisa las reglas del Security Group (puerto 25565 debe estar abierto)
- Asegúrate de que el firewall de la instancia no esté bloqueando el puerto

---

## Paso 6: Conectarse al Servidor desde Minecraft

¡El momento que has estado esperando! Ahora te conectarás a tu servidor desde el cliente de Minecraft.

### 6.1 Preparar el Cliente de Minecraft

1. Abre el **Launcher de Minecraft**
2. Asegúrate de tener instalado **Minecraft Java Edition** (la versión de Windows 10/Bedrock no es compatible)
3. Inicia el juego en la **misma versión** que tu servidor
   - Para verificar la versión del servidor, revisa los logs al iniciarlo o el archivo `version.json`

### 6.2 Agregar el Servidor

1. En el menú principal, haz clic en **"Multiplayer"**
2. Haz clic en **"Add Server"**
3. Completa la información:
   - **Server Name**: El nombre que quieras (ej: "Mi Servidor AWS", "Servidor FCFM")
   - **Server Address**: Tu Elastic IP (ej: `54.160.25.24`)
4. Haz clic en **"Done"**

### 6.3 Conectarse al Servidor

1. El servidor aparecerá en tu lista con indicadores de conexión:
   - **Barra verde**: Conexión perfecta
   - **Barra naranja**: Conexión aceptable
   - **Barra roja**: Conexión pobre
   - **Tachado rojo**: No se puede conectar
2. Haz doble clic en el servidor o selecciónalo y haz clic en **"Join Server"**
3. **¡Listo!** Deberías estar dentro de tu mundo de Minecraft

### 6.4 Solución de Problemas de Conexión

**Si no puedes conectarte**:

1. **Verificar que el servidor esté corriendo**:
   ```bash
   # Conectarse por SSH a la instancia
   # Si usaste screen:
   screen -r minecraft
   
   # Si usaste systemd:
   sudo systemctl status minecraft
   ```

2. **Verificar el puerto**:
   - Revisa el Security Group en AWS
   - El puerto 25565 debe estar abierto para 0.0.0.0/0

3. **Verificar la Elastic IP**:
   - Asegúrate de usar la IP correcta
   - Intenta hacer ping: `ping 54.160.25.24` (reemplaza con tu IP)

4. **Verificar la versión**:
   - El cliente y el servidor deben estar en la misma versión

5. **Revisar los logs del servidor**:
   ```bash
   # Si usaste systemd:
   sudo journalctl -u minecraft -f
   
   # Si usaste screen, estarás viendo los logs directamente
   ```

### 6.5 Comandos Útiles del Servidor

Una vez dentro del juego, si eres operador (OP) del servidor, puedes usar comandos:

**Para hacerte OP** (desde la consola SSH del servidor, no desde el juego):
```
op TuNombreDeUsuario
```

**Comandos comunes en el juego** (todos empiezan con `/`):
```
/gamemode creative        # Cambiar a creativo
/gamemode survival        # Cambiar a supervivencia
/tp JugadorA JugadorB    # Teletransportar JugadorA a JugadorB
/give Jugador minecraft:diamond 64  # Dar 64 diamantes
/time set day            # Cambiar a día
/weather clear           # Limpiar el clima
/seed                    # Ver la semilla del mundo
/whitelist add Jugador   # Agregar jugador a la lista blanca
/ban Jugador            # Banear jugador
```

**Para detener el servidor de forma segura**:
- Desde la consola SSH (donde corre el servidor), escribe: `stop`
- Esto guarda el mundo correctamente antes de apagar

---

## Recursos Adicionales

### Documentación Oficial

- **Minecraft Wiki**: [minecraft.wiki](https://minecraft.wiki/)
- **AWS EC2 Documentation**: [docs.aws.amazon.com/ec2](https://docs.aws.amazon.com/ec2/)
- **Paper MC**: [docs.papermc.io](https://docs.papermc.io/)
- **Spigot**: [www.spigotmc.org](https://www.spigotmc.org/)

### Comunidades y Foros

- **r/admincraft** (Reddit): Comunidad de administradores de servidores Minecraft
- **Minecraft Server Forum**: [www.minecraftforum.net/forums/servers-java-edition](https://www.minecraftforum.net/forums/servers-java-edition)
- **AWS Forums**: [forums.aws.amazon.com](https://forums.aws.amazon.com/)

### Herramientas Útiles

- **MCEdit**: Editor de mundos offline
- **NBTExplorer**: Editor de archivos NBT (datos del mundo)
- **Chunky**: Pre-generador de chunks para reducir lag
- **MCA Selector**: Herramienta para manipular chunks

### Calculadoras y Planificadores

- **AWS Pricing Calculator**: [calculator.aws](https://calculator.aws/)
- **Minecraft Server RAM Calculator**: Busca en Google para varias opciones

---

## Conclusión

¡Felicidades! Has configurado exitosamente tu propio servidor de Minecraft en AWS. Ahora tienes:
* Un servidor de Minecraft completamente funcional
* Acceso SSH para administración remota
* Una Elastic IP estática para fácil conexión
* WIP: Conocimientos de optimización y mantenimiento
* WIP: Estrategias de backup y recuperación

### Próximos Pasos Recomendados

1. **Personaliza tu servidor**: Instala mods, plugins, datapacks
2. **Invita a amigos**: Comparte la IP y disfruten juntos
3. **Optimiza costos**: Considera Spot o Reserved Instances si usarás el servidor a largo plazo
4. **Automatiza todo**: Scripts de backup, reinicio automático, actualizaciones
5. **Monitorea**: Configura alertas para estar al tanto de problemas

### Recordatorios Importantes

- **Detén la instancia** cuando no la uses para ahorrar dinero (no aplica a Elastic IP asociada)
- **Haz backups regularmente** - no esperes a perder tu mundo
- **Mantén el sistema actualizado** para seguridad y estabilidad
- **Monitorea los costos** en el AWS Billing Dashboard

---

## Soporte y Contribuciones

Si encuentras errores en esta guía o quieres contribuir mejoras, ¡tus aportes son bienvenidos!

### Autor Original

Cynthia Elizabeth Castillo Silva. AWS Community Builder en la categoría de Machine Learning. Entusiasta del aprendizaje y la colaboración.

### Presentado en
**Twitch** - En el canal de La Matemaga :)
**FCFM UANL** - Facultad de Ciencias Físico Matemáticas, Universidad Autónoma de Nuevo León

### Licencia

Esta guía es de uso libre para fines educativos y personales.

---

**¡Que disfrutes tu servidor de Minecraft en AWS!** 🎮⛏️🌍
