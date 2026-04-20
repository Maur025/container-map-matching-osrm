# Map Matching - OSRM

Environment to up software of OSRM to fetch data to routes, matches or anything reference to geolocation, which includes calculations.

> **NOTA** el contenedor docker solo funciona en el sistema operativo Linux en el momento que escribo esto.

The following steps are Spanish for your easy understanding, add steps in English soon.

## Setup server OSRM

1. Descarga el archivo de datos de tu interés podría ser de tu país o región, es necesario un archivo pbf para continuar con los siguientes pasos.

    Por ejemplo: puedes usar el siguiente comando:

    ```BASH
    wget https://download.geofabrik.de/south-america/bolivia-latest.osm.pbf
    ```

    Sirve para descargar los datos de geolocalización de Bolivia.

2. Crea la carpeta `data` en el directorio donde tienes el archivo docker-compose.yml

    ```TEXT
    └── carpeta-contenedora-osrm
        ├── /data   #carpeta a crear
        └── docker-compose.yml
    ```

3. Copia el archivo descargado con los datos del mapa, en la carpeta que creaste con el nombre `data`.

    ```TEXT
    └── carpeta-contenedora-osrm
        ├── /data
        │   └── copia-aqui-tu-archivo.pbf
        └── docker-compose.yml
    ```

4. Abre `docker-compose.yml` y revisa el contenido, si los servicios `osrm-extract` o `osrm-contract` esta comentados, quita temporalmente los `#` para usarlos. Podrías encontrarlo de esta forma.

    ```YML
    # osrm-extract:
      # container_name: osrm-extract
    ```

    Solo remueve los símbolos, guarda y continua con el siguiente paso.

5. Ahora tenemos que ejecutar los contenedores, lo haremos uno a uno en caso de `osrm-extract` y `osrm-contract` para evitar problemas y colisiones.

    > **NOTA**
    > Solo debe ejecutar estos servicios la primera vez que, por que esto se encarga de extraer y transformar los datos para OSRM, es costoso y puede demorar mucho. Luego solo tiene que correr el servicio `map-osrm-server` que usara los datos tratados.

6. Ejecuta el siguiente comando para extraer los datos:

    ```BASH
    docker compose run osrm-extrac
    ```

7. Ejecuta el siguiente comando para que osrm transforme y procese los datos extraídos.

    ```BASH
    docker compose run osrm-contract
    ```

8. Ahora para correr el servicio principal `map-osrm-server`, necesitamos que este solo en el archivo, puedes eliminar los bloques de código de `osrm-extract` y `osrm-contract`, o puedes comentarlo para que no interfieran con el servidor principal.

    > **NOTA**
    > Revisa que los servicios mencionados antes se encuentren comentados o eliminados antes de continuar, si no lo están el se demorará mucho mas en iniciar el servicio principal, por que en cada ocasión se pre procesaran los datos antes de iniciar, esta acción es innecesaria después de hacerlo la primera vez.

9. Antes de iniciar, revisa el archivo `docker-compose.yml` y verifica o ajusta los puertos, o para ajustar cualquier otro parámetro según lo que necesites. Una vez listo ejecuta el siguiente comando para iniciar el servicio principal:

    ```BASH
    docker compose run map-osrm-server
    ```

    También puedes ejecutar el siguiente comando:

    ```BASH
    docker compose up -d
    ```

10. Con esto ya tienes disponible el servidor de calculos de OSRM, puedes probarlo desde `postman` o por `curl` para asegurarte de que esta funcionando correctamente. Prueba el siguiente comando:

    ```BASH
    curl "http://localhost:4300/route/v1/driving/-68.06888809573269,-16.52954149841827;-68.07729724972369,-16.539156720555397?overview=full&geometries=geojson&steps=false"
    ```

    El resultado debería ser exitoso, un `200 OK` si tienes problemas revisa los pasos anteriores, si quieres probar otras ubicaciones según el archivo que descargaste, esta es el formato de la url que probaste:

    ```TEXT
    http://{HOST}:{PORT}/route/v1/driving/{LONGITUDE_INITIAL,LATITUDE_INITIAL};{LONGITUDE_END,LATITUDE_END}?geometries=geojson
    ```

    El query param especifica que tipo de coordenadas le estas enviando en el request, en este caso se esta usando [geojson](https://datatracker.ietf.org/doc/html/rfc7946), donde se maneja el formato `coordinates: [LNG, LAT]`, para el calculo requiere mínimamente 2, deben estar separadas por `;` puedes ver mas sobre el [API de OSRM](https://project-osrm.org/docs/v5.24.0/api/) en la documentación completa.
