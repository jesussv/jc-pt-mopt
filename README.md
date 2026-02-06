# Documento de Diseño Técnico Sistema de Control de Inventario y Stock (SCIS)
#### Por: Jehovani Chavez
## Instrucciones de instalación
1. Nos registramos en Google Cloud e ingresamos a Cloud SQL.
   ![](https://github.com/jesussv/jc-pt-mopt/blob/main/Cloud%20SQL.jpg)
2. Creamos una Instancia seleccionando el servidor con el plan que necesitemos con PostgreSQL.
   ![](https://github.com/jesussv/jc-pt-mopt/blob/main/InstanciaGoogleCloud.jpg)
3. Creamos una nueva base de datos y abrimos el archivo backup_schema.sql con bloc de notsa u otro editor para copiar el contenido y pegarlo en el editor
   ![](https://github.com/jesussv/jc-pt-mopt/blob/main/bd.jpg)
4. Descargamos el proyecto adjunto.
   
## Selección de Tecnologías. 
### Backend
#### Para el sistema de gestión de inventarios, yo lo justifico así:
1. Tecnológico: Me voy por ASP.NET Core Minimal APIs en .NET 8 porque necesito una API REST ligera, rápida y estable para lo típico de inventarios: CRUD de productos, movimientos, entradas/salidas, kardex y consultas con buena concurrencia. En hosting uso Cloud Run (serverless containers) porque me da escalado automático, despliegue por Docker, y no tengo que administrar VMs ni parches. La seguridad la manejo con JWT Bearer para trabajar y así no guardo información de la app y cada solicitud de cliente es independiente, con roles por usuario (bodega, admin, auditor). Y las llaves configuración sensibles las guardo en Secret Manager (por ejemplo Jwt Key) para no exponer secretos en código ni en variables sueltas.
2. Económico: Con Cloud Run pago por demanda (requests/CPU/memoria), así que no estoy pagando un servidor encendido 24/7 cuando el sistema tenga horas de poco tráfico. Además, bajo el costo operativo porque reduzco soporte de infraestructura.
3. Recurso humano: C#/.NET es un stack empresarial y hay talento disponible; y con Minimal APIs reduzco código repetitivo y acelero la entrega, sin sacrificar mantenibilidad ni escalabilidad.
- Framework: ASP.NET Core Minimal APIs Lenguaje/Runtime: C# / .NET 8
- Tipo de app: API REST
- Hosting: Serverless container en Cloud Run
- Auth: JWT Bearer
- Config/Secrets: Secret Manager
### Frontend 
#### Para el frontend del sistema de inventarios, yo lo justifico así:
1. Tecnológico: Me voy con Flutter (Dart) porque necesito una app multiplataforma real (Android/iOS y si quiero Web) con una sola base de código. La UI la construyo con Widgets (Material/Cupertino), lo cual me da consistencia y velocidad para pantallas típicas de inventario: catálogo, existencias, escaneo/registro, movimientos. La app consume mi backend .NET vía HTTP/HTTPS con JSON, que es estándar y fácil de debuggear.
2. Seguridad/Auth: La autenticación queda limpia: el login obtiene el JWT y luego la app manda el Bearer token en el header Authorization: Bearer <token> en cada request. Así manejo sesiones stateless y permisos por rol/claims desde el backend.
3. Económico: Con Flutter reduzco costo porque no desarrollo 2 o 3 apps separadas; mantengo una sola y despliego a varias plataformas, lo que baja tiempo y mantenimiento.
4. Recurso humano: El equipo se enfoca en una sola tecnología (Flutter/Dart) y el ciclo de desarrollo es rápido (componentes reutilizables). Eso facilita mantener y evolucionar el sistema sin duplicar esfuerzo.
- Tipo: App multiplataforma (Android / iOS / Web).
- Framework: Flutter.
- Lenguaje: Dart.
- UI: Widgets de Flutter (Material/Cupertino)
- Consumo de API: llamadas HTTP/HTTPS a tu backend (.NET) enviando/recibiendo JSON
- Auth: envía el Bearer token (JWT) en el header Authorization

## Motor de Base de Datos 
### Para el motor de base de datos del sistema de inventarios, yo lo justifico así:
1. Tecnológico: Uso PostgreSQL en Cloud SQL porque para inventarios necesito un motor relacional fuerte: integridad referencial, transacciones y consistencia en movimientos (entradas/salidas, ajustes, kardex). Además, PostgreSQL maneja bien la concurrencia, permite reducir bloqueos entre operaciones de lectura y escritura, asegurando mejor desempeño cuando hay múltiples usuarios operando al mismo tiempo.
2. Acceso a datos: Me voy con Dapper + Npgsql porque quiero rendimiento y control. Dapper es un micro-ORM liviano y me deja optimizar SQL e índices según el caso; Npgsql es el driver estable para Postgres en .NET.
3. Seguridad: Para prevenir SQL Injection, uso consultas parametrizadas con Dapper/Npgsql, así los valores entran como parámetros y no como parte del SQL.
4. Escalabilidad y costo: Cloud SQL soporta escalabilidad (tamaño, rendimiento, réplicas según necesidad) minimizando costos operativos: menos administración de infraestructura (backups, parches) y mejor control del gasto, lo cual impacta directo en lo económico.
- Base de datos: PostgreSQL (Cloud SQL) relacional.
- Acceso a DB: Dapper + Npgsql para prevención de inyecciones.

## Diagrama de Arquitectura
### La arquitectura propuesta para la solución queda montada en un esquema cliente → API → base de datos, todo orientado a servicios cloud administrados para escalar y reducir operación.
Patrón arquitectónico: Modular Monolith escalable según necesidades futuras.
#### 1-	Diagrama de contexto:
![](https://github.com/jesussv/jc-pt-mopt/blob/assets/SCIS_C4_Nivel1_Contexto.jpg)
#### 2- Diagrama de Arquitectura:
![](https://github.com/jesussv/jc-pt-mopt/blob/assets/SCIS_C4_Nivel2_Contenedores.jpg)
#### 3- Diagrama de componentes: 
![](https://github.com/jesussv/jc-pt-mopt/blob/assets/SCIS_C4_Nivel3_Componentes.jpg)
#### 4- 4-	Diagrama de código:
![](https://github.com/jesussv/jc-pt-mopt/blob/assets/SCIS_C4_Nivel4_Codigo.jpg)

## Estrategia de Branching
### Se utilizará Trunk Based Development, PR cortos y frecuentes + CI fuerte + feature flags, porque así 8 devs pueden trabajar en paralelo sin crear conflictos bloqueantes ni ramas eternas.
#### Flujo a utilizar:
![](https://github.com/jesussv/jc-pt-mopt/blob/assets/SCIS_Git_Branching_Flujo.jpg)




