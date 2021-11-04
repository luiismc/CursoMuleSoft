# Documentación de Curso MaveSoft
## Parte 1: Hello Mule - Luis Fernando Marin Calderon

Durante esta parte del curso se genero y lanzo una aplicación de Mule con un modulo HTTP (Listener) que retorna el texto 
>>Hello Mule

### Puntos vistos:
1. Instalación del entorno de AnyPoint Studio.
2. Exploración general del GUI de AnyPoint Studio.
3. Configurar un conector HTTP.
4. Pruebas con Postman.
5. Lanzar la aplicación a CloudHub.


### Instalacion del entorno de Anypoint Studio.
Utilizando el asistente  de Anypoint Studio se instalo el IDE dentro de la raiz del disco duro. Este mismo IDE contiene los archivos de Java para su uso, por lo que no fue necesario
la instalación de Java.

### Exploración general del GUI de AnyPoint Studio.
Se exploro los distintos elementos que ofrece la interfaz de Anypoint Studio, entre ellos el explorador de archivos, el Canvas donde se podra trabajar de manera mas grafica, consola, propiedades entre otros.

### Configurar un conector HTTP.
Se creo un proyecto Mule dentro del IDE, posteriomente utilizando la paleta de elementos que ofrece el canvas se arrastro un conector HTTP (Listener) hacia el canvas para configurar su conexión (host, puerto, protocolo) y la dirección que tendra. Abajo se presenta la confiruación que se utilizo:

- Protocolo: HTTP
- Host: localhost
- Puerto: 8081
- Dirección: /hellomule

Despues se agrego un payload que contiene el valor de Hello Mule como String.

### Pruebas con Postman.
Despues de guardar y correr el proyecto Mule se utilizó Postman para hacer una petición GET a la aplicación para realizar una prueba.

- Dirección:  0.0.0.0:8081/hellomule
- Petición: GET
- Resultado: 200 "Hello Mule"

### Lanzar la aplicación a CloudHub.
Se realizó el registro de una cuenta gratuita dentro de CloudHub y se autentifico dentro de Anypoint Studio utilizando la función de "Deploy to CloudHub Platform" dando click derecho al proyecto. Se creo una nueva aplicación dentro de la plataforma con el nombre "hellomule-guiasps" para subir el proyecto y posteriormente deployarlo. 
La dirección de la aplicación [es la siguiente] (hellomule-guiasps.us-e2.cloudhub.io)

Resultado de prueba con Postman: 
- Petición: GET
- Resultado: 200 "Hello Mule"

----------------------------------------------

## Parte 2: Configurar un endpoint HTTPS
Durante esta parte del curso se cambio el protocolo del servicio pasado Hello Mule a HTTPS, utilizando las funciones de keytools y Anypoint Studio. 

Se creo el archivo dentro del directorio "src/main/resources"

- "local.properties" Contiene las propiedades de seguridad para el proyecto. En este caso contiene la propiedad del puerto que estara escuchando la aplicación (8082).
- Dentro de la terminal se ejecuto el comando : 
>keytool -genkey -alias key-alias -keystore keystore-name.jks -keyalg RSA -storetype JKS
Y se ingreso la información necesaria para generar el almacen de llaves. Se creo un archivo .jks y este se guardo en la carpeta "src/main/resources"
- Se modifico las configuraciones de conexion del Listener de la aplicación. Se cambio el protocolo a HTTPS, se especificó el puerto utilziando la variable dentro de local.properties, se agrego la información y las contraseñas para el archivo .jks dentro de la configuracion TLS y tambien se agrego el mismo archivo como propiedades de configuración del proyecto.
- Por ultimo se realizó un test de conexión con esta nueva configuración el cual resulto Exitoso. Posteriormente tambien se realizo la prueba en Postman con la siguiente dirección
>https://0.0.0.0:8082/hellomule 

**Resultado** >  200, "Hello Mule" (Se desactivo la validación del certificado SSL)

----------------------------------------------
## Parte 3: Añadir seguridad a las propiedades.
En esta etapa el objetivo es aprender a añadir seguridad a todas las propiedades que podrian manejar información sensible o datos que se podrian llamar alrededor del proyecto.

1. Archivos de seguridad.
En esta fase se crearon dos archivos de seguridad (local.secure.properties y dev.secure.properties), dentro de cada una se añadieron las propiedades de usernames y password para sus respectivos ambientes (dev y local).

