# info de la materia: ST0263 <nombre>

**Estudiantes**: 
- **Autor**: Juan Felipe Pinzón Trejo - jfpinzont@eafit.edu.co
- **Coautora**: Maria Paula Ayala - mpayalal@eafit.edu.co

**Profesor**: Edwin Montoya -  emontoya@eafit.edu.co

**Título:** P2P - Comunicación entre procesos mediante API REST, RPC y MOM

**Objetivo:** Diseñar e implementar una red P2P para soportar un sistema distribuido de manejo de archivos.

**Sustentación:** presionando [aquí](https://eafit-my.sharepoint.com/:v:/g/personal/jfpinzont_eafit_edu_co/EVTzQbQimuFIqAFPKB_VXmwBXrqjZcjPuuTdaLaeWo-JBw?e=l27usd)

# 1. breve descripción de la actividad

## 1.1. Que aspectos cumplió o desarrolló de la actividad propuesta por el profesor 

En este reto se cumplió con la arquitectura de **red P2P no estructurada basada en servidor**, la cual será detalla en el apartado 2. Mediante esta, se logró la comunicación entre pServer, pClient (ambas entidades correspondientes al **peer**) y ServerCentral (entidad del **servidor central** de la red). Toda la comunicación se realiza por medio de **API REST**. 

Además, el cliente puede conectarse y desconectarse a la red. También se implementó las funciones dummy para subir y descargar archivos, comunicandose siempre por medio de un peer vecino que funciona como **relay server**. 

## 1.2. Que aspectos NO cumplió o desarrolló de la actividad propuesta por el profesor 

- No se realizó la comunicación por medio de gRPC.
- Hizo falta también el ping constante para verificar que los peers siguieran activos dentro de la red.
- Tampoco se guardan los datos de IP ni Puerto dinámicamente, cuando se inician el pServer y el pClient piden esta información por medio de la consola.
- No se libera un directorio donde se encuentran los archivos, se deben "subir" a la red mediante el servicio de upload y se almacena el índice en el pServer

# 2. información general de diseño de alto nivel, arquitectura.
![arqReto1y2](https://github.com/juanfepi27/jfpinzont_st0263/assets/85038450/64e52364-5540-4955-b0a4-f8027fa9fb8e)


# 3. Descripción del ambiente de desarrollo y técnico: 

## 3.1 MVP
Todo el proyecto fue desarrollado en el lenguaje Python usando el framework Flask, a la vez que las librerías requests y otras nativas. Este tiene una estructura como se puede observar a continuación:

```bash
  Reto1y2
  |
  |- ServerCentral
  |  |
  |  |- app.py
  |
  |- Peer
  |  |
  |  |- pClientApp.py
  |  |- pServerApp.py
```
La primera carpeta, ServerCentral, contiene solo un archivo, app.py, el cual fue realizado con el framework Flask y utiliza API REST para comunicarse con el pServer. Ahora, para entrar más a detalle, se explicará lo que hace cada ruta dentro de este archivo.

- /login [POST]: Este recibe los datos de IP y Puerto del pServer del cliente que se está conectando. Luego se revisa si ya hay más peers conectados a la red, de ser así, se escoge al último peer conectado como el vecino de este nuevo peer y se crea el objecto Peer con los datos ingresados; después, si había un peer pendiente por vecino se le asigna a este nuevo peer como su vecino. Si no existían peers antes se crea el objeto Peer y se añade a los peers pendientes por vecinos. Retorna el id con el que queda registrado el peer entrante y la url del vecino en caso de existir o None, en caso de no existir.

- /logout [POST]: Este método recibe el id del peer que se va a desconectar y lo primero que revisa es si este tenía archivos guardados, de ser así lo elimina de la "base de datos", que en este caso es un diccionario. Después revisa si tenía algún vecino asignado o si estaba como vecino de alguien más y los reasigna. Por último, se elimina su información de la "base de datos" de peers. Simplemente retorna un mensaje de verificación.

- /uplaod [POST]: Recibe el id del peer que quiere subir un archivo y el nombre del archivo que quiere guardar. Luego, se escoge aleatoriamente un peer en el cual guardar este archivo, ignorando al peer que quiere subirlo. Después de escoger a qué peer va, el servidor se comunica con el pServer de ese peer y espera una respuesta de confirmación, si el pServer devuelve una respuesta afirmativa, se guarda el nombre del archivo en la "base de datos" con el id del peer en donde se guardó.

- /sendFiles [GET]: En este método solo se mandan los nombers de los archivos guardados en la "base de datos". Retornando el índice en formato de lista.

- /sendFileOwner [POST]: Se recibe el nombre del archivo que se quiere descargar, se busca cuál es el primer peer registrado que tiene guardado este archivo y se retorna la URL de este.

- /checkClientNeighbour [POST]: Este método recibe el id del peer que mandó la petición y revisa si este tiene ya asignado algún vecino, de ser así le retorna la URL de este, si no, retorna un None.

Ahora, dentro de la carpeta Peer encontramos dos archivos, el primero es pClientApp.py, el cual es con el que el usuario interactúa. Dentro de este encontramos los siguientes métodos:

- display_menu(): Se muestra al usuario el menú de opciones que este puede y se llama al respectivo método según lo que escoja el usuario.

- upload(): Primero revisa que tenga un vecino asignado, de ser así, llama al servidor de este vecino y le manda el nombre del archivo que se quiere guardar. Si no tiene un vecino asignado no puede realizar esta operación y debe esperar a que otro peer se conecte.

- download(): Primero revisa que tenga un vecino asignado, si no lo tiene no puede realizar esta operación y debe esperar a que otro peer se conecte a la red. Si ya tiene un vecino asignado, le pide a este los archivos que se encuentran en la "base de datos" de la red y el usuario escoge cuál es el archivo que quiere descargar. Después, le pregunta al servidor del vecino por la URL de dónde se encuentra este archivo, este se lo devuelve y por último el cliente se conecta con el servidor que lo tiene y lo descarga.

- logout(): Se llama al propio servidor para que este realice la desconexión del servidor central y se apaga el cliente.

- check_neighbour(): Llama a su servidor para actualizar, en caso de ser posible, y guardar la información en la variable de vecino.

- main: Al momento de iniciar el programa se piden los datos del puerto del propio servidor(pServer) y del puerto por el que escuchará en la red (la demás información referente a las IPs se deja quemada en código, puesto que es un ambiente local y siempre será 127.0.0.1). Después se envía la información del propio pServer al servidor central para poder conectarse a la red y quedar a la espera de la información del posible peer vecino. 

Por úlitmo, tenemos el archivo pServerApp.py, el cual fue realizado con el framework Flask y utiliza API REST para comunicarse con el servidor central. Este archivo actúa más como un intermediaro entre el cliente y el servidor central, por lo que se encuentran las siguientes rutas:

- /askForFiles [GET]: En este método se le solicitan los nombres de los archivos que se encuentren en la "base de datos" al servidor central y se le devuelve esta información al cliente.

- /checkNeighbour [POST]: Se comunica con el servidor central para encontrar la URL del peer vecino y se la envía al cliente.

- /searchFileOwner [POST]: Le soliciata al servidor central la URL del peer en donde se encuentra un archivo en específico y se la devuelve al cliente.

- /saveFile [POST]: El servidor central le envía la información de un archivo y este lo guarda en su "base de datos", que es un arreglo de nombres de archivos y retorna la correcta ejecución.

- /notifyLogout [POST]: Se comunica con el servidor central para avisarle que ese peer se va a desconectar de la red y retorna un aviso cuando ya puede salir el cliente, una vez el server central ha eliminado su información.

- /download [POST]: Revisa que en su "base de datos" esté el archivo que se quiere descargar, si es así, lo devuelve con éxito.

- /fileToUpload [POST]: Se comunica con el servidor central para guardar un nuevo archivo, en donde le envía el id del peer que lo está subiendo y el nombre del archivo.

## 3.2 Ejecución.

### Prerrequisitos
1. Debes clonar el repositorio y tomar la carpeta **Reto1y2**
1. Debes tener una versión de Python 3.7 o superior (**actualmente se está usando la 3.11.4**)
1. Debes tener el gestor de paquetes pip, **actualmente se está usando la versión 23.3**
1. Debes de descargar las librerías del archivo `requirements.txt` así:
```bash
pip install -r requirements.txt
```

### Ejecución del sistema
Se hará el procedimiento para la conexión de 1 Peer

1. Lo primero que debe de encenderse es el server central, abrimos una consola y nos ubicamos en la carpeta `Reto1y2/ServerCentral` y allí ejecutamos el programa `app.py`, y debería de aparecernos algo así:

    ```bash
    >>>python app.py
    * Serving Flask app 'app'
    * Debug mode: on
    WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
    * Running on http://127.0.0.1:5000
    Press CTRL+C to quit   
    * Restarting with stat
    * Debugger is active!
    * Debugger PIN: 463-256-156
    ```
2. luego abriremos 2 consolas para nuestro Peer, 1 para su pServer y otra para su pClient, en ambas debemos de ubicarnos en la carpeta `Reto1y2/Peer`.
    1. En la consola del pServer ejecutaremos el programa `pServerApp.py` y llenaremos el puerto por el que escuchará, en este caso será el 5001, y debería de verse algo así

        ```bash
        >>>python pServerApp.py
        port:5001
        * Serving Flask app 'pServerApp'
        * Debug mode: on
        WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
        * Running on http://127.0.0.1:5001
        Press CTRL+C to quit
        * Restarting with stat
        port:5001
        * Debugger is active!
        * Debugger PIN: 463-256-156
        ```

        >Por alguna razón aún desconocida pregunta 2 veces por el puerto, por ello en el ejemplo aparece 2 veces la solicitud "port:"

    2. En la consola del pClient ejecutaremos el programa `pClientApp.py` y llenaremos el puerto por el que escuchará su pServer, en este caso será el 5001, luego por el que escuchará él mismo, que en este caso será el 5002, finalmente debería de verse algo así

        ```bash
        >>>python pClientApp.py
        pserverPort:5001
        port:5002
        pending for neighbour assignment      
        ------------------------------------- 
            What do you want to do:
            [1]. upload
            [2]. download
            [3]. logout

            insert the NUMBER and press enter:
        ```

3. Finalmente ya está configurado para su uso, sin embargo no puede usarse por completo hasta que se conecte otro peer, para ello replique el paso 2.

> Finalmente para más detalle visualize el video explicativo, presionando [aquí](https://eafit-my.sharepoint.com/:v:/g/personal/jfpinzont_eafit_edu_co/EVTzQbQimuFIqAFPKB_VXmwBXrqjZcjPuuTdaLaeWo-JBw?e=l27usd)

# referencias:
Toda la información requerida se tomaron de IA generativas, estas no brindan la totalidad de las fuentes que utilizan, sin embargo adjunto las conversaciones utilizadas para el sustento de este proyecto:
- https://chat.openai.com/share/7d768577-fdc7-4088-93b7-983c18c369e0
