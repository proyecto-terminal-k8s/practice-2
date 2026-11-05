# Práctica 2: exponer la calculadora frontend

### Propósito

Desplegar tres réplicas del frontend de la calculadora y exponerlas mediante dos
tipos de Service:

- `ClusterIP`, accesible dentro del clúster o con `port-forward`.
- `NodePort`, accesible desde el equipo anfitrión mediante Minikube.

Los ejemplos muestran cómo DNS y selectores desacoplan las réplicas de su dirección
de acceso estable.

## Preparar la imagen

```bash
cd ../calculadora-frontend
```

Entra al directorio que contiene el código y la imagen Docker del frontend.

```bash
docker build -t calculadora-frontend:v1 .
```

Construye la imagen que usará el Deployment.

```bash
minikube image load calculadora-frontend:v1
```

Carga la imagen localmente dentro del clúster Minikube.

```bash
cd ../practice-2
```

El manifiesto ya usa la imagen local cargada en Minikube. Para usar un registro,
cambia la imagen a `tu-registro/calculadora-frontend:v1`, ejecuta `docker push` y
usa ese mismo nombre en `calculadora-deployment.yaml`.

## 1. Crear recursos

```bash
kubectl apply -f namespace.yaml
```

Crea el namespace `practice-2`.

```bash
kubectl apply -f calculadora-deployment.yaml
```

Crea el Deployment con tres réplicas del frontend.

```bash
kubectl apply -f calculadora-service-clusterip.yaml
```

Crea un Service estable de tipo `ClusterIP` para acceder al frontend dentro del clúster.

```bash
kubectl rollout status deployment/calculadora-frontend -n practice-2
```

Espera hasta que las tres réplicas estén disponibles.

```bash
kubectl get pods,endpointslices -n practice-2 -l app.kubernetes.io/name=calculadora-frontend
```

## 2. Probar `ClusterIP`

```bash
kubectl port-forward service/calculadora-frontend 8080:80 -n practice-2
```

Abre `http://localhost:8080`. El Service reparte la solicitud entre las réplicas
que estén listas. Esta práctica presenta únicamente el frontend; las operaciones
se conectarán al backend a partir de las prácticas posteriores.

## 3. Cambiar el Service a `NodePort`

```bash
kubectl apply -f calculadora-service-nodeport.yaml
```

Reemplaza el Service `ClusterIP` por uno `NodePort`, manteniendo el nombre del recurso.

```bash
kubectl get service calculadora-frontend -n practice-2
```

Comprueba que el Service expone el puerto configurado `30080`.

```bash
minikube service calculadora-frontend --namespace=practice-2 --url
```

El puerto externo configurado es el `30080`.

## Limpieza

```bash
kubectl delete namespace practice-2
```
