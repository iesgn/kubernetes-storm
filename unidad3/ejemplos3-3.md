# Unidad 3 (III)

## Ejemplo 1: Variables de entorno

Podemos definir un Deployment que defina un contenedor configurado por
medio de variables de entorno.

Creamos el despliegue:

    kubectl create -f mariadb-deployment.yaml

O directamente ejecutando:

    kubectl run mariadb --image=mariadb --env MYSQL_ROOT_PASSWORD=my-password

Veamos el pod creado:

    kubectl get pods -l app=mariadb

Y probamos si podemos acceder, introduciendo la contraseña configurada:

    kubectl exec -it mariadb-deployment-fc75f956-f5zlt -- mysql -u root -p

## Ejemplo 2: ConfigMap

ConfigMap te permite definir un diccionario (clave,valor) para guardar
información que puedes utilizar para configurar una aplicación.

    kubectl create cm mariadb --from-literal=root_password=my-password \
                              --from-literal=mysql_usuario=usuario     \
                              --from-literal=mysql_password=password-user \
                              --from-literal=basededatos=test

    kubectl get cm
    kubectl describe cm mariadb

Creamos un deployment indicando los valores guardados en el ConfigMap:

    kubectl create -f mariadb-deployment-configmap.yaml
    kubectl exec -it mariadb-deploy-cm-57f7b9c7d7-ll6pv -- mysql -u usuario -p

## Ejemplo 3: Secrets

Los Secrets nos permiten guardar información sensible que será
codificada. Por ejemplo,nos permite guarda contraseñas, claves ssh, ...
Al crear un Secret los valores se pueden indicar desde un directorio,
un fichero o un literal.

    kubectl create secret generic mariadb --from-literal=password=root
    kubectl get secret
    kubectl describe secret mariadb

Creamos el despliegue y probamos el acceso:

    kubectl create -f mariadb-deployment-secret.yaml
    kubectl exec -it mariadb-deploy-secret-f946dddfd-kkmlb -- mysql -u root -p

## Ejemplo 4: Desplegando WordPress con MariaDB


mariadb

    kubectl create secret generic mariadb-secret \
                            --from-literal=dbuser=user_wordpress \
                            --from-literal=dbname=wordpress \
                            --from-literal=dbpassword=password1234 \
                            --from-literal=dbrootpassword=root1234 \
                            -o yaml --dry-run=client > mariadb-secret.yaml

    kubectl create -f mariadb-secret.yaml 

Creamos el servicio, que será de tipo ClusterIP:

    kubectl create -f mariadb-srv.yaml 

Y desplegamos la aplicación:
    
    kubectl create -f mariadb-deployment.yaml

wordpress

Lo primero creamos el servicio:

    kubectl create -f wordpress-srv.yaml 

Y realizamos el despliegue:

    kubectl create -f wordpress-deployment.yaml 

Por último creamos el recurso ingress que nos va a permitir el acceso
a la aplicación utilizando un nombre:

    kubectl create -f wordpress-ingress.yaml


## Ejemplo 5: StatefulSet

Vamos a crear los distintos objetos de la API:

    kubectl create -f service.yaml

### Creación ordenada de pods

En un terminal observamos la creación de pods y en otro terminal creamos los pods

    watch kubectl get pod
    kubectl create -f statefulset.yaml

### Comprobamos la identidad de red estable

Vemos los `hostname` y los nombres DNS asociados:

    for i in 0 1; do kubectl exec web-$i -- sh -c 'hostname'; done
    web-0
    web-1

    kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
    / # nslookup web-0.nginx
    ...
    Address 1: 172.17.0.4 web-0.nginx.default.svc.cluster.local
    / # nslookup web-1.nginx
    ...
    Address 1: 172.17.0.5 web-1.nginx.default.svc.cluster.local

### Eliminación de pods

 En un terminal observamos la creación de pods y en otro terminal eliminamos los pods

    watch kubectl get pod
    kubectl delete pod -l app=nginx

