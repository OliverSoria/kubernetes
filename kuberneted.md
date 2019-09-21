# Kubernetes

### Resumen y comandos más importantes de Kubernetes

Un **pod** es un envoltorio de un _contenedor<br/>
Los **pods** se definen en un archivo yml<br/>
Los pods **no** se pueden ver desde afuera del cluster, o sea que no es posible acceder a un pod por su ip/puerto<br/>
**minikube ip** nos devuelve la ip del minukube para poder conectarnos a sus servicios<br/>
**kubectl get pods/kubectl get po** muestra todo los _pods_ que están corriendo<br/>
**kubectl get services/kubectl get service** muestra todo los _servicios_ que están corriendo<br/>
**kubectl get all** muestra todo lo que tenemos definido en el cluster<br/>
**kubectl apply -f name-file.yml** crea un pod en base al archivo leído<br/>
**kubectl describe pod name-of-pod** describe el pod con el nombre que especifiquemos<br/>
**kubectl describe service name-of-service** describe el servicio con el nombre que especifiquemos<br/>
**kubectl delete pod mame-of-pod** borra el pod específicado<br/>
**kubectl delete pods --all** elimina todos los pods<br/>
**kubectl describe rs name-of-replica-set** Describe el _ReplicaSet_ que indiquemos <br/>

### Comandos más avanzados

**kubectl -it exec webapp sh** comando utiliado para conectarnos con un pod de forma interactiva y poder ejecutar comandos directamente a ese pod<br/>
**kubectl get po --show-labels** muestra los pods y además agerga las etiquetas<br/>
**kubectl get po --show-labels -l release=0** muestra los pods, filtrando por la etiqueta especificada y agregando la etiqueta, en el ejemplo se realiza un filtrado por la etiqueta _release=0_<br/>


### Creación de un pod

Un pod lo empezamos por definir en un archivo yml, ejemplo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  containers:
    - name: webapp
      image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
```
Posteriormente debemos desplegar ese pod, para lo cual usaremos el comando **kubectl apply -f mi-pod.yml**

### Creación de un servicio

Un pod no puede ser accedido desde afuera, pero un servicio sí, por ese motivo crearemos un servicio que nos permita acceder al pod creado anteriormente, esto se logra por medio de un archivo yml, ejemplo:

```yaml
apiVersion: v1
kind: Service
metadata:
  # Unique key of the Service instance
  name: fleetman-webapp

spec:
  selector:
    app: webapp

  ports:
    - port: 80
      name: http
      nodePort: 30080

  type: NodePort
```

Posteriormente desplegamos el servicio con el mismo comando que se despliega un pod **kubectl apply -f mi-servicio.yml**<br/>

Nota: es importante señalar que el _label_ del servicio debe coincider con el _app_ del pod para que tengan comunicación.

Ahora, para acceder a la aplicación y validar que todo esté bien, necesitamos la ip del minikube dado que todo lo estamos corriendo en un minikube, entonces ejecutamos: **minikube ip**, nos devielve la ip eg. 192.168.99.100 a la cual solo agregamos el puerto que definimos en el servicio y listo: http://192.168.99.100:30080/<br/>

### Un servicio, múltiples pods

Se puede tener un solo servicio y múltiples pods, de tal forma que el servicio se conecta a un solo pod, retomando el ejemplo anterior de _un pod-un servicio_, ahora crearemos un segundo pod, agergando ademas la propiedad _release_ dentro de _labels_, obviamente el _name_ del segundo pod debe ser disitnto del primer pod. entonces tenemos las siguientes definiciones de los pods:<br/>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: webapp
    release: "0"
spec:
  containers:
    - name: webapp
      image: richardchesterwood/k8s-fleetman-webapp-angular:release0
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-release-0-5
  labels:
    app: webapp
    release: "0-5"
spec:
  containers:
    - name: webapp
      image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
```
Nótese que el _name_ es distinto, así como el _release_ y por supuesto también la propiedad _image_.<br/>

Respecto al servicio tenemos que agregar la propiedad _release_ dentro de _selector_, obteniendo lo siguiente:

```yaml
apiVersion: v1
kind: Service
metadata:
  # Unique key of the Service instance
  name: fleetman-webapp

spec:
  selector:
    app: webapp
    release: "0-5"

  ports:
    - port: 80
      name: http
      nodePort: 30080

  type: NodePort
```

Ahora solo falta desplegar el respectivo pod y desde luego el servicio.<br/>

###ReplicaSets

<ul>
<li>Lo primero que vamos a cambiar será el _kind_, que antes era _Pod_, ahora será _ReplicaSet_, ya que no puede ser ambos. </li>
<li>El _name_ pasa a ser algo secundario ya que los nombres se generan en automático, así que lo borramos.</li>
<li>En este ejemplo en particular el _release_ tampoco es necesario.</li>
<li>Agregamos la sección _spec_, donde se especifica el número de réplicas y el _template_.</li>
<li>El _template_ tiene la información del pod, que hemos venido usando con anterioridad, es decir _metadata_ y _spec_.</li>
<li>Es muy importante agregar un _name_ para nuestro _ReplicaSet_, esto lo hacemos agregando la etiqueta _metadata_ y anidamos _name_.</li>
<li>También en _apiVersion_, aparte de la versión agregamos "_apps/_".</li>
<li>Dentro de _spec_ tenemos que agregar el _selector_, el cual a su vez anida _matchLabels_ y éste a su vez agrega la etiqueta _app_, necesaria para hacer "match" con el _label_ dentro del _template_.</li>
</ul>

Con todos estos cambios obtenemos el siguiente archivo:<br/>

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp
          image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
  selector:
    matchLabels:
      app: webapp
```

Ahora solo falta aplicar los cambios con el comando kubectl apply -f **nombre-del-archivo.yml** y en caso de que queramos ver sus detalles, podemos recurrir al siguiente comando: kubectl describe rs **name-of-replica-set**<br/>

###Deployments






