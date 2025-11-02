# 🧩 PFO3 – DevOps 3D  
## Despliegue de mini aplicación PHP en Kubernetes con Minikube y Docker  

---

## 📘 Descripción del proyecto  

Este proyecto consiste en el despliegue de una mini aplicación PHP dentro de un contenedor Docker, ejecutado en un clúster local de Kubernetes utilizando Minikube.  
El objetivo principal es aplicar los conceptos básicos de un entorno DevOps: construcción de imágenes, despliegue en contenedores y gestión de pods, servicios y recursos con Kubernetes.  

La aplicación muestra un mensaje con el nombre del host del contenedor, demostrando el correcto funcionamiento del entorno y la comunicación entre Docker, Minikube y Kubernetes.  

---

## 🛢️ Docker utilizado (imágenes y base)

- **Imagen de prueba (Parte I):** `kicbase/echo-server:1.0`  
  Usada para validar `Deployment` + `Service (NodePort)` y el acceso vía `minikube service`.

- **Imagen propia (Parte II):** `mi-miniapp:v1` (construida localmente).  
  **Base:** `php:fpm-alpine`  
  **Puerto expuesto en contenedor:** `80` (servidor embebido de PHP con `php -S 0.0.0.0:80`)  
  **Ruta de la app:** `/var/www/html` (convención estándar de aplicaciones web).  
  **Permisos:** `--chown=nobody` para evitar ejecutar con root y simplificar permisos de lectura/ejecución.  

Verificación rápida:
```bash
docker images | findstr mi-miniapp
docker images | findstr echo-server
```

---

## 🧱 Estructura del proyecto  

```
PFO3_DevOps/
├── src/
│   └── index.php
├── Dockerfile
├── deploy-mi-miniapp.yaml
└── README.md
```

- **index.php:** contiene el código PHP que genera el mensaje de bienvenida.  
- **Dockerfile:** define las instrucciones para crear la imagen Docker.  
- **deploy-mi-miniapp.yaml:** manifiesto de Kubernetes que describe el deployment.  
- **README.md:** documentación técnica y guía de ejecución del proyecto.  

---

## 🧰 Tecnologías utilizadas  

- Docker Desktop  
- Minikube  
- Kubernetes (kubectl)  
- PHP (imagen base alpine)  
- Windows 10 + WSL2  

---

## 🔧 Requisitos previos  

Antes de ejecutar el proyecto, se deben tener instaladas y configuradas las siguientes herramientas:  

- Docker Desktop  
- Minikube  
- Kubectl  
- Virtualización habilitada en BIOS (Intel VT-x o AMD-V)  

Comandos para verificar la instalación:  

```bash
docker --version
minikube version
kubectl version --client
```

---

## ▶️ Ejecución paso a paso  

### 1️⃣ Clonar el repositorio  

```bash
git clone https://github.com/Manuel-Correderas/PFO3_DevOps.git
cd PFO3_DevOps
```

---

### 2️⃣ Iniciar Minikube  

```bash
minikube start --driver=docker
```

Verificar el estado del clúster:  

```bash
kubectl get nodes
kubectl get po -A
```

Abrir el dashboard de Kubernetes (opcional):  

```bash
minikube dashboard
```

---

### 3️⃣ Construir la imagen Docker (imagen usada en el deploy)  

**Dockerfile (usado):**
```dockerfile
FROM php:fpm-alpine
RUN mkdir -p /var/www/html
COPY --chown=nobody src/ /var/www/html/
WORKDIR /var/www/html
CMD ["php", "-S", "0.0.0.0:80", "-t", "/var/www/html/"]
```

Construcción:
```bash
docker build -t mi-miniapp:v1 .
```

Verificar la creación de la imagen:  
```bash
docker images | findstr mi-miniapp
```

---

### 4️⃣ Probar la aplicación localmente  

```bash
docker run -p 8080:80 mi-miniapp:v1
```

Abrir el navegador y acceder a:  
👉 http://localhost:8080  

Mensaje esperado:  
> **Seminario Devop! Bienvenido a mi repo [hostname_del_contenedor]**  

---

### 5️⃣ Cargar la imagen en Minikube  

```bash
minikube image load mi-miniapp:v1
```

Verificar que se haya cargado correctamente:  

```bash
minikube image ls | findstr mi-miniapp
```

---

### 6️⃣ Crear el Deployment en Kubernetes  

Generar el manifiesto YAML:  

```bash
kubectl create deployment mi-miniapp --image=mi-miniapp:v1 --dry-run=client -o yaml > deploy-mi-miniapp.yaml
```

Aplicar el deployment:  

```bash
kubectl apply -f deploy-mi-miniapp.yaml
```

Verificar el estado del pod:  

```bash
kubectl get pods
kubectl get deployments
```

---

### 7️⃣ Exponer el servicio y acceder a la aplicación  

Exponer el deployment con un servicio NodePort:  

```bash
kubectl expose deployment/mi-miniapp --type="NodePort" --port=80
```

Verificar los servicios activos:  

```bash
kubectl get services
```

Abrir la aplicación en el navegador:  

```bash
minikube service mi-miniapp
```

Se abrirá automáticamente una pestaña con el mensaje de bienvenida generado por el pod en ejecución.  

---

## 🧠 Explicación técnica del funcionamiento  

- **Dockerfile:** construye la imagen a partir de `php:fpm-alpine` y copia el código a `/var/www/html`, ajustando el owner a `nobody`.  
- **Minikube:** ejecuta un clúster local con un único nodo Kubernetes.  
- **Deployment:** define la aplicación, crea el pod y gestiona su disponibilidad (réplicas, auto-restart).  
- **Service (NodePort):** expone el puerto 80 del contenedor hacia un puerto del nodo para acceso externo.  
- **minikube service:** resuelve la IP/puerto del Service y abre la URL en el navegador.  

Cada vez que se refresca la página, puede cambiar el hostname mostrado si Kubernetes recrea el pod o si hay múltiples réplicas.  

---

## 🧩 Comandos útiles  

| Acción | Comando |
|--------|----------|
| Ver todos los pods | `kubectl get pods` |
| Ver deployments | `kubectl get deployments` |
| Ver servicios | `kubectl get svc` |
| Eliminar deployment | `kubectl delete deployment mi-miniapp` |
| Eliminar servicio | `kubectl delete svc mi-miniapp` |
| Ver dashboard | `minikube dashboard` |
| Detener Minikube | `minikube stop` |

---

## 📊 Resultado esperado  

Una vez completados los pasos, el navegador muestra:  

> **Seminario Devop! Bienvenido a mi repo [hostname_del_pod]**  

Esto confirma que:  

- Docker y Minikube están correctamente configurados.  
- Kubernetes desplegó la aplicación sin errores.  
- El servicio NodePort expone la aplicación al exterior.  
- La comunicación entre contenedores y el host es funcional.  

---

## 📚 Créditos  

**Instituto:** IFTS N°29 – CABA  
**Materia:** DevOps 3D  
**Profesor:** Javier Blanco  
**Curso:** 3° Año D  
**Equipo 10 – HashTesters**  
- Manuel Correderas  
- María Nazar  
- Daniel Coria  

© 2025 – HashTesters 🚀  
