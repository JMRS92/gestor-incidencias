# Laboratorio: Despliegue de una aplicación web con Docker Compose

## Respuestas a las preguntas del laboratorio

### 1. ¿Por qué el frontend no necesita conectarse a PostgreSQL?
Porque el frontend solo sirve la interfaz visual (HTML, CSS y JavaScript) en el navegador del usuario. Cuando el usuario interactúa o pide datos, el frontend habla con la API del backend mediante peticiones HTTP (`/api/`). Es el backend el que se encarga de procesar la lógica y consultar la base de datos. Conectar el frontend directamente a la base de datos sería una mala práctica y un fallo grave de seguridad, ya que expondríamos las credenciales de la base de datos al navegador.

### 2. ¿Por qué el backend pertenece a dos redes?
Porque actúa como intermediario entre las dos partes. Necesita estar en la red `frontend` para que Nginx le pueda pasar las peticiones que llegan de los usuarios (`:8000`), y al mismo tiempo necesita estar en la red `datos` para poder conectarse a PostgreSQL (`:5432`) y guardar o consultar las incidencias.

### 3. ¿Qué servicio necesita publicar un puerto en el anfitrión?
Solo el frontend (en el puerto 8080 configurado en `.env`). Es el único servicio al que nosotros como usuarios tenemos que acceder desde el navegador en `localhost:8080`. El backend y la base de datos se comunican internamente por las redes de Docker y no necesitan estar expuestos a mi máquina anfitriona.

### 4. ¿Por qué publicar PostgreSQL hacia fuera no es la solución adecuada para un fallo de comunicación interno?
Porque cuando los contenedores fallan al comunicarse entre sí dentro de Docker, el problema está en la red interna o en la resolución DNS de Docker, no en el anfitrión. Publicar el puerto 5432 en el host solo permitiría acceder desde fuera del entorno Docker (desde mi PC), pero el backend seguiría sin poder llegar a `db` dentro de la red. Además, exponer la base de datos hacia fuera es peligroso a nivel de seguridad porque la dejas abierta a accesos indebidos.

### 5. ¿Qué ventaja ofrece Docker Compose respecto a ejecutar los tres contenedores manualmente?
Con Docker Compose tengo toda la arquitectura definida en un solo fichero (`compose.yaml`): servicios, redes, volúmenes, dependencias y variables. En lugar de tener que acordarme de escribir comandos larguísimos de `docker run`, crear las redes a mano con `docker network create` y enlazar cada contenedor en el orden correcto, con un simple `docker compose up -d` levanto todo el proyecto de golpe y sé que siempre se va a desplegar igual.

### 6. ¿Por qué hemos dividido la arquitectura en dos redes?
Por seguridad y aislamiento. Al tener una red `frontend` y otra red `datos`, el contenedor de Nginx no puede comunicarse directamente con PostgreSQL. Si alguien consiguiera vulnerar el frontend, no tendría acceso directo a la base de datos porque físicamente no comparten red. Solo el backend tiene permiso para acceder a ambas.

### 7. ¿Por qué el backend utiliza db como dirección de PostgreSQL?
Porque Docker tiene su propio servidor DNS interno dentro de las redes que creamos. Al poner `DB_HOST=db`, Docker resuelve ese nombre `db` a la IP interna que tenga el contenedor de PostgreSQL en ese momento. Así no dependemos de direcciones IP fijas, que cambiarían cada vez que reiniciamos o recreamos los contenedores.

### 8. ¿Qué diferencia existe entre detener un proyecto y eliminar sus volúmenes?
Si solo detengo o bajo el proyecto con `docker compose stop` o `docker compose down`, los contenedores se paran o se eliminan, pero el volumen con los datos de PostgreSQL (`postgres_data`) se mantiene intacto. Al volver a levantar con `up -d`, las incidencias siguen estando ahí. En cambio, si uso `docker compose down -v`, le estoy diciendo a Docker que borre también los volúmenes, por lo que se destruye la base de datos y se pierden todos los datos guardados.

### 9. ¿Qué consecuencias tendría para la empresa perder el volumen de PostgreSQL?
Sería crítico, porque se perderían todas las incidencias registradas por los usuarios y el histórico del sistema. La empresa se quedaría sin poder dar seguimiento a los problemas técnicos, perdería información clave de soporte y provocaría una parada en el servicio con pérdidas de tiempo y dinero hasta intentar restaurar una copia de seguridad.
