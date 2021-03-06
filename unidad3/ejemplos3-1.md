# Unidad 3: Despliegue de aplicaciones con k8s (I)

## Antes de empezar

Iniciamos minikube:

    minikube start

Comprobamos los nodos que hay en nuestro cluster y pedimos detalles del nodo:

    kubctl get nodes
    kubectl describe node minikube

Nosotros estamos trabajando en el espacio de nombres `default`:

    kubectl get ns

Podemos ver que con `kubectl` podemos acceder a la API de nuestro cluster de kubernetes, por que en el fichero `~/.kube/config` tengo definido las credenciales:

    cat ~/.kube/config

Finalmente podemos ver los distintos recursos que nos ofrece la API de kubernetes ejecutando:

    kubectl proxy

Y podemos acceder a un recurso de la API, por ejemplo a la información de los nodos del cluster:

    http://localhost:8001/api/v1/nodes

## Ejemplo 1: Pod

Para crear el pod desde el fichero yaml:

    kubectl create -f pod.yaml

Y podemos ver que el pod se ha creado:

    kubectl get pods

Si queremos saber en qué nodo del cluster se está ejecutando:

    kubectl get pod -o wide

Para obtener información más detallada del pod:

    kubectl describe pod nginx

Para eliminar el pod:

    kubectl delete pod nginx

Podemos modificar las características de cualquier recurso de kubernetes una vez creado, por ejemplo podemos modificar la definición del pod de la siguiente manera:

    kubectl edit pod nginx
    KUBE_EDITOR="nano" kubectl edit pod nginx

Para obtener los logs del pod:

    kubectl logs nginx

Si quiero conectarme al contenedor:

    kubectl exec -it nginx -- /bin/bash

Podemos acceder a la aplicación, redirigiendo un puerto de localhost al puerto de la aplicación:

    kubectl port-forward nginx 8080:80

Y accedemos al servidor web en la url `http://localhost:8080`.

Para obtener las labels de los pods que hemos creado:

    kubectl get pods --show-labels

Los Labels lo hemos definido en la sección `metadata` del fichero yaml, pero también podemos añadirlos a los pods ya creados:

    kubectl label pods nginx service=web --overwrite=true

Los Labels me van a permitir seleccionar un recurso determinado, por ejemplo para visualizar los pods que tienen un label con un determinado valor:

    kubectl get pods -l service=web

También podemos visualizar los valores de los labels como una nueva columna:

    kubectl get pods -Lservice

## Ejemplo 2: Pod multicontenedor

    kubectl create -f  pod-multi-container.yaml

    kubectl describe pod mc1

    kubectl exec mc1 -c 1st -- /bin/cat /usr/share/nginx/html/index.html
  
    kubectl exec mc1 -c 2nd -- /bin/cat /html/index.html

    kubectl port-forward mc1 8080:80

## Ejemplo 3: ReplicaSet

Creamos el ReplicaSet:
    
    kubectl create -f nginx-rs.yaml

    kubectl get rs
    kubectl get pods

¿Qué pasaría si borro uno de los pods que se han creado? Inmediatamente se creará uno nuevo para que siempre estén ejecutándose los pods deseados, en este caso 2:

    kubectl delete pod nginx-5b2rn
    kubectl get pods

Para escalar el número de pods:

    kubectl scale rs nginx --replicas=5
    kubectl get pods --watch

Como anteriormente vimos podemos modificar las características de un ReplicaSet con la siguiente instrucción:

    kubectl edit rs nginx

Por último si borramos un ReplicaSet se borraran todos los pods asociados:

    kubectl delete rs nginx

## Ejemplo 4: Deployment

Cuando creamos un Deployment, se crea el ReplicaSet asociado y todos los pods que hayamos indicado.

    kubectl apply -f nginx-deployment.yaml
    kubectl get deploy,rs,pod

Como ocurría con los replicaSets los Deployment también se pueden escalar, aumentando o disminuyendo el número de pods asociados:

    kubectl scale deployment nginx --replicas=4

Otras operaciones:

    kubectl port-forward deploy/nginx 8080:80
    kubectl logs deploy/nginx

