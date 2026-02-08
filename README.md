# Documento de Diseño Técnico  
## Sistema de Control de Inventario y Stock (SCIS)

**Por:** Jehovani Chavez

---

## Tabla de contenidos
- [Instrucciones de instalación](#instrucciones-de-instalación)
- [Selección de tecnologías](#selección-de-tecnologías)
  - [Backend](#backend)
  - [Frontend](#frontend)
- [Motor de base de datos](#motor-de-base-de-datos)
- [Diagramas de arquitectura](#diagramas-de-arquitectura)
- [Estrategia de branching](#estrategia-de-branching)
- [Deploy a Google Cloud Run](#Deploy-a-Google-Cloud-Run)
- [Demo Movil y Web](#Demo-Movil-y-Web)

---

## Instrucciones de instalación

1. Nos registramos en Google Cloud e ingresamos a **Cloud SQL**.

   <p align="center">
     <img src="https://github.com/jesussv/jc-pt-mopt/blob/main/Cloud%20SQL.jpg" width="720" />
   </p>

2. Creamos una instancia seleccionando el servidor con el plan que necesitemos con **PostgreSQL**.

   <p align="center">
     <img src="https://github.com/jesussv/jc-pt-mopt/blob/main/InstanciaGoogleCloud.jpg" width="720" />
   </p>

3. Creamos una nueva base de datos y abrimos el archivo `backup_schema.sql` con Bloc de notas u otro editor para copiar el contenido y pegarlo en el editor.

   <p align="center">
     <img src="https://github.com/jesussv/jc-pt-mopt/blob/main/bd.jpg" width="720" />
   </p>

4. Descargamos el proyecto adjunto.

## Deploy a Google Cloud Run

> Este deploy aplica para el **Backend (ASP.NET Core Minimal APIs .NET 8)**.

### Prerrequisitos
- Tener un proyecto en Google Cloud.
- Tener habilitado:
  - **Cloud Run**
  - **Cloud Build**
  - **Artifact Registry**
  - **Secret Manager**
- Tener instalado y autenticado `gcloud` (o usar **Cloud Shell**).

---

### 1) Autenticación y proyecto
- bash
- gcloud auth login
- gcloud config set project (Nombre proyecto)
- gcloud config set run/region
---
Pasos.
1. Hay que Ubicarse en la carpeta donde está el backend (donde está el .csproj):
   cd services/JC.LocationIngest
---
3. Deploy:
gcloud run deploy c-location-ingest-dev \
  --source . \
  --region us-central1 \
  --allow-unauthenticated
la API debe ser privada, quitamos --allow-unauthenticated y usamos IAM.
---
4. Deploy con Cloud SQL (PostgreSQL) + Secrets
   Cloud SQL y variables sensibles en Secret Manager:

gcloud run deploy c-location-ingest-dev \
  --source . \
  --region us-central1 \
  --add-cloudsql-instances evocative-reef-133021:us-central1:jchavez-pt-mopt-dev2026 \
  --set-secrets "ConnectionStrings__JCPostgres=jc-connstr:latest,Jwt__Key=jc-jwt-key:latest" \
  --allow-unauthenticated
  
Obtener la URL del servicio
gcloud run services describe c-location-ingest-dev \
  --region us-central1 \
  --format="value(status.url)"

---
5. Validación rápida (health/status)

power-Shell
curl -s "$(gcloud run services describe c-location-ingest-dev --region us-central1 --format='value(status.url)')/health"
Ajustar /health por  endpoint real si se llama distinto.

---
6. Variables de entorno 

Para ver variables configuradas en el contenedor:

gcloud run services describe c-location-ingest-dev \
  --region us-central1 \
  --format="yaml(spec.template.spec.containers[0].env)"
### Demo Movil y Web
## 🚀 DESCARGA LA APP DEMO
> **Instala SCIS y empezá a probar como controlar un inventario.**  
✅ Login seguro • ✅ Entradas/Salidas • ✅ Ajustes • ✅ Transferencias • ✅ Stock por bodega • ✅ Movimientos

<p align="center">
  <a href="https://drive.google.com/file/d/1As7YoFjR2aEGx0lt0hOshhIq4LVGYQHV/view?usp=sharing">
    <img src="https://img.shields.io/badge/📥%20DESCARGAR%20APP-Releases-brightgreen?style=for-the-badge" />
  </a>
</p>
<!-- ✅ Preview Movil -->
<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/main/apk1.png" width="720" />
</p>
<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/main/apk2.png" width="720" />
</p>
<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/main/apk3.png" width="720" />
</p>

## 🚀 VERSION WEB DEMO
### <!-- ✅ Preview WEB -->
<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/main/WEB1.png" width="720" />
</p>
<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/main/WEB2.png" width="720" />
</p>
<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/main/apk3.png" width="720" />
</p>


## Selección de tecnologías

### Backend

**Justificación:**

1. **Tecnológico:** ASP.NET Core Minimal APIs en .NET 8 para una API REST ligera, rápida y estable (CRUD de productos, movimientos, entradas/salidas, kardex y consultas con buena concurrencia).  
   Hosting en **Cloud Run** (contenedores serverless) por escalado automático, despliegue por Docker y cero administración de VMs/parches.  
   Seguridad con **JWT Bearer** (stateless) y roles (bodega, admin, auditor).  
   Configuración sensible en **Secret Manager** (ej.: Jwt Key).

2. **Económico:** Cloud Run se paga por demanda (requests/CPU/memoria), evitando pagar infraestructura 24/7 en horas de bajo tráfico y reduciendo el costo operativo.

3. **Recurso humano:** C#/.NET es stack empresarial con talento disponible; Minimal APIs reduce boilerplate y acelera la entrega sin perder mantenibilidad ni escalabilidad.

**Stack:**
- **Framework:** ASP.NET Core Minimal APIs  
- **Lenguaje/Runtime:** C# / .NET 8  
- **Tipo de app:** API REST  
- **Hosting:** Cloud Run (serverless containers)  
- **Auth:** JWT Bearer  
- **Secrets:** Secret Manager  

---

### Frontend

**Justificación:**

1. **Tecnológico:** Flutter (Dart) para app multiplataforma real (Android/iOS y opcional Web) con una sola base de código. UI con Widgets (Material/Cupertino) para pantallas típicas de inventario: catálogo, existencias, registro/escaneo y movimientos.  
   Consumo del backend .NET vía HTTP/HTTPS y JSON.

2. **Seguridad/Auth:** Login obtiene el JWT y luego se envía en cada request:  
   `Authorization: Bearer <token>`  
   Sesiones stateless y permisos por rol/claims desde backend.

3. **Económico:** Un solo desarrollo para múltiples plataformas reduce costo y mantenimiento.

4. **Recurso humano:** Un solo stack (Flutter/Dart), componentes reutilizables y ciclo de desarrollo rápido.

**Stack:**
- **Tipo:** App multiplataforma (Android / iOS / Web)  
- **Framework:** Flutter  
- **Lenguaje:** Dart  
- **UI:** Material/Cupertino  
- **Consumo de API:** HTTP/HTTPS + JSON  
- **Auth:** JWT Bearer Token  

---

## Motor de base de datos

**Justificación:**

1. **Tecnológico:** PostgreSQL en Cloud SQL por integridad referencial, transacciones y consistencia en movimientos (entradas/salidas, ajustes, kardex). Concurrencia sólida y menos bloqueos entre lecturas/escrituras.

2. **Acceso a datos:** Dapper + Npgsql para rendimiento y control (micro-ORM liviano y driver estable).

3. **Seguridad:** Consultas parametrizadas con Dapper/Npgsql para prevenir SQL Injection.

4. **Escalabilidad y costo:** Cloud SQL soporta escalado (tamaño/rendimiento/réplicas) y reduce administración (backups/parches), mejorando control del gasto.

**Stack:**
- **Base de datos:** PostgreSQL (Cloud SQL)  
- **Acceso:** Dapper + Npgsql  

---

## Diagramas de arquitectura

La arquitectura propuesta se basa en el esquema **cliente → API → base de datos**, usando servicios cloud administrados para escalar y reducir operación.

**Patrón arquitectónico:** Modular Monolith (escalable según necesidades futuras).

### 1) Diagrama de contexto (C4 - Nivel 1)

<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/assets/SCIS_C4_Nivel1_Contexto.jpg" width="720" />
</p>

### 2) Diagrama de contenedores (C4 - Nivel 2)

<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/assets/SCIS_C4_Nivel2_Contenedores.jpg" width="720" />
</p>

### 3) Diagrama de componentes (C4 - Nivel 3)

<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/assets/SCIS_C4_Nivel3_Componentes.jpg" width="720" />
</p>

### 4) Diagrama de código (C4 - Nivel 4)

<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/assets/SCIS_C4_Nivel4_Codigo.jpg" width="720" />
</p>

---

## Estrategia de branching

Se utilizará **Trunk Based Development**, PRs cortos y frecuentes + CI fuerte + feature flags, para que 8 devs trabajen en paralelo sin conflictos bloqueantes ni ramas eternas.

### Flujo a utilizar

<p align="center">
  <img src="https://github.com/jesussv/jc-pt-mopt/blob/assets/SCIS_Git_Branching_Flujo.jpg" width="720" />
</p>
