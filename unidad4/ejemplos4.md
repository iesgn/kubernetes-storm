# Unidad 4: Despliegue de aplicaciones completas

## Antes de empezar

Instalamos `helm` desde [https://github.com/helm/helm/releases/](https://github.com/helm/helm/releases/)

Inicialmente aparece el repositorio
`kubernetes-charts.storage.googleapis.com`, que como se nos advierte,
está obsoleto:

```
helm repo list
WARNING: "kubernetes-charts.storage.googleapis.com" is deprecated for "stable" and will be deleted Nov. 13, 2020.
WARNING: You should switch to "https://charts.helm.sh/stable" via:
WARNING: helm repo add "stable" "https://charts.helm.sh/stable" --force-update
NAME  	URL                                             
stable	https://kubernetes-charts.storage.googleapis.com
```

Así, que actualizmos a `https://charts.helm.sh/stable`, tal como se
recomienda:

    helm repo add "stable" "https://charts.helm.sh/stable" --force-update
	
Y ahora el repositorio `stable` corresponde a charts.helm.sh/stable:

```
helm repo list
NAME  	URL                          
stable	https://charts.helm.sh/stable
```

## Ejemplo 1: Desplegar una aplicación con Helm

Buscamos una aplicación en el hub de helm, por ejemplo redis:

```
helm search hub redis
URL                                               	CHART VERSION	APP VERSION	DESCRIPTION                                       
https://hub.helm.sh/charts/drycc-canary/redis     	1.0.0        	           	A Redis database for use inside a Kubernetes cl...
https://hub.helm.sh/charts/groundhog2k/redis      	0.1.2        	6.0.9      	A Helm chart for Redis on Kubernetes              
https://hub.helm.sh/charts/wener/redis            	12.1.1       	6.0.9      	Open source, advanced key-value store. It is of...
https://hub.helm.sh/charts/hephy/redis            	2.4.2        	           	A Redis database for use inside a Kubernetes cl...
https://hub.helm.sh/charts/drycc/redis            	1.1.0        	           	A Redis database for use inside a Kubernetes cl...
https://hub.helm.sh/charts/bitnami/redis          	12.1.1       	6.0.9      	Open source, advanced key-value store. It is of...
https://hub.helm.sh/charts/choerodon/redis        	0.2.5        	0.2.5      	redis for Choerodon                               
https://hub.helm.sh/charts/inspur/redis-cluster   	0.0.2        	5.0.6      	Highly available Kubernetes implementation of R...
https://hub.helm.sh/charts/hkube/redis-ha         	3.6.1005     	5.0.5      	Highly available Kubernetes implementation of R...
https://hub.helm.sh/charts/dandydeveloper/redis-ha	4.10.5       	6.0.7      	Highly available Kubernetes implementation of R...
https://hub.helm.sh/charts/softonic/redis-sharded 	0.3.0        	6.0.6      	A Helm chart for sharded redis                    
https://hub.helm.sh/charts/bitnami/redis-cluster  	4.0.1        	6.0.9      	Open source, advanced key-value store. It is of...
https://hub.helm.sh/charts/kfirfer/redis-commander	0.0.2        	latest     	A Helm chart for redis-commander                  
https://hub.helm.sh/charts/prometheus-community...	3.7.0        	1.11.1     	Prometheus exporter for Redis metrics             
https://hub.helm.sh/charts/hmdmph/redis-pod-lab...	1.0.2        	1.0.0      	Labelling redis pods as master/slave periodical...
https://hub.helm.sh/charts/wyrihaximusnet/redis...	1.0.2        	v1.0.1     	Redis Database Assignment Operator                
https://hub.helm.sh/charts/enapter/keydb          	0.16.2       	6.0.16     	A Helm chart for KeyDB multimaster setup          
https://hub.helm.sh/charts/pozetron/keydb         	0.5.1        	v5.3.3     	A Helm chart for multimaster KeyDB optionally w...
```

Se puede obtener información sobre un chart concreto con:

```
helm show all bitnami/redis-cluster

annotations:
  category: Database
apiVersion: v2
appVersion: 6.0.9
dependencies:
- name: common
  repository: https://charts.bitnami.com/bitnami
  tags:
  - bitnami-common
  version: 1.x.x
description: Open source, advanced key-value store. It is often referred to as a data
  structure server since keys can contain strings, hashes, lists, sets and sorted
  sets.
home: https://github.com/bitnami/charts/tree/master/bitnami/redis-cluster
icon: https://bitnami.com/assets/stacks/redis/img/redis-stack-220x234.png
keywords:
- redis
- keyvalue
- database
maintainers:
- email: containers@bitnami.com
  name: Bitnami
name: redis-cluster
sources:
- https://github.com/bitnami/bitnami-docker-redis
- http://redis.io/
version: 4.0.1

---
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and imagePullSecrets
##
global:
  #   imageRegistry: myRegistryName
  #   imagePullSecrets:
  #     - myRegistryKeySecretName
  #   storageClass: myStorageClass
  redis: {}

## Bitnami Redis image version
## ref: https://hub.docker.com/r/bitnami/redis/tags/
##
image:
  registry: docker.io
  repository: bitnami/redis-cluster
  ## Bitnami Redis image tag
  ## ref: https://github.com/bitnami/bitnami-docker-redis#supported-tags-and-respective-dockerfile-links
  ##
  tag: 6.0.9-debian-10-r13
  ## Specify a imagePullPolicy
  ## Defaults to 'Always' if image tag is 'latest', else set to 'IfNotPresent'
  ## ref: http://kubernetes.io/docs/user-guide/images/#pre-pulling-images
```

Alternativamente podemos hacer la búsqueda en la web
[https://artifacthub.io/](https://artifacthub.io/):

[https://artifacthub.io/packages/search?page=1&ts_query_web=redis&kind=0](https://artifacthub.io/packages/search?page=1&ts_query_web=redis&kind=0)

En la que podemos seleccionar la opción que queremos y nos da detalles
sobre el chart y los parámetros que podemos utilizar en la
instalación.

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-redis bitnami/redis-cluster
```

Probamos que accedemos a la aplicación siguiendo las instrucciones y una vez que queramos eliminarla, lo hacemos con:

```
helm delete my-redis
```

## Ejemplo 2: Creación de un chart

Un chart tiene una estructura básica que podemos obtener con:

```
helm create chart
Creating chart1
cd chart1/ && ls
charts  Chart.yaml  templates  values.yaml
```

La estructura principal del Chart se define en `Chart.yaml`, los
objetos de k8s se definen en el directorio `templates` y los valores
que queremos asignar a dichos objetos en `values.yml`

## Ejemplo 3: Chart con despliegue nginx

Vamos a probar la utilización de un chart muy sencillo que permite
hacer un despliegue de nginx con el número de réplicas que se defina,
utilizar un servicio de tipo ClusterIP o NodePort a elección del
usuario y opcionalmente definir el nombre del servicio para accder a
través de ingress.

Editamos el fichero `mynginx/values.yaml` y ajustamos el valor del
host a la dirección de nuestro cluster de minikube. Luego creamos una
instancia del chart:

    helm install prueba mynginx/

Y comprobamos que se puede acceder al despliegue.
