## Fase 1: Infraestructura de Red (VPC y Subredes)
El primer paso fue crear el entorno de red aislado en AWS para alojar las máquinas.

* **Creación de la VPC:** En la consola de AWS, se creó una Virtual Private Cloud (VPC) con un bloque CIDR (por ejemplo, `10.0.0.0/16`).
* **Creación de las Subredes:** Se dividió la VPC en tres subredes distintas para mantener la separación de capas:
  * **Subred Pública:** Para el Frontend.
  * **Subred Privada 1:** Para el Backend.
  * **Subred Privada 2:** Para la Base de Datos.
* **Configuración de Enrutamiento:** Se creó un Internet Gateway (IGW) y se conectó a la VPC. Luego, se creó una Tabla de Enrutamiento Pública que enviaba el tráfico `0.0.0.0/0` hacia el IGW y se asoció solo a la Subred Pública. Las subredes privadas se configuraron sin ruta directa a internet.

## Fase 2: Grupos de Seguridad (Firewalls)
Se configuraron las reglas de entrada (Inbound Rules) para dictar quién podía comunicarse con quién.

* **Grupo de Seguridad Frontend (`sg-frontend`):**
  * Se permitió HTTP (Puerto `80`) desde Cualquier IPv4 (`0.0.0.0/0`).
  * Se permitió SSH (Puerto `22`) desde la IP local o cualquier IPv4.
  * Se permitió ICMP (Ping) desde cualquier IPv4.
* **Grupo de Seguridad Backend (`sg-app` o `sg-backend`):**
  * Se permitió TCP Personalizado (Puerto `3001`) solo desde el origen `sg-frontend`.
  * Se permitió SSH (Puerto `22`) desde cualquier IPv4 (o desde la red del laboratorio).
* **Grupo de Seguridad Base de Datos (`sg-db`):**
  * Se permitió MySQL (Puerto `3306`) solo desde el origen `sg-app` (Backend).
  * Se permitió SSH (Puerto `22`).

## Fase 3: Creación de Instancias EC2 e Instalaciones
Con la red y la seguridad listas, se desplegaron los servidores virtuales.

* **Lanzamiento de Instancias:** Se crearon tres máquinas EC2 (utilizando Amazon Linux 2023, por ejemplo) y se colocaron en sus respectivas subredes con sus respectivos Grupos de Seguridad. Se aseguró de que solo la EC2 del Frontend recibiera una IP Pública.
* **Conexión:** Se accedió a cada máquina mediante SSH o AWS Systems Manager (SSM).
* **Instalación de Git y Docker:** Se ejecutaron los siguientes comandos en las tres instancias para prepararlas:
  * Actualizar paquetes: `sudo yum update -y`
  * Instalar Git: `sudo yum install git -y`
  * Instalar Docker: `sudo yum install docker -y`
  * Iniciar el servicio: `sudo systemctl start docker`
  * Habilitar Docker al arranque: `sudo systemctl enable docker`
  * Dar permisos al usuario: `sudo usermod -aG docker ec2-user`

## Fase 4: Repositorio y Workflows en GitHub
Con las máquinas listas, se preparó el código y la automatización.

* **Estructura del Proyecto:** Se creó un repositorio en GitHub con tres carpetas principales (`frontend`, `backend`, `db`), cada una con su código y su propio `Dockerfile`.
* **Configuración de IPs (Hardcodeo):** * En `frontend/default.conf`, se apuntó el proxy inverso a la IP privada de la EC2 del Backend.
  * En `backend/server.js`, se configuró la variable `DB_HOST` con la IP privada de la EC2 de la Base de Datos.
* **Creación de Workflows:** Dentro de la carpeta `.github/workflows/`, se crearon los archivos YAML (`cicd-tienda-frontend.yml`, `cicd-tienda-backend.yml`, `cicd-tienda-db.yml`). Estos archivos le indicaron a GitHub cómo construir las imágenes Docker, subirlas a Amazon ECR y ejecutar los comandos remotos vía SSM en las EC2 usando la política `--restart unless-stopped`.

## Fase 5: Adición de Secrets en GitHub
Para que los flujos de trabajo tuvieran permisos en AWS, se inyectaron credenciales seguras.

* Se ingresó a la configuración del repositorio en GitHub > Secrets and variables > Actions.
* Se añadieron las credenciales de sesión de AWS: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` y `AWS_REGION`.
* Se añadieron los IDs exactos de las máquinas: `EC2_FRONTEND_INSTANCE_ID`, `EC2_BACKEND_INSTANCE_ID`, `EC2_DB_INSTANCE_ID`.
* Se añadieron las URLs de los repositorios ECR: `ECR_REGISTRY`, `ECR_REPO_URL_FRONTEND`, `ECR_REPO_URL_BACKEND`, `ECR_REPO_URL_DB`.

## Fase 6: Despliegue y Visualización
El paso final para que todo cobrara vida.

* **Trigger de GitHub Actions:** Se realizó un `git push`. Esto activó los tres flujos de trabajo en paralelo. Las imágenes se construyeron en la nube, se guardaron en ECR y los contenedores se levantaron dentro de las EC2.
* **Visualización:** Se abrió el navegador web, se ingresó la Dirección IP Pública de la EC2 del Frontend. Nginx sirvió el archivo `index.html` y redirigió las peticiones API de forma interna.
