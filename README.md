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

el navegador solo pide index.php y los otros dos no se piden nunca por url los mete php por dentro

index.php define $titulo y luego incluye encabezado.php que trae el doctype y el head y el body abierto y despues saca el h1 e incluye pie.html

o sea que el html que llega al navegador no esta entero en ningun fichero se arma en el momento con los tres trozos

# lo que mas me ha gustado

que la variable viaja al include

en encabezado.php se usa $titulo y ahi no esta definida sino en index.php y funciona porque php no ejecuta el include aparte sino que lo pega en el sitio de la linea y sigue con las mismas variables por eso el titulo de la pestaña sale bien

si pones el include antes de definir $titulo el title sale vacio asi que el orden de las lineas importa

y el pie es un .html y lo incluye igual porque include no mira la extension

# seguridad

aqui el include lleva el nombre escrito a mano asi que no hay por donde meterle mano pero si en vez de eso lo cogiera de la url

    include($_GET['pagina']);

entonces el que pide la pagina elige que fichero abre php y eso es un lfi

con algo asi en la barra

    ?pagina=../../../../etc/passwd

php se lee el fichero y lo escupe en la pagina y si el servidor tiene permitido abrir urls de fuera ya no es leer es traerse un fichero php de otro sitio y ejecutarlo

por eso el include nunca se monta con lo que venga de fuera y si hace falta elegir pagina se hace con una lista cerrada de las que valen

# cosas que me pasaron

el enunciado pone cabecera.php en el arbol pero el codigo hace include encabezado.php asi que deje encabezado.php que es lo que pide el include

me salio un 403 forbidden al levantarlo y en docker logs php-practicas estaba el motivo

    "/app/index.php" is forbidden (13: Permission denied)

el umask de la maquina es 0007 asi que los ficheros nacen con 660 y dentro del contenedor nginx corre como usuario nginx y no como application y se arregla con chmod -R o+rX

las dos practicas usan el mismo container_name y el mismo 8080 asi que hay que bajar una para levantar la otra igual que me paso con nginx y apache peleandose por el 80