2. Agregar dependencia.
Se utilizo el gestor de dependencias "Search in Exchange" de AnyPoint Studio para buscar el modulo "Mule Secure Configuration Properties" y posteriormente descargarlo y añadirlo al proyecto. Una vez instalada, se agrego el nuevo elemento "Secure Properties Config" dentro de la configuración global. Dentro de las propiedades se indicaron los archivos , la llave para desencriptar y el algoritmo Blowfish. Se utilizaron variables globales para esta configuración. (Parte 2 del Curso de MuleSoft).

3. Encriptar contraseñas.
Se descargo la herramienta "secure-properties-tool.jar" de la doc oficial de MuleSoft para encriptar las contraseñas y usuarios que se encuentran en los archivos de secure.properties.

**Comando que se utilizo en el SImbolo del Sistema Windows** > java -cp secure-properties-tool.jar com.mulesoft.tools.SecurePropertiesTool  string  encrypt  Blowfish  CBC  MyMuleSoftKey  "myPasswordLocal" 

(Se cambio el string "myPasswordLocal" por cada usuario y contraseña para generar su propio texto encriptado.)


4. Testing.
Al guardar todos los cambios, antes de correr el proyecto se agrego una propiedad al ambiente de ejecución que contiene la llave que se utilizo para hacer el encriptado de usuarios y contraseñas. Una vez el proyecto en ejecución se hizo una prueba de postman, trayendo el mismo resultado de "Hello Mule" pero con la diferencia que en los logs de la aplicación mostraba el usuario y contraseña sin encriptar. 

5. Deploy.
Se sube los cambios del proyecto dentro de la aplicación que tenemos ya creada en CloudHub, tambien se agrega las variables de ambiente env=dev para probar las propiedades en dicho ambiente. Los logs de CloudHub mostraron el usuario y contraseña de dev.secure.properties al momento de hacer una petición atravez de postman.

----------------------------------------------
## Parte 4: API Autodiscovery.

### ApiManager ###
Al ya tener lista la seguridad de las propiedades del proyecto, se procedio a crear una API desde el Anypoint Platform y configuramos su endpoint como una aplicación Mule hosteada en CloudHub. Generamos el API Autodiscory API ID que nos ayudara a vincularla con nuestra aplicación.
**API ID: 17400382**
Agregamos el valor de api.id a cada archivo de propiedad que tenemos en el proyecto (dev.properties y local.properties).

### API Autodiscovery ###
Dentro de nuestro archivo global.xml agregamos el elemento API Autodiscovery para indicar la propiedad api.id . Por ultimo se extrajo nuestra Client ID y Client Secret para posteriormente incluirlas dentro de las preferencias del proyecto > Anypoint Studio > Api Manager y validamos el status.

### Deploy en CloudHub ###
Al momento de quere hacer un deploy desde Anypoint Studio, verificamos que en las propiedades se encontraban los nuevos parametros (client id, client secret, api id) y realizamos el update. Dentro del API Manager nos marca **Api status: Active** indicando que si se realizo la conexion entre estas plataformas.

----------------------------------------------
## Parte 5: Añadir politica de autorización con Client ID. 
Agregamos la politica de autorización por Cliente ID desde el API Manager a nuestra aplicación de Hello Mule. Posteriormente generamos las credenciales desde Exchange:

Client ID:
18191e498979467da14b4751280544cb
Client Secret:
d07Ab89923664B489393f50D1160EFaE

Si no agregamos estas credenciales al postman nos arrojara un error de autorización.

----------------------------------------------
## Parte 6: Crear una Especificación API con API Designer.
 En esta parte del curso aprendimos lo que es una especificación API, que es basicamente la interfaz de servicios y valores esperados de una API. En este caso creamos rutas de Direcciones y contactos, con sus debidos parametros, a si como tambien especificamos los protocolos a utilizar y el formato en la que se incluye la información (JSON).

 Generamos un link publico para pruebas de esta especificación API: 

Public link: https://anypoint.mulesoft.com/mocking/api/v1/links/d4926f3e-2a61-4c44-a824-9d39750b5071





## DATOS EXTRAS:
En el archivo mule-artifact.json encontramos las propiedades que deben de estar seguras del proyecto.
El archivo mule-artifact.json es una lista de dependencias para Maven.