Si eliminamos el Deployment se eliminarán el ReplicaSet asociado y los pods que se estaban gestionando.

    kubectl delete deployment nginx

Para modificar un Deployment (o cualquier objeto k8s):

* Modificando el parámetro directamente. `kubectl set`
* Modificando el fichero yaml y aplicando el cambio. `kubectl apply -f`
* Modificando la definición del objeto: `kubectl edit deployment ...`

Por ejemplo para hacer una actualización de la aplicación:

* `kubectl set image deployment nginx nginx=nginx:1.16 --all`
* Modifico el fichero deployment.yaml y ejecuto:
    * `kubectl apply -f deployment.yaml`
* `kubectl edit deployment nginx`

Actualización y rollout de la aplicación:

    kubectl set image deployment nginx nginx=nginx:1.16 --all

Comprobamos que se ha creado un nuevo ReplicaSet, y unos nuevos pods con la nueva versión de la imagen.

    kubectl get rs
    kubectl get pods

La opción `--all` fuerza a actualizar todos los pods aunque no estén inicializados.

Si queremos volver a la versión anterior de nuestro despliegue, tenemos que ejecutar:

    kubectl rollout undo deployment nginx

Y comprobamos como se activa el antiguo ReplicaSet y se crean nuevos pods con la versión anterior de nuestra aplicación:

    kubectl get rs

Se puede volver a una revisión determinada poniendo el número:

    kubectl rollout history deployment nginx            
    kubectl rollout undo deployment nginx --to-revision=2
    
## Ejemplo 5: mediaWiki

Vamos a desplegar la aplicación mediawiki:

    kubectl apply -f mediawiki-deploy.yaml --record
    kubectl get all

Comprobamos que se puede acceder a la aplicación y veremos la versión
desplegada:

    kubectl port-forward deployment/mediawiki 8080:80

Esta despliegue de mediawiki no está conectado a una base de datos
común, por lo que si optamos por usar una base de datos local
(sqlite), no habrá consistencia en los datos entre los pods y además
no permanecerán si uno de los pods se elimina.

Con la opción `--record` guardamos el comando como una anotación del
despliegue (simplemente a título informativo, no hace nada
más). Comprobamos el historial de actualizaciones y desplegamos una
nueva versión:

    kubectl rollout history deployment/mediawiki
    kubectl set image deployment/mediawiki mediawiki=mediawiki:1.31.10 --all --record
    kubectl get all

Ahora vamos a desplegar una versión que da un error (versión no
existe). ¿Podremos volver al despliegue anterior?

    kubectl rollout history deployment/mediawiki
    kubectl set image deployment/mediawiki mediawiki=mediawiki:2 --all  --record

Dependiendo de la estrategia de despliegue, esto puede provocar que la
aplicación se quede en la versión anterior o que no haya ningún pod
válido desplegado. En cualquier caso, se puede volver a la versión
anterior del despliegue mediante `rollout`:

    kubectl rollout undo deployment mediawiki
    kubectl get all

Este ejercicio nos sirve para ver de manera rápida y fácil la gestión
que hace kubernetes de las diferentes versiones y como podemos volver
rápidamente a un despliegue de una versión anterior, pero todavía no
podemos realizar despliegues reales porque no sabemos cómo conectar
diferentes despliegues (servicios o microservicios) entre sí.

## Ejemplo 6: guestbook (parte 1)

Procedemos ahora a la ejecución de una aplicación "completa" de
ejemplo en la que varios despliegues se relacionan entre sí:

    kubectl apply -f frontend-deployment.yaml
    kubectl apply -f redis-master-deployment.yaml
    kubectl apply -f redis-slave-deployment.yaml

En este despliegue se han creado tres pods de la aplicación
"guestbook" que almacena sus datos en un servidor redis principal y
que está replicado en tres servidores redis secundarios. Podemos usar
y acceder a la aplicación en modo pruebas mediante "port-forward":

    kubectl port-forward deployment/guestbook 3000:3000

Aunque esta aplicación ya tiene separados los servicios o
microservicios en diferentes despliegues, no pueden conectarse entre
sí todavía porque debemos usar algún mecanismo para que se puedan
comunicar.