Comprobamos la identidad de red estable: Vemos los hostnames y los nombres DNS asociados (Las IP pueden cambiar):

    for i in 0 1; do kubectl exec web-$i -- sh -c 'hostname'; done

    kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
    / # nslookup web-0.nginx
    / # nslookup web-1.nginx

### Escribiendo en los volúmenes persistentes

Comprobamos que se han creado volúmenes para los pods:

    kubectl get pv,pvc

Escribimos en los documentroot y accedemos al servidor:

    for i in 0 1; do kubectl exec "web-$i" -- sh -c 'echo "$(hostname)" > /usr/share/nginx/html/index.html'; done
    for i in 0 1; do kubectl exec -i -t "web-$i" -- sh -c 'curl http://localhost/'; done
    web-0
    web-1

Volvemos a eliminar los pods, y comprobamos que la información es persistente al estar guardadas en los volúmenes:

    kubectl delete pod -l app=nginx
    for i in 0 1; do kubectl exec -i -t "web-$i" -- sh -c 'curl http://localhost/'; done

### Escalar statefulset

Escalamos a más o menos pods:

    kubectl scale sts web --replicas=5

Comprobamos los pods y los volúmenes:

    kubectl get pod,pv,pvc

Si reducimos el número de pods los volúmenes no se eliminan.

Para terminar eliminamos el statefulset y el service:

    kubectl delete -f .

Para borrar los volúmenes:

    kubectl delete --all pv,pvc

## Ejemplo 6: DaemonSet

Para ver este ejemplo es necesario tener un clúster con varios nodos
(workers), en este caso se ha utilizado un cluster con k3s:

    kubectl get nodes
    NAME    STATUS   ROLES    AGE   VERSION
    k3s-1   Ready    <none>   17d   v1.14.1-k3s.4
    k3s-2   Ready    <none>   17d   v1.14.1-k3s.4
    k3s-3   Ready    <none>   17d   v1.14.1-k3s.4

    kubectl create -f ds.yaml

    kubectl get pods -o wide
    NAME            READY   STATUS    RESTARTS   AGE   IP       NODE
    logging-5v4dh   1/1     Running   0          9s    10.42.2.26   k3s-2
    logging-gqfbb   1/1     Running   0          9s    10.42.1.55   k3s-3
    logging-wbdjj   1/1     Running   0          9s    10.42.0.25   k3s-1

Podemos seleccionar los nodos en los que queremos que se ejecuten los
pod por medio de un selector.

    kubectl create -f ds2.yaml

    kubectl get pods -o wide
    No resources found.

    kubectl get ds
    NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR      AGE
    logging   0         0         0       0            0           app=logging-node   17s

    kubectl label node k3s-3 app=logging-node --overwrite

    kubectl get ds
    NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR      AGE
    logging   1         1         0       1            0           app=logging-node   41s

    kubectl get pods -o wide
    NAME            READY   STATUS    RESTARTS   AGE   IP           NODE    NOMINATED NODE   READINESS GATES
    logging-556r9   1/1     Running   0          7s    10.42.1.56   k3s-3   <none>           <none>

## Ejemplo 7: Horizontal Pod AutoScaler

Veamos un ejemplo: creamos un despliegue de una aplicación php y
modificamos lo que va a reservar del CPU el pod (0,2 cores de CPU):

    kubectl create deploy php-apache --image=k8s.gcr.io/hpa-example
    kubectl expose deploy php-apache --port=80 --type=NodePort
    kubectl set resources deploy php-apache --requests=cpu=200m

Creamos el recurso hpa, indicando el mínimo y máximos de pods que va a
terner el despliegue, y el límite de uso de CPU que va a tener en
cuenta para crear nuevos pods:

    kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=5

Vamos a hacer una prueba de estrés a nuestra aplicación y observamos
cómo se comporta:

    service=$(minikube service php-apache --url)
    while true; do wget -q -O- "$service"; done

    watch kubectl get pod
    kubectl get hpa -w

