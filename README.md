# Laboratorio de include

Adrian Donoso - Implementacion de Aplicaciones Web

Practica para ver como funcionan los includes de php, montada sobre el entorno
de docker con nginx y php-fpm que hicimos en la practica anterior.

## que hay aqui

    mi-include-php/
    ├── .vscode/launch.json     configuracion del debug con xdebug
    ├── app/
    │   ├── index.php           la pagina principal
    │   ├── encabezado.php      la cabecera con el <head>
    │   └── pie.html            el pie
    ├── docker-compose.yml
    └── README.md

## como se levanta

    docker compose up -d

y se entra en http://localhost:8080. para pararlo `docker compose down`.

## como funciona el codigo

el navegador solo pide index.php, los otros dos ficheros no se piden nunca por
url, los mete php por dentro antes de mandar la respuesta.

index.php hace tres cosas en este orden

1. define la variable `$titulo`
2. `include("encabezado.php")` que trae el doctype el head y el `<body>` abierto
3. imprime el `<h1>` con el titulo y despues `include("pie.html")` con el `<h3>`
   del pie, y cierra `</body></html>`

o sea que el html completo que llega al navegador no existe en ningun fichero,
se arma en el momento juntando los tres trozos. lo he comprobado con curl y
sale todo seguido

    <!DOCTYPE html>
    <html lang="es">
    <head>
        <meta charset="UTF-8">
        <title>Página con includes</title>
    </head>
    <body>
    <h1>Página con includes</h1>
    <h3>Pie de Página</h3>
    </body>
    </html>

## lo que me ha parecido mas interesante

**la variable viaja al include.** en encabezado.php se usa `<?= $titulo ?>` y esa
variable no esta definida ahi, esta definida en index.php. funciona porque el
include no es un fichero aparte que se ejecuta solo, php lo pega en el sitio
donde esta la linea del include y sigue con las mismas variables que ya tenia.
por eso en la pestaña del navegador pone "Página con includes".

si pones el include ANTES de definir `$titulo` el `<title>` sale vacio, el orden
de las lineas importa.

**pie.html es html y aun asi lo incluye.** include no mira la extension, se traga
lo que sea. la diferencia es que un .php que se incluye puede llevar codigo y se
ejecuta, y el .html entra tal cual. ademas como pie.html no se pide por url no
importa que sea un trozo suelto sin doctype ni nada.

**para que sirve esto.** la cabecera y el pie se escriben una vez y valen para
todas las paginas, si cambias el menu lo cambias en un sitio y no en veinte.

## cosas que me han pasado montandolo

**el enunciado se contradice con los nombres.** en el arbol de carpetas pone
`cabecera.php` pero el codigo de index.php hace `include("encabezado.php")` y
mas abajo te manda crear `app/encabezado.php`. he dejado `encabezado.php` que es
lo que pide el include, con el otro nombre da un error de que no encuentra el
fichero y la pagina sale a medias.

**error 403 forbidden la primera vez.** levante el contenedor y en vez de la
pagina salia un 403 de nginx. en el log del contenedor estaba el motivo

    "/app/index.php" is forbidden (13: Permission denied)

el umask de esta maquina es 0007 asi que los ficheros se crean con permisos 660
y las carpetas 770, o sea sin permiso para "otros". el problema es que dentro
del contenedor nginx corre con el usuario `nginx` y no con `application`, que es
el que coincide con mi uid 1000, asi que nginx no podia leer el index.php. se
arregla dando lectura a otros

    chmod -R o+rX .

y ya devuelve 200. no es cosa de docker ni de la imagen, es el umask de la
maquina virtual.

**solo puede haber un contenedor a la vez.** las tres practicas usan el mismo
`container_name: php-practicas` y el mismo puerto 8080, asi que si levantas esta
sin haber hecho `docker compose down` en la otra, falla por nombre repetido. es
lo mismo que me paso con nginx y apache peleandose por el puerto 80 en la
practica de mariadb.
