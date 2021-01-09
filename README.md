# Node-RED-Docker

Plantilla de projecto para correr una app node-red en un docker.

Para hacer un bundle de una app Node-RED completa se necesita: 

1. application flow 
2. credential file
3. package.json file, que tienen los nodos que Node.js y Node-RED necesitan para los flows
4. los source files del runtime Node-RED


The code in this repository is to provide a starting project for Node-RED, where the aim is to run in a production, cloud based environment.  There have been a few modifications over the standard Node-RED released code.

## Como se usa:

los tutoriales de donde sale este paso a paso están: [aquí](https://github.com/binnes/Node-RED-container-prod)

1. hacer un clone de este repo. Y tendremos un docker con imagen limpia de NR --> a standard install of Node-RED with the [projects](https://nodered.org/docs/user-guide/projects/) feature enabled.
2. sustituir el fichero flows.json en este repo, por un fichero flows.json que incluye los flujos que quiero meter en mi imagen de docker de la app NR. 
3. hago lo mismo, con el fichero flows_cred.json 
4. en el fichero package.json, si estamos usando nodos que no vengan en el NR estandard, actualizamos la lista de dependencias y los incluímos; los módulos que vayan a ser necesarios en los flujos que estan en flows.json anterior.
5. el fichero settings.js incluido en este repo, tiene modificado el original, para activar la opcion NR de "proyectos", que hace aparecer una pestaña de integración con github. Si lo editamos, podemos ver cómo se activa esta opción en una instalación NR normal):
        
        projects: {
        // To enable the Projects feature, set this value to true
        enabled: true
6. cuando la app NR funcione, y esté lista para construirse una imagen, se usa el Dockerfile que está incluido en este repo para hacer build de la app. La funcion que hace el Dockerfile es "descubrir" todas las dependencias que tiene mi app y que no están declaradas en NR, sino que son dependencias del runtime que usa NR (que se llama Node.js) y que son en realidad cosas que el docker "espera" que estén en el SO del host en el que corre.
7. arrancamos un docker NR que tire del repo:

        docker run -itd -p 1880:1880 -v /home/lu4t/repos-github/NRdata:/data -e NODE_RED_ENABLE_PROJECTS=true -e TZ=Europe/Madrid --name lu4tNR nodered/node-red

8. con el NR corriendo, verificamos que tiene el flujo que queremos "empaquetar" en la app dockerizada. Abrimos http://localhost:1880 y comprobamos.
9. activamos la funcion buildx de docker, y activamos la opción de emulador de arquitecturas (permite hacer build de un docker multi-arquitectura).
        
        export DOCKER_CLI_EXPERIMENTAL=enabled
        
        docker run --rm --privileged docker/binfmt:66f9012c56a8316f9244ffd7622d7c21c1f6f28d
        
10. Desde el mismo directorio donde esté el Dockerfile, creamos un nuevo builder:
        
        docker buildx create --name NRbuilder --use
 
11. Arrancamos el proceso de build:

        docker buildx inspect --bootstrap
        
12. nos logueamos en docker (o el repositorio de imagenes que vamos a usar), y arrancamos el proceso de build y luego le hacemos un push a dockerhub:

         docker login
         docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 -t lu4t/nr-docker-sample-lu4t --push .
         
13. cuando se halla completado el build, aparecerá la imagen de un container en el repo de docker, bajo mi cuenta de docker.

14. para descargar y probar el docker que acabo de crear, verifico que no está corriendo en local ningún otro NR, y ejecuto:

        docker run -dit -p 1880:1880 --name mi-primer-docker lu4t/nr-docker-sample-lu4t:latest
        
        
         
        

## Changes from a standard Node-RED install

### Node-RED Editor

In a production system the Node-RED editor should be disabled.  However, in this project the editor has been moved from the root (**/**) endpoint to the **/admin** endpoint to allow the application to be examined within the container.

It is recommended that the editor be disabled for any production workload.  This is achieved by modifying the **settings.js** file to change the **httpAdminRoot** property from **"/admin"** to **false** : `**httpAdminRoot: false,**`

For more details about configuring the runtime using the **settings.js** file see the [configuration section](https://nodered.org/docs/user-guide/runtime/configuration) of the Node-RED documentation.

### Health endpoints

When running containers within a managed cloud environment, it is often useful to provide some endpoints within the container, so the cloud management system can verify the container is still alive and responding to requests.

This project adds the Health Checking capability from [CloudNativeJS.io](https://www.cloudnativejs.io).  The integration has be done in the **server.js** file by adding the following lines of code:

```JavaScript
var health = require('@cloudnative/health-connect');

var healthcheck = new health.HealthChecker();
app.use('/live', health.LivenessEndpoint(healthcheck));
app.use('/ready', health.ReadinessEndpoint(healthcheck));
app.use('/health', health.HealthEndpoint(healthcheck));
```
