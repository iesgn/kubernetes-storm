# Unidad 3 (II): Acceso a las aplicaciones desplegadas en k8s

## Ejemplo 1: Services

Vamos a crear un despliegue básico compuesto de dos pods nginx y vamos
a definir para el mismo un servicio de tipo ClusterIP y luego otro de
tipo NodePort.

    kubectl apply -f deployment.yaml
	
**Service ClusterIP**

Creamos el servicio de tipo ClusterIP:

    kubectl apply -f service_clusterip.yaml

También podríamos haber creado el servicio sin usar el fichero yaml,
mediante comandos imperativos, de la siguiente manera:

    kubectl expose deployment/nginx --port=80 --type=ClusterIP

Podemos ver el servicio que hemos creado:

    kubectl get svc

Puede ser bueno acceder desde exterior, por ejemplo en la fase de desarrollo de una aplicación para probarla:

    kubectl proxy

Y accedemos a la URL:

    http://localhost:8001/api/v1/namespaces/<NAMESPACE>/services/<SERVICE NAME>:<PORT NAME>/proxy/

**Service NodePort**

Eliminamos el servicio anterior y creamos un servicio de tipo
nodeport:

    kubectl delete service nginx
    kubectl apply -f service_np.yaml

También podríamos haber creado el servicio sin usar el fichero yaml, de la siguiente manera:

    kubectl expose deployment/nginx --port=80 --type=NodePort

Podemos ver el servicio que hemos creado:

    kubectl get svc

O de forma detallada con:

    kubectl describe service nginx
	
Podemos ver que además de una IP de cluster (rango 10.102.*.* en el
caso de minikube), se ha creado un puerto superior al 30000/tcp en uno
de los nodos controladores y podemos acceder al servicio desde el
exterior a través del mismo:

    http://<IP_MASTER>:<PUERTO_ASIGNADO>

## Ejemplo 2: guestbook (parte 2)

Creamos servicios para los despliegues de "guestbook":

    kubectl apply -f frontend-deployment.yaml
    kubectl apply -f redis-master-deployment.yaml
    kubectl apply -f redis-slave-deployment.yaml
    kubectl apply -f frontend-srv.yam
    kubectl apply -f redis-master-srv.yaml
    kubectl apply -f redis-slave-srv.yaml

También podríamos hacer:

    kubectl apply -f ejemplo2/

Los servicios para redis se definen como clusterIP y el accesible
desde el exterior como nodeport.

También podemos eliminarlo todo con:

    kubectl delete -f ejemplo2/

Aunque lo dejamos funcionando para probar el siguiente ejemplo.

## Ejemplo 3: DNS

    kubectl create -f busybox.yaml
    kubectl exec -it busybox -- nslookup nginx
    kubectl exec -it busybox -- wget http://nginx

Comprobando el servidor DNS:

    kubectl get pods --namespace=kube-system -o wide
    kubectl get services --namespace=kube-system
    kubectl exec -it busybox -- cat /etc/resolv.conf

## Ejemplo 4: Balanceo de carga

Hemos creado una imagen docker que nos permite crear un contenedor con
una aplicación PHP que muestra el nombre del servidor donde se
ejecuta, el fichero index.php:

    <?php echo "Servidor:"; echo gethostname();echo "\n"; ?>

Si tenemos varios pod de esta aplicación, el objeto Service balancea
la carga entre ellos:

    kubectl create deployment pagweb --image=josedom24/infophp:v1
    kubectl expose deploy pagweb --port=80 --type=NodePort
    kubectl scale deploy pagweb --replicas=3

Al acceder hay que indicar el puerto asignado al servicio:

    for i in `seq 1 100`; do curl http://<MASTER-IP>:<SVC-PORT>; done
    Servidor:pagweb-84f6d54fb7-56zj6
    Servidor:pagweb-84f6d54fb7-mdvfn
    Servidor:pagweb-84f6d54fb7-bhz4p

## Ejemplo 5: Servicio para acceder a servidor remoto

Supongamos que tenemos una aplicación desplegada en kubernetes que
tiene que acceder a un servicio externo, por ejemplo, una base de
datos. En ese caso podemos definir un servicio que puedan usar todos
los pods del espacio de nombres y asociado a él un "endpoint" que
apunte al servicio externo.

Creamos el servicio:

    kubectl apply -f service.yaml

Creamos el endpoint:

    kubectl apply -f endpoint.yaml

Comprobamos que el servicio está apuntando al endpoint:

    kubectl describe service mariadb
	kubectl describe endpoints mariadb

## Ejemplo 6: Despliegue canary

Vamos a realizar de forma simple un despliegue de tipo canario, pero
ya que esta estrategia no está implementada directamente en k8s,
usamos dos despliegues diferentes, en lugar de dos replicasets.

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

Una vez que comprobamos que funciona bien, podemos escalar y eliminar
la versión 1:

    kubectl scale --replicas=5 deploy my-app-v2
    kubectl delete deploy my-app-v1

## Ejemplo 7: ingress

En primer lugar vamos a habilitar el complemento ingress en minikube,
esto lo que hace es crear una serie de objetos k8s en el espacio de
nombres `kube-system`:

    minikube addons enable ingress
    kubectl get pod -n kube-system
    
Para probar que funciona correctamente ingress en nuestro cluster de
k8s, vamos a hacer un despliegue de pruebas y vamos a acceder a él
desde el exterior:

Ejecutamos un despligue de nginx:

    kubectl apply -f deployment.yaml
	
Asociamos un servicio de tipo NodePort:

    kubectl apply -f service_np.yaml
	
Averiguamos la IP de nuestro cluster de minikube:

	minikube ip
	
O también podríamos obtener con minikube la URL del servicio `nginx`:

	minikube service nginx --url
	
Con estos datos editamos el YAML del ingress y hacemos uso de
`nip.io`:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: nginx.<MASTER IP>.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```
	
    kubectl apply -f ingress.yaml
	
Comprobamos que se ha definido el objeto ingress y abrimos en el
navegador usando el "nombre" de `nip.io`:

    kubectl get ingress

## Ejemplo 8: LetsChat

Esto es un ejemplo de una aplicación "completa" de dos capas, que
tiene define dos despliegues diferentes, cada uno con su servicio
respectivo y es accesible desde el exterior.

    kubectl apply -f letschat-deployment.yaml
    kubectl apply -f mongo-deployment.yaml
    
    kubectl apply -f letschat-srv.yaml
    kubectl apply -f mongo-srv.yaml
    
    kubectl apply -f ingress.yaml

También sería posible crear la aplicación completa con:

    kubectl apply -f ejemplo8/

Para comprobar el funcionamiento hay que editar el fichero de
resolución estática del equipo y que `www.letschat.com` apunte a la
dirección IP de minikube (o del cluster de k8s en su caso).
