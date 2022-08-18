---
title: "Como tener una web gratis"
date: 2022-08-07T16:44:21+02:00
draft: false
tags: ["Hugo","Markdown","Github"]
---

# Introducción

En este post os voy a enseñar como he creado esta web y como no me esta costando ni un céntimo mantenerla. Y no és difícil. De hecho ni he tocado un solo archivo html, pero lo explicaré luego.

# Pasos a seguir

1. Instalar Hugo
2. Crear repositorio de github e iniciar hugo en el mismo
3. Escojer un tema 
4. Crear el primer post
5. Como hostearla con github
6. Como actualizar la web
7. Utilizar un dominio personalizado(opcional)

# Instalar Hugo

Distros basadas en Debian

```shell
apt install hugo
```

Distros basadas en Arch

```shell
pacman -S hugo
```

Distros basadas en RedHat

```shell
dnf install hugo
```

En windows recomiendo seguir esta guía

https://www.techielass.com/how-to-install-hugo-on-windows-10/

# Crear un repositorio en github e iniciar hugo en el mismo

El primer paso es crear un repositorio en github, lo utilizaremos para tener los archivos a partir de los cuales hugo creará la pàgina web guardados en la nube.

Para crearlo, vamos a la página web de [Github](https://github.com) y entramos en nuestro perfil:

Después clicamos en el boton que pone "New" en la sección de repositorios personales. Este repositorio lo puedes crear con el nombre que quieras y con el modo de privacidad que prefieras, aunque personalmente lo creé como un repositorio privado llamado Blog. Una vez lo hemos creado lo clonamos en nuestro equipo: 

```bash
git clone [url del repo]
```

Para iniciar hugo en el repositorio, ejecutas:

```bash
hugo new website
```

Al ejecutar la orden anterior se generan una serie de carpetas y archivos como `config.toml`, `content` , `themes` y `static`. Nombro estas porque son las que mas usaráras a la hora de crear tu web pero se crean otras también.

# Escojer tema

El tema de hugo que escojas cambiara cosas como el layout de tu pagina, diferentes widgets que vengan de base y como funcionarán mecánicas como las "tags" de la web. Puedes tener varios instalados para cambiar entre ellos de forma rápida, e incluso puedes crear los tuyos propios. 

Recomiendo esta web para escojer temas de hugo [gratuitos](https://hugothemesfree.com). También puedes comprar uno pero, entre tu y yo, no me gusta pagar por algo cuando puedo hacer uno mas personalizado como proyecto personal.

Para añadir un tema a tu grupo de temas, añade este como un submodulo de git dentro de la carpeta de themes de tu repo:

```bash
git submodule add -f [url del repo de github del tema] themes/[nombre que le quieras poner]
```

Y después modifica la sección de tu config.toml en la que especifica el tema para que la variable contenga el nombre de la carpeta en la que el tema está.

```toml
theme = "[nombre de la carpeta]"
```

También aprobecha para cambiar el título de la web y el subtítulo.

# Crear el primer post

Como post, me refiero a una pagina que se va a enseñar en el anexo principal de tu web. Se suele utilizar en los blogs como este. Para crear uno, hay que ejecutar la siguiente orden:

```bash
hugo new post [nombre del post]
```

Este comando te generará un archivo markdown dentro de la carpeta `content/posts/`. A dentro encontrarás las secciones de: `

- `title` donde cambiaras el título
- `date` donde pondrás la fecha en la que se creó, 
- `draft` que tendrá un valor booleano segun si quieres que se muestre este post cuando vuelvas a generar la web con hugo 
- `tags` esta no la encontrarás de base, pero la puedes configurar para que aparezca. En esta podrás ponerle las etiquetas con las que el post se identifica con la siguiente sintaxis: 
`tags : ["tag1", "tag2", "tag3"]`

# Como hostearla con github

Ahora te debes estar preguntando: Ok, ahora tengo unos archivos markdown, un repo y una estructura de carpetas. *¿Como creo y hosteo la web?*

No te preocupes, ahora viene la magia que va ha hacerte tener tu web gratis.

Primero, vamos a crear un segundo repositorio en github. Esta vez tienes que crear uno público y el nombre que le pongas da igual.

Después, borra la carpeta public y añade tu repo como submodulo como hemos hecho anteriormente con el tema en la carpeta `/public`:

```bash
git submodule add [url de tu repo de github] public
```

Una vez hecho esto, ejecutamos lo siguiente para generar la web estática en la carpeta public:

```bash
hugo -t [nombre del tema]
```

Si todo ha salido bien (que seguramente sea así), tendrás un archivo `index.html` en la carpeta public. Si és asi, haz un push a tus respectivos repositorios y entra en el repositorio de la carpeta públic de https://github.com y ves a configuración > pages:

![githubpages.png](/github_pages.png)

Selecciona la rama y carpeta en la que tendrá que buscar el archivo index.html (en este caso pon la rama main y la carpeta root como se enseña en la imagen)

Una vez hecho esto, en la misma pestaña de github pages, te darán el enlace para visualizar tu web.

# Como actualizar la web

A partir de ahora, lo único que tienes que hacer es crear nuevos posts, editarlos y hacerle push a los repositorios para añadir nuevos contenidos a tu web. Lo bueno de hugo, es que tiene un montón de funciones avanzadas para añadirle a tu web. Aunque no las explicaré en este post, las podrás encontrar de otras fuentes (o en esta misma si veo que tengo tiempo).

# Utilizar un dominio personalizado (opcional)

El único problema que tiene este método es que la dirección de tu web tiene un nombre feo de narices, pero se puede substituir. Para hacerlo necesitaràs comprar un dominio y añadirlo en la pagina de github pages mas abajo de dónde configuramos la dirección del archivo `index.html` antes. También tendremos que redirigir los servidores DNS de nuestro dominio (en mi caso serian desde cloudflare porque los utilizo para tener conexiones tls) para que redirigan hacia la dirección que te dió github originalmente.

Una vez hecho esto, edita el archivo config.toml en tu pagina web para cambiar el nombre de la dirección a tu dominio.

Después de media/una hora, tendrás accesso a tu web desde tu dominio personalizado.

Bien hecho! Ya tienes tu propia web.

---
pd:

- He hecho este post porque creo que és muy importante tener una página web para cualquier persona, ya sea como currículum, portafolio o simplemente como un blog personal. Entre el tráfico "random" que puedes obtener, la gente a la que envies un currículum con la url va a consumir mucho mas tiempo mirando tu web y que tener una web demuestra que tienes ganas de hacer y aprender nuevas cosas, tienes muchas mas probabilidades de obtener un puesto de trabajo. 

- Ademas és bastante educativo y (al menos para mi), muy chulo.
