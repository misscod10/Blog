---
title: "Grafana"
date: 2022-07-26T20:17:02+02:00
---
# Proyecto Grafana

## Introducción

En este proyecto haré las configuraciones necesarias para monitorizar la utilización de recursos de un grupo de servidores desde una interfaz web.

### Programas y herramientas necesarias

Para cumplir este objetivo, utilizaré Telegraf como recolector de datos, InfluxDB como base de datos donde guardarlos y Grafana como interficie web para representarlos.

#### Telegraf

Se encarga de recolectar la información que le especificamos en la configuración (Ej.: CPU/RAM/LOAD) y los envía al output configurado (Ej.: una base de datos).

#### InfluxDB

Es una base de datos en tiempo real escrita en Go. Es a donde Telegraf va a mandar la información.

#### Grafana

Es el Dashboard que se encarga de representar toda la información que InfluxDB tiene almacenado. Accederemos a este servicio desde la dirección localhost:3000.

#### Traefik

Es la proxy inversa que vamos a utilizar para implementar el protocolo HTTPS cuando accedamos al dashboard web del Grafana y para hacer la traducción del nombre grafana.labs.pue.es a la IP y puerto del servicio (puerto 3000).

#### MySQL (no obligatorio)

Grafana viene de base con una base de datos SQLite que emplea para guardar sus usuarios, contraseñas, datasources etc. En la gran mayoría de casos esto sería suficiente, pero, me he puesto como desafío utilizar una MySQL en vez de la SQLite.

### Esquema Final

![Diagrama de Grafana](/Diagrama1.png 'Diagrama de grafana')

Telegraf recogerá los datos de cada màquina y los enviará a la base de datos InfluxDB que está en el server central. Una vez los datos están en la base de datos, Grafana enviarà una request cada 10s para saber los datos que tengamos configurados en el dashboard y representarlos. 

### Pasos seguidos

-   ☒ Instalar una instancia básica en una máquina virtual
-   ☒ Crea interfaz para entender como funcionan las queries de Influx
-   ☒ Explicar que representa la interfaz
-   ☒ Instalar en múltiples hosts con un solo dB
    -   ☒ Hacer Docker-compose para el server central con InfluxDB Grafana Y MySQL
        -   ☒ Escribir-lo
        -   ☒ Probarlo
        -   ☒ Solucionar el error con el container del grafana
        -   ☒ Probarlo en el server central
            -   ☒ Solucionar error al acceder desde un buscador web al grafana en el server central
            -   ☒ Hacer que se pueda acceder a grafana desde https con traefik
    -   ☒ Haz que InfluxDB escuche por un puerto a otros servers
    -   ☒ Instalar telegraf fuera de los containers
    -   ☒ Instalar telegraf en los otros servers con ansible
        -   ☒ Escribe el archivo para que funcione con servers Ubuntu y red hat
        -   ☒ Prueba-lo en maquinas virtuales
        -   ☒ Hacer conectividad entre los servers
        -   ☒ Deploy
    -   ☒ Crear las dashboards y confirmar que se recibe la información
    -   ☒ Documentar como lo he hecho, desde el compose hasta el telegraf.conf y el ansible deploy.
-   ☒ Documentar el proceso

## Documentación del proceso

### 1. Hacer una instalación de prueba en una máquina virtual Ubuntu Server.

#### 1.1 Preparaciones pre-instalación

Antes de instalar nada, hacemos un "update" para tener los últimos repositorios y ejecutamos la orden siguiente para instalar los programas que necesitaremos para la instalación:

``` bash
sudo apt install curl gnupg gnupg1 gnupg2 net-tools software-properties-common
```

Estas herramientas suelen venir instaladas en la gran mayoría de distribuciones, pero no las encontraremos en una instalación de Ubuntu server mínima, así que viene bien ejecutarlo para asegurarse de que están, ya que son las herramientas que nos permitirán añadir repositorios extras.

### 1.2 Instalamos el software

Añadimos el repositorio de Influx para poder instalar Telegraf y InfluxDB:

