
En esta folder se almacena archivos Containerfiles, DevContainers, S2I, ....
Para la creacion de imagen se usa script bash de mi '.files' ubicado en el repo 'https://github.com/lestbanpc/dotfiles':

- Script bash ubicados en './shared/linux/' .....
- Script bash ubicados en './setup/linux/' .....


# ContainerFiles> Creando imagenes usando Podman/BuildAh

Para crear una imagen debera seguir los siguientes pasos ..

1. Ir a ubicacion del archivo Dockerfile

   ```shell 
   #Ir a ubicacion del la carpeta de imagen Cliente Oracle 
   cd ~/code/files/images/containerfiles/10_devtools/02_dbclient/oracle-cli/
   #Ir a ubicacion del la carpeta de imagen de utilitarios de Networking
   cd ~/code/files/images/containerfiles/10_devtools/01_networking/networking-tools/
   
   #Analizar la data
   bat ./1.0-x64-fed39/Dockerfile
   vim ./1.0-x64-fed39/Dockerfile 
   ```


2. Compilar la imagen

   ```shell 
   #Creando la imagen "Cliente Oracle" en repositorio del usuario (rootless)
   podman build --build-arg UID=1000 --build-arg GID=1000 -t lucianoepc/oracle-cli:1.0-x64-fed39 ./1.0-x64-fed39/
   #Creando la imagen "Networking Tools" en repositorio del usuario (rootless)
   podman build --build-arg UID=1000 --build-arg GID=1000 -t lucianoepc/networking-tools:1.0-x64-alp3.19 ./1.0-x64-alp3.19/
   ```

3. Prueba: ejecutando un contenedor en background:

   ```shell 
   #....
   podman run --name oracle-cli-1 -d lucianoepc/oracle-cli:1.0-x64-fed39
   podman container ls --all
   podman exec -it oracle-cli-1 /bin/bash
   podman container kill oracle-cli-1
   
   #Imagen para para pruebas de Networking (termina rapido ... modificar el entry point)
   podman run --name networking-1 -d lucianoepc/networking-tools:1.0-x64-alp3.19
   podman container ls --all
   podman exec -it networking-1 /bin/bash
   podman container kill networking-1
   ```

4. Prueba: ejecutando un contenedor en forma interactiva:

   ```shell 
   #....
   podman run --name oracle-cli-1 -it --rm lucianoepc/oracle-cli:1.0-x64-fed39
   
   #Imagen para para pruebas de Networking (no termina hasta ...)
   podman run --name networking-1 -it --rm lucianoepc/networking-tools:1.0-x64-alp3.19
   ```

5. Mantemiento del Container-Runtime

   ```shell 
   #Eliminar la capas de imagenes temporales generados durante la contruccion
   podman image prune
   ```

7. Subir la imagen a registro de imagenes

   ```shell 
   #Subir al repositorio doker.io/lucianoepc (si no esta registrado el auto-login en ..)
   podman login -u lucianoepc
   podman push lucianoepc/networking-tools:1.0-x64-alp3.19
   ```

# ContainerFiles> Creando imagenes usando ContainerD/BuildKit

Para crear una imagen debera seguir los siguientes pasos ..


1. Inicializando

   ```shell 
   #..
   systemctl --user start containerd.service
   
   #...
   systemctl --user start buildkit.service
   ```

2. Ir a ubicacion del archivo Dockerfile

   ```shell 
   #Ir a ubicacion del la carpeta de imagen Cliente Oracle 
   cd ~/code/files/images/containerfiles/10_devtools/02_dbclient/oracle-cli/
   #Ir a ubicacion del la carpeta de imagen de utilitarios de Networking
   cd ~/code/files/images/containerfiles/10_devtools/01_networking/networking-tools/
   
   #Analizar la data
   bat ./1.0-x64-fed39/Dockerfile
   vim ./1.0-x64-fed39/Dockerfile 
   ```


