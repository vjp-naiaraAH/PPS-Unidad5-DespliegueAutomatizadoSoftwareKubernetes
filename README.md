# Actividad - Despliegue automatizado de software: Kubernetes - naiara
## ELIMINANDO CLUSTER DE MINIKUBE
Elimino el cluster que he estado usando durante toda la duración del vídeo.
```bash
minikube delete -p multinodo
```
![borrado](/images/1.png)

---

## PREPARACIÓN DEL LABORATORIO
Descomprimo ***store-app-postgree.zip*** que está para descargar en el [siguiente enlace](https://github.com/jmmedinac03vjp/PuestaProduccionSegura/blob/main/Unidad5-SistemasDesplegadoAutomatizadoSoftware/Actividad-DespliegueAutomatizadoSoftwareKubernetes/files/store-app-postgree.zip)
```bash
# descargar store-app.zip y colocarla en la carpeta
# descomprime 
unzip store-app-postgree.zip 
# comprueba que se ha descomprimido
cd store-app
```
![decomprimir](/images/2.png)

Me **descargo** los siguientes archivos:
+ [Dockerfile para crear la imágen](https://github.com/jmmedinac03vjp/PuestaProduccionSegura/blob/main/Unidad5-SistemasDesplegadoAutomatizadoSoftware/Actividad-DespliegueAutomatizadoSoftwareKubernetes/files/Dockerfile)
+ [Manifiesto ***Kubernetes*** para desplegar store-app](https://github.com/jmmedinac03vjp/PuestaProduccionSegura/blob/main/Unidad5-SistemasDesplegadoAutomatizadoSoftware/Actividad-DespliegueAutomatizadoSoftwareKubernetes/files/store-app-k8s.yaml)
![manifiestos](/images/3.png)

Imágen de los archivos ya creados y colocados en la carpeta de store-app (concretamente en a raíz)
![archivos creados](/images/4.png)

Antes de aplicar el manifiesto, entrar dentro del entorno docker y construir la imágen de la app dentro del entorno de Minikube para que ***Kubernetes*** puede usarla localmente ejecutando lo siguiente:
```bash
# iniciar minikube con un perfil llamado store-app.
minikube start --nodes 3 -p store-app
# Construye imagen store-app preparado para postgree
docker build -t store-app:latest .
# cargamos la imagen en el contesto ya que sino sólo está en el nodo de control
minikube image load store-app:latest --profile=store-app
# comprueba que se ha creado la imagen store-app correctamente en docker de minikube
minikube -p store-app image ls
# debe aparecer la imagen store-app:latest
```
![despliegue minikube](/images/5.png)

---

## DESPLIEGUE
Hago el despliegue en ***kubernetes***.
```bash
# aplica manifiesto kubernetes de despliegue
kubectl apply -f store-app-k8s.yaml  
```
![despliegue de kubernetes](/images/6.png)

Para comprobar si todo esta funcionando ejecuto:
```bash
minikube -p store-app service store-app
```
![para ver si funciona](/images/7.png)

Para comprobar si todo está funcionando ejecuto el comando:
```bash
minikube -p store-app service store-app
```
y se abrirá la aplicación en una web en firefox
![comprobaciones](/images/8.png)

---

## VERIFICACIÓN
Compruebo el estado del despliegue.
1. **Muestra** todos los **pods en ejecución**, su **estado** (Running), si están listos (1/1), cuántas veces reiniciaron y cuánto tiempo llevan activos.
```bash
kubectl get pods
```
![estado pods](/images/9.png)

2. **Muestra** todos los **Services (servicios) de Kubernetes, su tipo, IP interna, puerto expuesto y edad.**
```bash
kubectl get svc
```
![servicios](/images/10.png)

3. **Muestra** todos los **ConfigMaps existentes en el namespace actual.**
```bash
kubectl get configmap
```
![ConfigMaps](/images/11.png)

4. **Muestra** todos los **Secrets** existentes.
```bash
kubectl get secret
```
![secrets](/images/12.png)

5. **Muestra** los **logs del Deployment store-app** (toma los logs del primer pod por defecto).
```bash
kubectl logs deploy/store-app
```
!logs de deployment[](/images/13.png)

6. **Muestra** los **logs del Deployment de la base de datos.**
```bash
kubectl logs deploy/store-db
```
![logs base de datos](/images/14.png)

7. **Permite ejecutar psql** (cliente de PostgreSQL) **dentro del pod de la base de datos.**
```bash
kubectl exec -it deploy/store-db -- psql -U app -d store
```
![ejecuta psql](/images/15.png)

8. **Muestra la URL externa para acceder a tu servicio store-app.**
```bash
minikube -p store-app service store-app --url
```
![muetra utl](/images/16.png)

9. **Muestra** un **resumen** general **de los recursos más importantes en el namespace** (Pods, Services, Deployments y ReplicaSets).
```bash
kubectl get all
```
![resumen recursos](/images/17.png)

10. **Muestra las IPs** internas **de los pods que están detrás del Service store-app.**
```bash
kubectl get endpoints store-app
```
![ips internas](/images/18.png)

### RESPUESTAS A LAS PREGUNTAS

**1. ¿Qué diferencia hay entre ConfigMap y Secret?**  
El **ConfigMap** se utiliza para ***almacenar información de configuración no sensible*** (como *URLs*, *puertos* o *variables de entorno*). En cambio, el **Secret** está pensado para ***datos sensibles*** como *contraseñas* o *claves*, y Kubernetes los maneja de forma **más segura**.

**2. ¿Por qué PostgreSQL usa ClusterIP y no NodePort?**  
Porque la base de datos solo necesita ser accesible desde dentro del cluster por la aplicación `store-app`. El **ClusterIP** es un servicio interno y más seguro. Usar **NodePort expondría la base de datos fuera del clúster**, lo cual no es necesario y supondría un riesgo de seguridad.

**3. ¿Para qué sirve el initContainer?**  
El **initContainer** es un **contenedor** que se ejecuta **antes** del **contenedor principal** de la aplicación. Sirve para realizar comprobaciones o tareas previas, como esperar a que la base de datos esté lista antes de arrancar la aplicación.

**4. ¿Qué pasaría si borras el Secret?**  
Si borro el ***Secret***, la aplicación **no podrá leer la contraseña** de la base de datos. Los pods de `store-app` fallaran al iniciar y entraran en estado **CrashLoopBackOff**, haciendo que la aplicación deje de funcionar.

**5. ¿Qué cambiarías para que la base de datos tenga almacenamiento persistente?**  
Crearía un **PersistentVolumeClaim (PVC)** y lo montaría en la ruta `/var/lib/postgresql/data` del contenedor de PostgreSQL. Además, es recomendable cambiar el **Deployment** de `store-db` por un **StatefulSet**, ya que está pensado para aplicaciones con estado como las bases de datos.