``` bash
sudo curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -

source /etc/lsb-release

echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
```

Después actualizamos el listado de paquetes de los repositorios y instalamos InfluxDB:

``` bash
sudo apt update

sudo apt install influxdb
```

Iniciamos el servicio y lo activamos para que cada vez que se encienda la máquina, se encienda el servicio:

``` bash
sudo systemctl start influxdb
sudo systemctl enable influxdb
```

Para terminar de configurar Influx entramos en su terminal, creamos una base de datos y un usuario para la misma:

``` influx
influx
create database telegraf
create user user_que_quieras with password 'la_contra_que_quieras'
```

Después instalamos telegraf y activamos su servicio:

``` bash
sudo apt install telegraf
sudo systemctl start telegraf
sudo systemctl enable telegraf
```

Más tarde entramos en el archivo /etc/telegraf/telegraf.conf y vamos a la sección `[[outputs.influxdb]]` y des-comentamos la línea que contiene lo siguiente:

``` yml
urls = ["http://localhost:8086"]
```

En esa línea se especifica la URL a la que Telegraf enviara sus datos y al puerto al que lo hará. En este caso, es localhost porque es una instalación que solo se comunicara consigo misma.

Después añadimos el repositorio de grafana:

``` bash
sudo wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```

Actualizamos los repositorios e instalamos grafana:

``` bash
sudo apt-get update -y
sudo apt-get install grafana -y
```

Y por último activamos e iniciamos el servicio grafana-server:

``` bash
systemctl start grafana-server
systemctl enable grafana-server
```

#### 1.3 Configuración de Grafana

Para entrar a la UI de grafana, accedemos desde el buscador a la URL lohalhost:3000:

![Pastedimage20220627124826.png](/Pasted_image_20220627124826.png '')

El login por default es admin:admin, pero una vez entras te obliga a cambiar de contraseña.

Una vez inicias sesión ves lo siguiente:

![Pastedimage20220627125135.png](/Pasted_image_20220627125135.png?1658297391991)

Es la pantalla de recibimiento. Hay varios tutoriales sobre como hacer cosas básicas, noticias de las actualizaciones, etc.

En la sección de dashboards es donde crearemos nuestros paneles de monitorización, para crear uno nuevo le clicas a new dashboard.

Cada panel es una tabla que representa un query que se le hace a la base de datos. Depende de lo que quieras extraer tenderas que cambiar la query. Hay dos maneras de formular la misma.

Con una plantilla:

![Pastedimage20220627125704.png](/Pasted_image_20220627125704.png?1658297391991)

O con una orden entera directamente:

![Pastedimage20220627125946.png](/Pasted_image_20220627125946.png?1658297391991)

También se pueden importar dashboards enteros desde dashboard\>import utilizando el ID del dashboard que se quiere importar, su URL o subiendo un archivo .json:

![Pastedimage20220627130210.png](/Pasted_image_20220627130210.png?1658297391991)

#### 1.4 Mi interfaz personalizada

De momento, he hecho que Grafana exponga:

-   Tiempo que lleva la máquina encendida
-   Espacio del disco que esta siendo utilizado
-   Procesos actuales activos
-   Procesos "zombies"
-   Uso Total de la CPU
-   Uso Total de la RAM
-   Uso Total de la SWAP
-   Carga del sistema promedio
-   Uso de la CPU total en una timeline
-   Uso de la ram total en una timeline

Foto del resultado:

![Pastedimage20220628131116.png](/Pasted_image_20220628131116.png?1658297391991)

### 2 Instalación en los servidores reales

#### 2.1 Instalar en el server central InfluxDB y Grafana con MySQL con containers y Telegraf como servicio

Creamos los contenedores de Grafana, InfluxDB y MySQL con el siguiente compose:

