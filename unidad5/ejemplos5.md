# Unidad 5: Almacenamiento

## Ejercicio 1

Comprobamos que `minikube` proporciona un `storageClass` por defecto
que permite el aprovisionamiento dinámico de volúmenes:

```
minikube addons list
...

kubectl get storageclasses.storage.k8s.io 
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  2d6h

kubectl describe storageclasses.storage.k8s.io standard 
Name:            standard
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"labels":{"addonmanager.kubernetes.io/mode":"EnsureExists"},"name":"standard"},"provisioner":"k8s.io/minikube-hostpath"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           k8s.io/minikube-hostpath
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```

Creamos un `PersistentVolumeClaim` de 1 GiB:

```
kubectl apply -f pvc1.yaml
```

Y vemos si se ha creado:

```
kubectl get pvc pvc1 
NAME   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc1   Bound    pvc-f2c0699b-f49d-45df-be0e-fd8aeace699f   1Gi        RWO            standard       1m9s
```

Levanamos un pod "busybox" con el volumen montado en "/mnt" y
escribimos un fichero index.html en ese directorio:

```
kubectl apply -f busybox.yaml
kubectl exec -it busybox -- /bin/sh

/ # echo "<!DOCTYPE html><html><head><title>Hola!</title></head><body><h1>Hola!</h1></body></html>" > /mnt/index.html
```

Salimos y borramos el contenedor (no se borrará el volumen).

Ahora creamos el pod nginx, que usará el volumen montado en el
directorio raíz del sitio web:

```
kubectl apply -f webserver
kubectl port-forward webserver 8080:80
```

## Ejemplo 2: PV y PVC (estático)

Creamos el PersistantVolumen:

    kubectl create -f pv.yaml

    kubectl get pv
    NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
    pv1       5Gi        RWX               Recycle             Available         manual                    5s

Creamos el PersistantVolumenClaim:

    kubectl create -f pvc1.yaml

    kubectl get pv          
    NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS   REASON   AGE
    pv1       5Gi         RWX               Recycle             Bound       default/pvc1  manual                    66s
    
    kubectl get pvc
    NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    pvc1   Bound       pv1   5Gi          RWX            manual           31s

Escribimos un fichero index.html en el directorio correspondiente al volumen:

    minikube ssh                           
    $ sudo sh -c "echo 'Hello from Kubernetes storage' > /data/pv1/index.html"

Creamos un pod con el volumen

    kubectl create -f pod.yaml
    kubectl get pod
    kubectl describe pod task-pv-pod

Accedemos el pod, instalamos curl y probamos a acceder al servidor web

    kubectl exec -it task-pv-pod -- /bin/bash
    root@task-pv-pod:/# curl localhost
    Hello from Kubernetes storage

## Ejemplo 3: WordPress con almacenamiento persistente en NFS

Instalamos inicialmente un servidor NFS en el sistema en el que se
ejecuta minikube, creamos un directorio a exportar que lo hacemos
accesible a la red en la que se está ejecutando minikube:

```
exportfs 
/MVs/NFS      	192.168.39.250/32
```

Levantamos todo el  escenario y en este caso tendremos la posibilidad
de escalar como queramos el despliegue de wordpress, ya que todos los
nodos pueden leer y escribir sobre el mismo volumen la configuración
sin posibilidad de que se pierda.

    kubectl apply -f ejemplo3/
	
Tendremos que esperar un rato hasta que se copie completamente
wordpress en el volumen, lo que podemos controlar con:

    kubectl log wordpress-...
	
Una vez descargado, estará disponible el servidor web y podremos
acceder a la URL del servicio proporcionada por ingress.

Ahora podemos subir o bajar el número de nodos y la información
permanece.

    kubectl scale deployment wordpress-deployment --replicas=0
	
    kubectl scale deployment wordpress-deployment --replicas=4
	
No ocurre lo mismo con la base de datos, que con este escenario se
ejecutaría en un solo pod.
