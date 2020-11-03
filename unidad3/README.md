# Unidad 3: Despliegue de aplicaciones con k8s (I)

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

Comprobamos el historial de actualizaciones y desplegamos una nueva versión:

    kubectl rollout history deployment/mediawiki
    kubectl set image deployment/mediawiki mediawiki=mediawiki:1.27 --all --record
    kubectl get all

Ahora vamos a desplegar una versión que da un error (versión no existe). ¿Podremos volver al despliegue anterior?

    kubectl rollout history deployment/mediawiki
    kubectl set image deployment/mediawiki mediawiki=mediawiki:2 --all --record
    kubectl rollout undo deployment/mediawiki
    kubectl get all

## Ejemplo 6: guestbook (parte 1)

    kubectl apply -f frontend-deployment.yaml
    kubectl apply -f redis-master-deployment.yaml
    kubectl apply -f redis-slave-deployment.yaml

    kubectl port-forward deployment/guestbook 3000:3000

# Unidad 3: Acceso a las aplicaciones desplegadas en k8s

## Ejemplo 1: Services

**Service ClusterIP**

    kubectl apply -f deployment.yaml

    kubectl apply -f service_ci.yaml

También podríamos haber creado el servicio sin usar el fichero yaml, de la siguiente manera:

    kubectl expose deployment/nginx --port=80 --type=ClusterIP

Podemos ver el servicio que hemos creado:

    kubectl get svc

Puede ser bueno acceder desde exterior, por ejemplo en la fase de desarrollo de una aplicación para probarla:

    kubectl proxy

Y accedemos a la URL:

    http://localhost:8001/api/v1/namespaces/<NAMESPACE>/services/<SERVICE NAME>:<PORT NAME>/proxy/

**Service NodePort**

    kubectl apply -f service_np.yaml

También podríamos haber creado el servicio sin usar el fichero yaml, de la siguiente manera:

    kubectl expose deployment/nginx --port=80 --type=NodePort

Podemos ver el servicio que hemos creado:

    kubectl get svc

Desde el exterior accedemos a:

    http://<IP_MASTER>:<PUERTO_ASIGNADO>

## Ejemplo 2: guestbook (parte 2)

    kubectl apply -f frontend-deployment.yaml
    kubectl apply -f redis-master-deployment.yaml
    kubectl apply -f redis-slave-deployment.yaml
    kubectl apply -f frontend-srv.yam
    kubectl apply -f redis-master-srv.yaml
    kubectl apply -f redis-slave-srv.yaml


## Ejemplo 3: DNS

    kubectl create -f busybox.yaml
    kubectl exec -it busybox -- nslookup nginx
    kubectl exec -it busybox -- wget http://nginx

Comprobando el servidor DNS:

    kubectl get pods --namespace=kube-system -o wide
    kubectl get services --namespace=kube-system
    kubectl exec -it busybox -- cat /etc/resolv.conf

## Ejemplo 4: Balanceo de carga

Hemos creado una imagen docker que nos permite crear un contenedor con una aplicación PHP que muestra el nombre del servidor donde se ejecuta, el fichero index.php:

    <?php echo "Servidor:"; echo gethostname();echo "\n"; ?>

Si tenemos varios pod de esta aplicación, el objeto Service balancea la carga entre ellos:

    kubectl create deployment pagweb --image=josedom24/infophp:v1
    kubectl expose deploy pagweb --port=80 --type=NodePort
    kubectl scale deploy pagweb --replicas=3

Al acceder hay que indicar el puerto asignado al servicio:

    for i in `seq 1 100`; do curl http://192.168.99.100:32376; done
    Servidor:pagweb-84f6d54fb7-56zj6
    Servidor:pagweb-84f6d54fb7-mdvfn
    Servidor:pagweb-84f6d54fb7-bhz4p


## Ejemplo 5: Servicio para acceder a servidor remoto

Creamos el servicio:

    kubectl apply -f service.yaml

Creamos el endpoint:

    kubectl apply -f endpoint.yaml

Comprobamos que el servicio está apuntando al endpoint:

    kubectl describe service mariadb

Creamos un pod con un cliente de mariadb y accedemos usando el nombre del servicio:

    kubectl apply -f mariadb-deployment.yaml 

    kubectl exec -it pod/mariadb -- mysql -u prueba -p -h mariadb.default.svc.cluster.local

## Ejemplo 6: Despliegue canary

Creamos el servicio y el despliegue de la primera versión (5 réplicas):

    kubectl apply -f service.yaml
    kubectl apply -f deploy1.yaml

En un terminal, vemos los pods:

    watch kubectl get pod

En otro terminal, accedemos a la aplicación:

    service=$(minikube service my-app --url)
    while sleep 0.1; do curl "$service"; done

Desplegamos una réplica de la versión 2, para ver si funciona bien:

    kubectl apply -f deploy2.yaml

Una vez que comprobamos que funciona bien, podemos escalar y eliminar la versión 1:

    kubectl scale --replicas=5 deploy my-app-v2
    kubectl delete deploy my-app-v1

## Ejemplo 7: ingress

    minikube addons enable ingress
    kubectl get pod -n kube-system
    
    kubectl apply -f nginx-ingress.yaml 
    kubectl get ingress

## Ejemplo 8: LetsChat

    kubectl apply -f letschat-deployment.yaml
    kubectl apply -f mongo-deployment.yaml
    
    kubectl apply -f letschat-srv.yaml
    kubectl apply -f mongo-srv.yaml
    
    kubectl apply -f ingress.yaml
