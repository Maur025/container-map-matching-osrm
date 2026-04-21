# Map Matching - OSRM

Environment to up software of OSRM to fetch data to routes, matches or anything reference to geolocation, which includes calculations.

> **NOTE**
> this container only works in Linux systems, at the time I'm writing this.

The following steps are Spanish for your easy understanding, add steps in English soon.

## Setup server OSRM (Es)

1. Descarga el archivo de datos de tu interés podría ser de tu país o región, es necesario un archivo pbf para continuar con los siguientes pasos.

    Por ejemplo: puedes usar el siguiente comando:

    ```BASH
    wget https://download.geofabrik.de/south-america/bolivia-latest.osm.pbf
    ```

    Sirve para descargar los datos de geolocalización de Bolivia.

2. Renombra el archivo a `geo-data-latest.osm.pbf`

3. Crea la carpeta `data` en el directorio donde tienes el archivo docker-compose.yml

    ```TEXT
    └── carpeta-contenedora-osrm
        ├── /data   #carpeta a crear
        ├── /options
        │   └── bolivia.lua
        └── docker-compose.yml
    ```

4. Copia el archivo descargado y renombrado `geo-data-latest.osm.pbf`, en la carpeta que creaste con el nombre `data`.

    ```TEXT
    └── carpeta-contenedora-osrm
        ├── /data
        │   └── geo-data-latest.osm.pbf
        ├── /options
        │   └── bolivia.lua
        └── docker-compose.yml
    ```

5. Abre `docker-compose.yml` y revisa el contenido, si el servicio `osrm-setup` esta comentado, quita temporalmente los `#` para usarlo. Podrías encontrarlo de esta forma.

    ```YML
    # osrm-setup:
      # container_name: osrm-setup
    ```

    Solo remueve los símbolos, guarda y continua con el siguiente paso.

6. Notarás que existe una carpeta con el nombre de `options`, aquí puedes cambiar la configuración que usa OSRM para realizar los cálculos, actualmente tiene un archivo con el nombre `bolivia.lua` esta creado en base a `car.lua`, adaptando o simulando el comportamiento en Bolivia, para que los tiempos no sean muy optimistas.

7. Si tienes tu propia configuración, o si deseas usar la lógica de otro país puedes reemplazar este archivo, pero no olvides hacer referencia a el antes de ejecutar el servicio `osrm-setup` dentro del archivo `docker-compose.yml`

    ```YML
    services:
        osrm-setup:
            # El otro código
            command: sh -c "osrm-extract -p /options/{nombre-de-tu-archivo}.lua /data/geo-data-latest.osm.pbf && osrm-contract /data/geo-data-latest.osrm"
    ```

    > Si quieres usar la configuración por defecto de osrm después de `-p` usa esto `/data/car.lua`, también puedes usar otras configuraciones propias de OSRM, inclusive podrías volver más compleja la inicialización.

8. Ahora continuando debemos preparar los datos y seguidamente ejecutar el servidor de OSRM, tienes 2 opciones para hacerlo.

    > **NOTA**
    > En un escenario ideal, el servicio `osrm-setup` que sirve para preparar los datos solo se debería ejecutar una sola vez. Reduciendo el tiempo de inicialización del servidor OSRM, si lo ejecutas en cada ocasión podría demorar en estar disponible.

9. **Opción 1**, puedes ejecutar por separado cada servicio, verificando que sigan con el orden correcto.

    Primero ejecuta el servicio `osrm-setup` para preparar los datos:

    ```BASH
    docker compose run osrm-setup
    ```

    Segundo ejecuta el servicio `map-osrm-server` que levanta el servidor para consultar y realizar los cálculos:

    ```BASH
    docker compose run map-osrm-server
    ```

    Cuando necesites el servidor principal solo deberías ejecutar el segundo comando, teniendo en cuenta que ya tienes los datos.

10. **Opción 2** puedes dejarle el trabajo a es script la primera vez, pero para usarlo después debes hacer unas modificaciones.

    Primero ejecuta este comando que se asegurará de hacer todo lo necesario para levantar el servidor:

    ```BASH
    docker compose up
    ```

    Si no quieres ver lo que hace internamente puedes usar `-d` para ocultarlo, además que evitará que bloquee la terminal.

    Luego, cuando lo tengas disponible debes comentar el servicio `osrm-setup` en el archivo `docker-compose.yml` de esta forma:

    ```YML
    # osrm-setup:
    #   # profiles:
    #   #   - setup
    #   container_name: osrm-setup
    #   image: osrm/osrm-backend:v5.27.1
    #   hostname: osrm-setup
    #   command: sh -c "osrm-extract -p /options/bolivia.lua /data/geo-data-latest.osm.pbf && osrm-contract /data/geo-data-latest.osrm"
    #   volumes:
    #     - ./data:/data
    #     - ./options:/options
    #   networks:
    #     - compose-net
    ```

    También comenta esto en el servicio `map-osrm-server`:

    ```YML
      map-osrm-server:
        container_name: map-osrm-server
        # el otro código

        # depends_on:
        #     osrm-setup:
        #       condition: service_completed_successfully
    ```

    Esto evitará que te de un error de dependencia inexistente:

    Por ultimo cada vez que requieras el servidor solo debes usar:

    ```BASH
    docker compose up -d
    ```

De esta forma ya deberías tener disponible el servidor de cálculos de OSRM, puedes probarlo desde `postman` o por `curl` para asegurarte de que esta funcionando correctamente. Prueba el siguiente comando:

```BASH
curl "http://localhost:4300/route/v1/driving/-68.06888809573269,-16.52954149841827;-68.07729724972369,-16.539156720555397?overview=full&geometries=geojson&steps=false"
```

El resultado debería ser exitoso, un `200 OK` si tienes problemas revisa los pasos anteriores, si quieres probar otras ubicaciones según el archivo que descargaste, esta es el formato de la url que probaste:

```TEXT
http://{HOST}:{PORT}/route/v1/driving/{LONGITUDE_INITIAL,LATITUDE_INITIAL};{LONGITUDE_END,LATITUDE_END}?geometries=geojson
```

El query param especifica que tipo de coordenadas le estas enviando en el request, en este caso se esta usando [geojson](https://datatracker.ietf.org/doc/html/rfc7946), donde se maneja el formato `coordinates: [LNG, LAT]`, para el calculo requiere mínimamente 2, deben estar separadas por `;` puedes ver mas sobre el [API de OSRM](https://project-osrm.org/docs/v5.24.0/api/) en la documentación completa.
