# Kubernetes for Storm

## Unidad 1: Despliegue de aplicaciones en contenedores. Introducción a k8s

* [Presentación unidad](unidad1/presentacion_unidad1.pdf)
* Despliegue de aplicaciones en contenedores.
  * Introducción a los contenedores
  * Arquitectura de microservicios
  * Tecnologías subyacentes y diferencias entre ellas: docker, cri-o, LXC, ...
  * Ciclo de vida en el despliegue de aplicaciones con docker

* Introducción a kubernetes
  * Características, historia, estado actual del proyecto kubernetes (k8s)
  * Arquitectura básica de k8s
  * Alternativas para instalación simple de k8s: minikube, kubeadm, k3s

## Unidad 2: Despliegue de aplicaciones con k8s (I)

* [Presentación unidad](unidad2/presentacion_unidad2.pdf)
* Pods
* ReplicaSet: Tolerancia y escalabilidad
* Deployment: Actualizaciones y despliegues automáticos
* Estrategias de despliegue en k8s: blue/green, canary, etc.
* [Ejemplos](unidad2/README.md)

## Unidad 3: Acceso a las aplicaciones desplegadas en k8s

* [Presentación unidad](unidad2/presentacion_unidad3.pdf)
* Services
* DNS
* Ingress
* Ejemplos de uso y despliegues
* [Ejemplos](unidad3/README.md)

## Unidad 4: Despliegue de aplicaciones con k8s (II)

* Configuración de aplicaciones: Variables de entorno, ConfigMaps, Secrets, ...
* Ejemplo de despliegue parametrizado
* StatefulSet
* DaemonSet
* AutoScale
* Jobs, cronjobs
  
 ## Unidad 5: Almacenamiento persistente en k8s
 
* Almacenamiento
* PersistantVolumen
* PersistentVolumenClaim
* 


* Despliegue de aplicaciones con Helm



* Despliegue de aplicaciones de k8s en entornos en producción
