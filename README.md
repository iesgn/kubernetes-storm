# Kubernetes for Storm

## Unidad 1: Despliegue de aplicaciones en contenedores. Introducción a k8s

* [Presentación unidad](unidad1/presentacion_unidad1.pdf)
* Despliegue de aplicaciones en contenedores.
  * Introducción a los contenedores
  * Arquitectura de microservicios
  * Tecnologías subyacentes y diferencias entre ellas: docker, cri-o, LXC, ...
  * Ciclo de vida en el despliegue de aplicaciones con docker

## Unidad 2: Introducción a kubernetes
* [Presentación unidad](unidad2/presentacion_unidad2.pdf)
  * Características, historia, estado actual del proyecto kubernetes (k8s)
  * Arquitectura básica de k8s
  * Alternativas para instalación simple de k8s: minikube, kubeadm, k3s

## Unidad 3: Despliegue de aplicaciones con k8s

* [Presentación unidad](unidad3/presentacion_unidad3-1.pdf)
  * Pods
  * ReplicaSet: Tolerancia y escalabilidad
  * Deployment: Actualizaciones y despliegues automáticos
  * Estrategias de despliegue en k8s: blue/green, canary, etc.
* [Ejemplos](unidad3/README.md)
* [Presentación unidad](unidad2/presentacion_unidad3-2.pdf)
  * Services
  * DNS
  * Ingress
  * Ejemplos de uso y despliegues
* [Ejemplos](unidad3/README.md)
* [Presentación unidad](unidad2/presentacion_unidad3-3.pdf)
* Configuración de aplicaciones: Variables de entorno, ConfigMaps, Secrets, ...
* Ejemplo de despliegue parametrizado
* StatefulSet
* DaemonSet
* AutoScale
* Jobs, cronjobs
* [Ejemplos](unidad3/README.md)

## Unidad 4: Despliegue de aplicaciones con Helm

## Unidad 5: Almacenamiento persistente en k8s
 
* Almacenamiento
* PersistantVolume
* PersistentVolumeClaim

## Unidad 6: Despliegue de aplicaciones de k8s en entornos en producción
