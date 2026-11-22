## Laboratorio de include
Adrián Donoso  Implementación de Aplicaciones Web

# estructura

    mi-include-php/
    ├── .vscode/launch.json
    ├── app/
    │   ├── index.php
    │   ├── encabezado.php
    │   └── pie.html
    ├── docker-compose.yml
    └── README.md

# como se levanta

docker compose up -d

y se entra en http://localhost:8080 y para pararlo docker compose down

# como funciona

el navegador solo pide index.php y los otros dos los mete php por dentro

index.php define $titulo y mete encabezado.php con el doctype y el head y el body abierto y luego saca el h1 y mete pie.html

o sea que el html que llega al navegador no esta entero en ningun fichero se arma en el momento con los tres trozos

# lo que mas me ha gustado

que la variable viaja al include

en encabezado.php se usa $titulo y ahi no esta definida sino en index.php y funciona porque php lo pega en el sitio de la linea y sigue con las mismas variables por eso sale bien el titulo de la pestaña

si el include va antes de definir $titulo el title sale vacio

# seguridad

aqui el nombre del fichero va escrito a mano pero si lo cogiera de la url

include($_GET['pagina']);

el que pide la pagina elige que fichero abre php y eso es un lfi

http://localhost:8080/index.php?pagina=../../../../etc/passwd

lo que va detras del ? son los parametros de la url y de ahi los saca $_GET asi que el valor de pagina lo pongo yo desde la barra

php se lo lee y lo escupe en la pagina y si el servidor deja abrir urls de fuera ya no es leer es traerse un php de otro sitio y ejecutarlo

por eso el include no se monta nunca con lo que venga de fuera

# cosas que me pasaron

el enunciado pone cabecera.php en el arbol pero el codigo hace include encabezado.php asi que deje el del include

me salio un 403 al levantarlo

"/app/index.php" is forbidden (13: Permission denied)

el umask de la maquina deja los ficheros en 660 y nginx dentro del contenedor no corre con mi usuario asi que no podia leerlos y se arregla con chmod -R o+rX

las dos practicas usan el mismo nombre de contenedor y el mismo 8080 asi que hay que bajar una para levantar la otra

# repositorio

https://github.com/htfrfr/iaweb-include-php
