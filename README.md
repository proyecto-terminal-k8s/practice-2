# Práctica 2: exponer la calculadora frontend

### Propósito

Desplegar tres réplicas del frontend de la calculadora y exponerlas mediante dos
tipos de Service:

- `ClusterIP`, accesible dentro del clúster o con `port-forward`.
- `NodePort`, accesible desde el equipo anfitrión mediante Minikube.

Los ejemplos muestran cómo DNS y selectores desacoplan las réplicas de su dirección
de acceso estable.

## 1. Entrar al proyecto frontend

Entra al directorio que contiene el frontend.

```bash
cd ../calculadora-frontend
```

## 2. Construir la imagen

Construye la imagen que usará el Deployment.

```bash
docker build -t calculadora-frontend:v1 .
```

## 3. Cargar la imagen en Minikube

Carga la imagen localmente dentro del clúster.

```bash
minikube image load calculadora-frontend:v1
```

## 4. Volver a los manifiestos

Regresa al directorio de esta práctica.

```bash
cd ../practice-2
```

El manifiesto ya usa la imagen local cargada en Minikube. Para usar un registro,
cambia la imagen a `tu-registro/calculadora-frontend:v1`, ejecuta `docker push` y
usa ese mismo nombre en `calculadora-deployment.yaml`.

## 5. Crear el namespace

Crea el namespace `practice-2`.

```bash
kubectl apply -f namespace.yaml
```

## 6. Crear el Deployment

Crea el Deployment con tres réplicas del frontend.

```bash
kubectl apply -f calculadora-deployment.yaml
```

## 7. Crear el Service ClusterIP

Crea el Service estable para acceder al frontend dentro del clúster.

```bash
kubectl apply -f calculadora-service-clusterip.yaml
```

## 8. Esperar las réplicas

Espera hasta que las tres réplicas estén disponibles.

```bash
kubectl rollout status deployment/calculadora-frontend -n practice-2
```

## 9. Verificar Pods y endpoints

Comprueba las réplicas y los endpoints asociados al selector del Service.

```bash
kubectl get pods,endpointslices -n practice-2 -l app.kubernetes.io/name=calculadora-frontend
```

## 10. Abrir el acceso mediante ClusterIP

Mantén el proceso abierto y abre `http://localhost:8080`. El Service reparte las solicitudes entre las réplicas listas.

```bash
kubectl port-forward service/calculadora-frontend 8080:80 -n practice-2
```

## 11. Cambiar el Service a NodePort

Reemplaza el Service `ClusterIP` por uno `NodePort`, manteniendo el mismo nombre.

```bash
kubectl apply -f calculadora-service-nodeport.yaml
```

## 12. Verificar el puerto NodePort

Comprueba que el Service expone el puerto externo configurado `30080`.

```bash
kubectl get service calculadora-frontend -n practice-2
```

## 13. Obtener la URL de Minikube

Abre o copia la URL generada por Minikube para el servicio.

```bash
minikube service calculadora-frontend --namespace=practice-2 --url
```

## 14. Limpiar la práctica

Elimina el namespace y todos los recursos asociados.

```bash
kubectl delete namespace practice-2
```