``` yml
version: '3'

services:
  mysql:
    image: mysql
    container_name: mysql
    environment:
      - "MYSQL_ROOT_PASSWORD=MaNF48Fc"
      - "MYSQL_DATABASE=grafana"
      - "MYSQL_USER=administrador"
      - "MYSQL_PASSWORD=MaNF48Fc"
    volumes:
      - /data/mysql/grafana:/var/lib/mysql
    restart: unless-stopped
    networks:
      - grafana-db 
  influx:
    image: influxdb:1.8.4-alpine
    container_name: influxdb
    ports:
      - "10.91.240.155:8086:8086"
    environment:
      - "INFLUXDB_REPORTING_DISABLED=true"
      - "INFLUXDB_DB=influx"
      - "INFLUXDB_ADMIN_USER=administrador"
      - "INFLUXDB_ADMIN_PASSWORD=MaNF48Fc"
      - "INFLUXDB_WRITE_USER="
      - "INFLUXDB_WRITE_USER_PASSWORD="
      - "INFLUXDB_DATA_QUERY_LOG_ENABLED=false"    
    volumes:
      - /data/influxdb/data:/var/lib/influxdb
    restart: unless-stopped
    networks:
      - grafana-db 

  grafana:
    
    image: grafana/grafana
    container_name: grafana
    depends_on:
      - mysql
    environment:
      - "GF_SERVER_DOMAIN="
      - "GF_SERVER_ROOT_URL"
      - "GF_SECURITY_ADMIN_PASSWORD="
      - "GF_DATABASE_TYPE=mysql"
      - "GF_DATABASE_SSL_MODE=verify-full"
      - "GF_DATABASE_NAME=grafana"
      - "GF_DATABASE_USER=administrador"
      - "GF_DATABASE_HOST=mysql"
      - "GF_DATABASE_PASSWORD=MaNF48Fc"
      - "GF_ANALYTICS_REPORTING_ENABLED=false"
      - "GF_PATHS_PLUGINS=/plugins"
    volumes:
      - /data/grafana/plugins:/plugins
    labels:
      - traefik.enable=true
      - traefik.http.routers.grafana.rule=Host(`grafana.labs.pue.es`)
      - traefik.http.routers.grafana.entrypoints=web
      - traefik.http.routers.grafana_s.rule=Host(`grafana.labs.pue.es`)
      - traefik.http.routers.grafana_s.entrypoints=web-secure
      - traefik.http.routers.grafana_s.tls.certresolver=letsencrypt
      - traefik.docker.network=traefik
    restart: unless-stopped
    networks:
      - traefik
      - grafana-db
networks:
  grafana-db:
  traefik:
    external: true
```

Este docker-compose, aparte de levantar los contenedores, abre el puerto 8086 desde la IP 10.91.240.155 como si fuera el puerto 8086 de la máquina host. Utilizaremos esa IP y puerto después cuando hagamos que telegraf envíe los datos hacia la base de datos en el server central.

Para poder hacer que Traefik traduzca el nombre y nos proporcione HTTPS hay que añadir las siguientes líneas al docker-compose.

``` yml
labels:
# Para decirle a traefik que exponga este container
        - traefik.enable=true
# Para decirle a traefik que a traves de que dominio va a acceptar                                                     conexiones http
        - traefik.http.routers.grafana.rule=Host(`grafana.labs.pue.es`)
# Para decirle a traefik el entrypoint por el que estas conexiones             tienen que entrar
        - traefik.http.routers.grafana.entrypoints=web
# Para decirle a traefik que a traves de que dominio va a acceptar                                                     conexiones https
        - traefik.http.routers.grafana_s.rule=Host(`grafana.labs.pue.es`)
# Para decirle a traefik el entrypoint por el que estas conexiones             tienen que entrar
        - traefik.http.routers.grafana_s.entrypoints=web-secure
# Le indica a traefik que certificado utilizar
        - traefik.http.routers.grafana_s.tls.certresolver=letsencrypt
# Le indica la red en la que se encuentra el container de traefik
        - traefik.docker.network=traefik
```

El archivo telegraf.conf que usaremos para enviar datos desde todos los servers a el server central es el siguiente:

