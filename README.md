# OpenShift con Developer Sandbox Red Hat - Curso CKA

## Descripción
Este repositorio contiene un laboratorio diseñado para aprender a trabajar con OpenShift utilizando la **Developer Sandbox gratuita de Red Hat**. Exploraremos conceptos avanzados como despliegues de aplicaciones, resolución de problemas en Pods, y estrategias de despliegue como **Blue-Green** y **Canary**.

## Prerrequisitos
- **Registro en la Sandbox de Red Hat OpenShift**:
   - Regístrate en la [Sandbox de OpenShift](https://www.redhat.com/en/openshift-sandbox).
   - Accede a tu cuenta personal y asegúrate de tener un entorno funcional.
- **Herramientas necesarias**:
   - CLI de OpenShift (`oc`), ya preinstalada en la Sandbox.
   - Navegador web para interactuar con la consola gráfica de OpenShift.
- **Conocimientos previos**:
   - Kubernetes (pods, servicios, manifiestos YAML).
   - Docker y estrategias de contenedores.

## Objetivos
- Aprender a utilizar la Sandbox de OpenShift proporcionada por Red Hat.
- Realizar configuraciones avanzadas, como rutas, servicios e ingress.
- Implementar estrategias de despliegue Blue-Green y Canary.
- Utilizar herramientas de resolución de problemas en Pods.
- Desplegar aplicaciones desde repositorios Git y manifiestos YAML.

## Paso 1: Acceso a la Sandbox
- Dirígete a la [consola de OpenShift](https://console-openshift-console.apps.sandbox-m3.1530.p1.openshiftapps.com/).
- Usa las credenciales de tu cuenta Red Hat para iniciar sesión.
- Una vez dentro, utiliza la consola gráfica o el terminal en la esquina superior derecha para ejecutar los comandos.

## Paso 2: Crear un nuevo proyecto (namespace)
- Configura tu entorno de trabajo con el siguiente comando:
  ```bash
  oc project <nombre-del-proyecto>
  ```
  > Este comando establece el namespace o proyecto en el cual trabajarás.

## Lab 1: Despliegue de Aplicaciones
Desplegar una aplicación desde un repositorio Git local:

1. Clona el repositorio de ejemplo:
  ```bash
  git clone https://github.com/openshift/source-to-image.git
  cd examples/nginx-centos7
  ```
  > Aquí clonamos el repositorio localmente y navegamos hasta la carpeta con la aplicación.

2. Crear un nuevo BuildConfig basado en Docker:
  ```bash
  oc new-build --name s2i-nginx --binary --strategy docker
  ```
  > `oc new-build` crea un objeto `BuildConfig` para construir una imagen Docker a partir de la fuente local especificada.

3. Inicia el proceso de construcción:
  ```bash
  oc start-build s2i-nginx --from-dir=.
  ```
  > `oc start-build` inicia la construcción con el contenido local. Este comando tomará los archivos presentes en el directorio actual para construir la imagen.

## Lab 7: Autoescalado Horizontal (Horizontal Pod Autoscaler - HPA)

1. Crear un despliegue para una aplicación simple:
  ```bash
  oc new-app --name=my-httpd --docker-image=httpd:latest
  ```
  > `oc new-app` crea un despliegue de Apache HTTP Server a partir de una imagen Docker pública.

2. Crear un Autoescalador Horizontal para la aplicación:
  ```bash
  oc autoscale deployment/my-httpd --min=1 --max=10 --cpu-percent=50
  ```
  > `oc autoscale` configura el escalado automático del despliegue `my-httpd` para mantener entre 1 y 10 réplicas, dependiendo del uso de CPU.

3. Generar carga para probar el escalado:
  ```bash
  for i in {1..100}; do curl -s http://$(oc get route my-httpd -o jsonpath='{.spec.host}'); done
  ```
  > Este comando genera múltiples solicitudes HTTP para aumentar la carga de la aplicación y activar el escalado.

4. Verificación:
  ```bash
  oc get hpa
  ```
  > Verifica el estado del Autoescalador Horizontal y las métricas de uso de la CPU.

## Lab 8: Implementación de ConfigMap y Secret

1. Crear un ConfigMap para una aplicación:
  ```bash
  oc create configmap my-config --from-literal=APP_COLOR=blue
  ```
  > `oc create configmap` crea un ConfigMap con una clave-valor llamada `APP_COLOR`.

2. Crear un Secret para la aplicación:
  ```bash
  oc create secret generic my-secret --from-literal=DB_PASSWORD=mysecretpassword
  ```
  > `oc create secret` crea un Secret que almacena una contraseña de base de datos.

3. Actualizar el despliegue para usar el ConfigMap y el Secret:
  ```bash
  oc set env deployment/my-httpd --from=configmap/my-config
  oc set env deployment/my-httpd --from=secret/my-secret
  ```
  > `oc set env` agrega variables de entorno al despliegue desde el ConfigMap y el Secret.

4. Verificación:
  ```bash
  oc describe deployment/my-httpd
  ```
  > Verifica que el despliegue `my-httpd` tenga las variables de entorno configuradas desde el ConfigMap y el Secret.