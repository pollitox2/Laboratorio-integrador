# Registro de decisiones — equipo01

Integrantes: Agustín Becerra, Isidora Martinez y Vicente Farias.

Completa **una fila por decisión**. Explica el porqué en una o dos frases: la defensa oral se basa en este documento.

## Decisiones técnicas (obligatorias)

| # | Decisión | Alternativas que consideraron | Por qué eligieron esta | Riesgo o costo que aceptan |
|---|----------|-------------------------------|------------------------|----------------------------|
| 1 | Imagen base de `catalog` y `worker` | La alternativa que consideramos fue utilizar python:3.12-slim | Elegimos esta opción (slim) porque es ligera | Es un poco más pesada que si hubieramos usado alpine, por lo que si o si debemos usar multi-stage build para no pasarnos el límite de tamaño |
| 2 | Imagen base de `orders` | La alternativa que consideramos fue utilizar node:20-alpine | Elegimos esta imagen ya que es ligera y reduce la imagen drasticamente | El riesgo que puede tener es resolución de dns con IPV6 obligando cambiar localhost por la ip numérica IPV4 dentro del healtcheck |
| 3 | Imagen base y usuario de `frontend` | La alternativa que consideramos fue Imagen nginx:alpine estándar, creando un usuario no-root manualmente | Elegimos esta imagen ya que viene configurada automaticamente para correr sin privilegios de root | El riesgo que podemos tener es que el usuario al no ser root, el sistema no permite usar los puertos privilegiados, entonces en nuestro caso al no tener estos puertos no podemos usar el 80 y estar obligados a usar los puertos más altos, nosotros en particular usar el 8080 |
| 4 | Cómo se manejan los secretos (contraseña de la base) | Mediante las variables en un archivo .env | Elegimos esta para que evitar que las credenciales queden en las capas de la imagen, evitamos queden expuestas si alguien accede a ellas | Se nos podria olvidar el .env en el gitignore y asi se podría subir accidentalmente con las credenciales |
| 5 | Diseño de redes: qué servicio ve a cuál | Crear dos redes separadas | Elegimos separar las dos redes ya que aislamos totalmente la base de datos del front y solo hay una red que se conecta que es catalog y el order, estos ultimos actuan como un puente entre el frontend y el backend | La complejidad del archivo compose ya que todos los servicios deben tener una red a la que asociarse |
| 6 | Healthcheck de `worker` y de `db`/`cache` | La alternativa que consideramos fue que se definiera en el docker compose | Elegimos estos healthcheck en el docker compose para asegurar que procesen correctamente. Para las bases utilizamos pg_isready y un ping y para worker utilizamos un /tmp/healthy/ que evalua antigüedad | En el caso de que un healthcheck falle o algun microcorte ocurra, todo el orquestador detienen el despliegue |
| 7 | Dónde viven los datos y cómo sobreviven a un reinicio | Volumenes que los gestiona el docker | Elegimos el bd_data para que la base de datos persista y no se borre al reiniciar | Para reiniciar la bd de 0 hay que usar docker compose down -v |

## Problemas que encontraron y cómo los diagnosticaron

El primer problema que encontramos fue al ejecutar el docker compose up, el contenedor de orders se quedaba en estado unhealthym, esto generaba que no se pudiera levantar el frontend. Lo diagnosticamos analalizando los logs y viendo que el Node.js estaba levantando en 0.0.0.0, pero el wget lo que buscaba en realidad era el localhost. Lo solucionamos cambiando el localhost por 127.0.0.1 en el Dockerfile de orders.

El segundo problema que tuvimos fue la falla del verificador al evaluar depends_on, ya que el script de verify.py nos descontaba 0.5 puntos a pesar de que estuviera correcta. Lo diagnosticamos usando el Select-String en el script de python. Tambien vimos que la advertencia que nos daba sobre la versión rompía el script. Lo solucionamos borrando la version en el docker compose.

## Incidentes

| Incidente | Qué pasó | Qué cambiaron | Cuánto demoraron |
|-----------|----------|---------------|------------------|
| 1 | El script verificador arrojó "AVISO codigo_de_app_modificado" en varios archivos, fallando la prueba por modificación de código original.| Por error, hubo un cambio en una identación adicional en los archivos, por lo que solo se tuvó que copiar y pegar los archivos originales para evitar que el script siguiera marcando que hubo un cambio en el archivo.| 25 minutos|
| 2 | Docker Compose V2 arrojaba un WARN por usar el atributo obsoleto version: '3.8'. La advertencia que estaba en la consola,  corrompía la lectura JSON del script verificador.| Se eliminó por completo la línea version: '3.8' del archivo docker-compose.yml para entregar una salida limpia al script.| 15 minutos|
| 3 | Pérdida de 1 punto en prueba "Caché de capas" durante la primera ejecución de verify.py.| No fue un fallo del código, sino de ejecución. Ya que, estábamos corriendo el script con el flag explícito --no-cache-check. Volvimos a correr el script limpio.| 5 minutos|

## Uso de IA

_Utilizamos Gemini como apoyo para entender el flujo y la arquitectura del proyecto. Específicamente, la IA nos ayudó a analizar qué hacía el código de la aplicación, revisar qué contenía cada archivo para identificar las configuraciones que faltaban, y nos ayudó para agregar algunas cosas mínimas necesarias en los Dockerfiles y organización del Docker-Compose._

## Qué haríamos distinto con más tiempo

_1.Lo primero sería automatizar las pruebas, ya que implementaríamos un pipeline básico para que el script verify.py se ejecute automáticamente en cada Pull Request, lo que nos habría ahorrado las pérdidas de puntos por formato en el docker-compose.
2.Lo segundo sería optimizar de mejor manera las imágenes, investigando cómo reducir aún más el peso final de los contenedores utilizando técnicas de limpieza de caché de paquetes en las capas de los multi-stage builds._
