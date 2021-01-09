# Node-RED-Docker

Plantilla de projecto para correr una app node-red en un docker.

The code in this repository is to provide a starting project for Node-RED, where the aim is to run in a production, cloud based environment.  There have been a few modifications over the standard Node-RED released code.

## Como se usa:

1. using a standard install of Node-RED with the [projects](https://nodered.org/docs/user-guide/projects/) feature enabled, clone a fork of this repository
2. sustituir el fichero flows.json en este repo, por un fichero flows.json que incluye los flujos que quiero meter en mi imagen de docker de la app NR. 
3. hago lo mismo, con el fichero flows_cred.json 
4. en el fichero package.json, si estamos usando nodos que no vengan en el NR estandard, actualizamos la lista de dependencias y los incluímos; los módulos que vayan a ser necesarios en los flujos que estan en flows.json anterior.
5. el fichero settings.js incluido en este repo, tiene modificado el original, para activar la opcion NR de "proyectos", que hace aparecer una pestaña de integración con github. Si lo editamos, podemos ver cómo se activa esta opción en una instalación NR normal):
        
        projects: {
        // To enable the Projects feature, set this value to true
        enabled: true
6. cuando la app NR funcione, y esté lista para construirse una imagen, se usa el Dockerfile que está incluido en este repo para hacer build de la app.

tutorials [aquí](https://github.com/binnes/Node-RED-container-prod)

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