3. Compilar la imagen

   ```shell 
   #Creando la imagen "Cliente Oracle" en repositorio del usuario (rootless)
   nerdctl build --build-arg UID=1000 --build-arg GID=1000 -t lucianoepc/oracle-cli:1.0-x64-fed39 ./1.0-x64-fed39/
   #Creando la imagen "Networking Tools" en repositorio del usuario (rootless)
   nerdctl build --build-arg UID=1000 --build-arg GID=1000 -t lucianoepc/networking-tools:1.0-x64-alp3.19 ./1.0-x64-alp3.19/
   ```

4. Prueba: ejecutando un contenedor en background:

   ```shell 
   #....
   nerdctl run --name mssql-cli-1 -d lucianoepc/mssql-cli:1.0-x64-alp3.19
   nerdctl container ls --all
   nerdctl exec -it mssql-cli-1 /bin/bash
   nerdctl container kill mssql-cli-1
   
   #Imagen para para pruebas de Networking
   nerdctl run --name networking-1 -d lucianoepc/networking-tools:1.0-x64-alp3.19
   nerdctl container ls --all
   nerdctl exec -it networking-1 /bin/bash
   nerdctl container kill networking-1
   ```

5. Prueba: ejecutando un contenedor en forma interactiva:

   ```shell 
   #....
   nerdctl run --name oracle-cli-1 -it --rm lucianoepc/oracle-cli:1.0-x64-fed39
   
   #Imagen "Networking Tools"
   nerdctl run --name networking-1 -it --rm lucianoepc/networking-tools:1.0-x64-alp3.19
   ```

6. Mantemiento del Container-Runtime

   ```shell 
   #....
   ...
   ```

7. Subir la imagen a registro de imagenes

   ```shell 
   #Subir al repositorio doker.io/lucianoepc
   nerdctl login -u lucianoepc
   nerdctl push lucianoepc/networking-tools:1.0-x64-alp3.19
   ```

# ContainerFiles> Usando la imagen creada en Kubernates




1. Alternativa 1: ....

   ```shell 
   #....
   oc run net-tools1 -it --rm --image=lucianoepc/basic-tools:1.0.0-alpine --restart=Never -- bash
   ```


# ContainerFiles> Usando la imagen creada en Kubernates (RH Openshift)

Las imagenes creadas usan un ID ... por lo que requiere que antes se relize la siguiente configuracion:
- Crear/Configurar un proyecto donde ...
- ....

Luego de esta configuracion use:

1. Alternativa 1: 

   ```shell 
   oc run podo-mssql-cli-1 --overrides='
   {
     "spec": {
       "serviceAccount": "sacc-with-anyuid",
       "serviceAccountName": "sacc-with-anyuid",
       "containers": [
         {
           "name": "cntr-main",
           "image": "lucianoepc/mssql-cli:1.0-x64-alp3.20",
           "command": [
             "tail"
           ],
           "args": [
             "-f",
             "/dev/null"
           ],
           "imagePullPolicy": "Always",
           "securityContext": {
             "allowPrivilegeEscalation": true,
             "runAsNonRoot": false,
             "runAsUser": 1000,
             "runAsGroup": 1000
           }
         }
       ]
     }
   }
   ' --image=dummy --restart=Never
   ```
   
   Comandos a usar:

   ```shell 
   oc exec pod/podo-mssql-1 -itn ppoc-lpenac -- bash
   
   oc delete pod/podo-mssql-1 -n ppoc-lpenac
   ```

2. Alternativa 2
   
   Crear el yaml:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: podo-mssql-cli-1
     namespace: ppoc-lpenac
   spec:
     serviceAccount: sacc-with-anyuid
     serviceAccountName: sacc-with-anyuid
     containers:
       - name: cntr-main
         image: lucianoepc/mssql-cli:1.0-x64-alp3.20
         command: ["tail"]
         args: ["-f", "/dev/null"]
         imagePullPolicy: Always
         securityContext:
           allowPrivilegeEscalation: true
           runAsNonRoot: false
           runAsUser: 1000
           runAsGroup: 1000
   ```
   
   Comandos a usar:

   ```shell 
   bat podo-mssql-cli-1.yaml
   
   oc create -f podo-mssql-cli-1.yaml
   oc get pod -n ppoc-lpenac
   oc exec pod/podo-mssql-cli-1 -itn ppoc-lpenac -- bash
   
   oc delete pod/podo-mssql-cli-1 -n ppoc-lpenac
   ```


