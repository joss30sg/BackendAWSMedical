
## 📦 Descarga del proyecto

Este repositorio contiene el código fuente del backend para el sistema **Citas Médicas AWS**.  
Debido a que GitLab no permite subir archivos mayores a 100 MB, el paquete completo (build)  
ha sido alojado externamente en Google Drive.

👉 **[Descargar BackendAWSMedical.zip (400 MB)](https://drive.google.com/file/d/1Dm8pQRkwyCM3QSFHy0i53tP58BJIR_rN/view?usp=sharing)**

> 💾 Contiene la versión compilada y lista para despliegue en AWS Lambda.
>  
> Si prefieres compilar el proyecto localmente, consulta la sección **Instalación y ejecución**.

# 🏥 Appointment Service API

Backend para el **agendamiento de citas médicas** en Perú y Chile, desarrollado con **Node.js**, **TypeScript**, **AWS Lambda**, **Aurora Serverless v2**, y el **framework Serverless**.

---

## 🌐 URL de despliegue
**Entorno de desarrollo / producción:**
➡️ https://AgendarCitasMedicas.execute-api.us-east-1.amazonaws.com/dev

---

## 🚀 Endpoints Principales

| Método | Endpoint | Descripción |
|---------|-----------|-------------|
| **POST** | `/appointments` | Registra una nueva cita médica (envía datos al DynamoDB con estado *pending*). |
| **GET** | `/appointments/{insuredId}` | Lista las citas de un asegurado con su estado actual (*pending*, *completed*). |
| **PATCH** | `/appointments/{appointmentId}` | Actualiza información o estado de una cita. |
| **GET** | `/health` | Verifica el estado del servicio. |

---

## 🧾 Documentación OpenAPI / Swagger

La especificación completa de la API se encuentra en el archivo:

📄 [`swagger/openapi.yaml`](./swagger/openapi.yaml)

Puedes visualizarla con Swagger UI localmente ejecutando:
```bash
npx swagger-ui-dist ./swagger/openapi.yaml
```

O acceder directamente al endpoint desplegado (si se configuró Swagger Gateway):
➡️ https://AgendarCitasMedicas.execute-api.us-east-1.amazonaws.com/dev/docs

### 📘 Ejemplo de `swagger/openapi.yaml` (extracto)
```yaml
openapi: 3.0.3
info:
  title: Appointment Service API
  version: 1.0.0
  description: API para la gestión de citas médicas en Perú y Chile.
paths:
  /appointments:
    post:
      summary: Crea una nueva cita médica
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                insuredId:
                  type: string
                  example: "00042"
                scheduleId:
                  type: integer
                  example: 100
                countryISO:
                  type: string
                  enum: [PE, CL]
                  example: "PE"
      responses:
        "200":
          description: Cita registrada exitosamente
    get:
      summary: Lista todas las citas por código de asegurado
      parameters:
        - in: path
          name: insuredId
          schema:
            type: string
          required: true
          description: Código del asegurado (5 dígitos)
      responses:
        "200":
          description: Lista de citas con estado actual
```

---

## ☁️ Arquitectura AWS

La aplicación utiliza una arquitectura serverless completamente administrada en AWS:

```
Cliente → API Gateway → Lambda (appointment)
        ↳ DynamoDB (estado pending)
        ↳ SNS → SQS (por país)
             ↳ Lambda (appointment_pe / appointment_cl)
                 ↳ Aurora MySQL (RDS)
                 ↳ EventBridge → SQS (confirmación)
        ↳ Lambda (appointment) actualiza estado a completed
```

### Recursos creados automáticamente:
- **API Gateway**
- **Lambda Functions** (`appointment`, `appointment_pe`, `appointment_cl`)
- **DynamoDB** (almacenamiento temporal de estado)
- **Aurora Serverless v2 (MySQL)** — persistencia final
- **SNS & SQS** — colas de procesamiento por país (`SQS_PE`, `SQS_CL`)
- **EventBridge** — eventos de confirmación
- **Secrets Manager** — credenciales seguras de Aurora
- **IAM Roles** — permisos granulares
- **DLQ (Dead Letter Queue)** — recuperación ante fallos

---

## 🧩 Despliegue con Serverless Framework

### 1. Configuración previa
Asegúrate de tener configurado el CLI de AWS:
```bash
aws configure
```
(usa la cuenta `727646509015` en la región `us-east-2`)

### 2. Despliegue
```bash
npm install
npm run build
npx serverless deploy --region us-east-2 --stage dev
```

O usa el script automatizado:
```bash
cd aws
chmod +x deploy.sh
./deploy.sh
```

### 3. Inicializar la base de datos Aurora
```bash
aws rds-data execute-statement   --resource-arn <AuroraClusterArn>   --secret-arn <AuroraSecretArn>   --database appointmentsdb   --sql file://db/schema.sql
```

---

## 🧪 Pruebas

Ejecutar pruebas unitarias:
```bash
npm test
```

Cobertura mínima recomendada: **80%**

---

## 📜 Licencia
MIT © 2025 - Equipo BackendAWSMedical