``` yml
# Configuration for sending metrics to InfluxDB
[[outputs.influxdb]]
  ## The full HTTP or UDP URL for your InfluxDB instance.
  ##
  ## Multiple URLs can be specified for a single cluster, only ONE of the
  ## urls will be written to each interval.
  # urls = ["unix:///var/run/influxdb.sock"]
  # urls = ["udp://127.0.0.1:8089"]
  urls = ["http://10.91.240.155:8086"]
```

Esta vez, en vez de ser localhost, usamos la IP que el container de InfluxDB ha abierto a la intranet de servers con el puerto 8086.

#### 2.2 Deploy

Para instalar y configurar telegraf en cada server, utilicé el siguiente yml de ansible:

``` yml
---

- name: setup servers pue
  hosts: servers
  vars:
    user: santiago
  become: yes
  tasks:
  
  - name: Update repos, instalar herramientas previas 
    apt:
      update_cache: yes 
      force_apt_get: yes 
      cache_valid_time: 3600
    when: (ansible_facts['os_family'] == "Debian")

  - name: instalar herramientas previas 
    apt:
      state: present
      name:
        - curl
        - gnupg
        - gnupg1
        - gnupg2 
        - net-tools
        - software-properties-common

    when: (ansible_facts['os_family'] == "Debian")

  - name: Import InfluxDB GPG signing key
    apt_key: 
      url: https://repos.influxdata.com/influxdb.key 
      state: present 
    when: (ansible_facts['os_family'] == "Debian")
  
  - name: Add InfluxDB repository
    apt_repository: 
      repo: 'deb https://repos.influxdata.com/ubuntu trusty stable' 
      state: present 
    when: (ansible_facts['os_family'] == "Debian")
  
  - name: Install telegraf
    apt:
      name: telegraf
      state: latest
    when: (ansible_facts['os_family'] == "Debian")

  - name: Add telegraf repository
    yum_repository:
      name: influxdb
      description: InfluxDB Repository - RHEL $releasever
      baseurl: https://repos.influxdata.com/rhel/$releasever/$basearch/stable
      gpgcheck: yes
      gpgkey: https://repos.influxdata.com/influxdb.key
    when: (ansible_facts['os_family'] == "RedHat")

  - name: Install telegraf REDHAT 
    yum:
      name: telegraf
      state: latest
      disablerepo: "*" 
      enablerepo: influxdb
    when: (ansible_facts['os_family'] == "RedHat")

  - name: Copy config
    copy:
      src: ./telegraf.conf
      dest: /etc/telegraf/telegraf.conf
      owner: root
      group: root
      mode: 0644
  
  - name: restart telegraf
    service:
      name: telegraf
      state: restarted
```

Este ansible sirve tanto para las distros de la familia Debian como para las distros de la familia RedHat. Lo que hace es añadir el repositorio de Influx, instalar Telegraf y copiar la configuración del mismo que está en local a la dirección /etc/telegraf/telegraf.conf y reinicia el servicio para que la nueva configuración esté en efecto.

Mientras hacia el deploy, encontré que los servidores CentOS 8 no pueden instalar ningún paquete porque los repositorios oficiales de CentOS han sido deprecados y han cambiado de URL.

Para poder instalar-lo en estos servers modifiqué la siguiente parte.

``` yml
- name: Install telegraf REDHAT 
    yum:
      name: telegraf
      state: latest
      disablerepo: "*" 
      enablerepo: influxdb
    when: (ansible_facts['os_family'] == "RedHat")
```

Obligo a yum a buscar telegraf en el repositorio influx-db des-habilitando todos los repositorios y activando solo el repositorio influx-db.

#### 2.3 Dashboard

Para monitorizar mejor los servers, me dijeron que sería mejor que utilizaremos el dashboard estándar de Telegraf. Se puede importar con la id 928.

## Conclusiones

Grafana es una herramienta muy útil para

### Proyecto hecho por:

Santiago Pastor Serrano el 22-07-2022 a las 11:28

### Tutor de prácticas

Ramón de la Rosa
