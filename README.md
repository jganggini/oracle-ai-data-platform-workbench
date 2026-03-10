[![Issues][issues-shield]][issues-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<!-- Intro -->
<br />
<div align="center" style="text-align:center;">
  <h1 style="font-size:40px; font-bload"><b style="color:#ec4b42">Oracle AI</b> Data Platform Workbench</h1>
  
  <p>Laboratorio guiado: Análisis de Sentimiento con OCI Language + Autonomous Database + Agent Factory</p>
  
  <a style="font-size:large;" href="https://objectstorage.us-chicago-1.oraclecloud.com/n/axzs95biupm2/b/labs/o/lab.zip">📄 Descargar Plantilla »</a>
  <br/>
  <a href="https://github.com/jganggini/oracle-ai-data-platform-workbench/issues">💣 Report Bug</a>
  ·
  <a href="https://github.com/jganggini/oracle-ai-data-platform-workbench/pulls">🚀 Request Feature</a>
    
  <div align="center">
  <a href="https://www.youtube.com/watch?v=zIxA0ebo2v4&list=PLMUWTQHw13gZlzbOKtsJwXOGizrOCsOkx">
    <img src="https://img.shields.io/badge/YouTube-Ver%20Tutorial-red?style=for-the-badge&logo=youtube" alt="Video Tutorial"/>
  </a>
</div>

</div>

---

## 📋 Descripción del Proyecto

Este laboratorio demuestra cómo utilizar **Oracle AI Data Platform Workbench** para analizar sentimientos de comentarios en X/Twitter utilizando **OCI AI Language**, almacenar los resultados en tablas Delta y **Oracle Autonomous Database**, y finalmente integrar todo con **Agent Factory** para consultas conversacionales.

### ¿Qué aprenderás?

- Configurar Oracle AI Data Platform Workbench
- Conectarte a la API de X/Twitter para extraer comentarios
- Analizar sentimientos con OCI AI Language (positivo, negativo, neutral)
- Clasificar comentarios por nivel de toxicidad (Hate Speech, Offensive Language, Harassment, Acceptable)
- Crear tablas Delta con clustering optimizado
- Sincronizar datos con Oracle Autonomous Database
- Crear workflows/jobs programados
- Integrar con Agent Factory para consultas en lenguaje natural

---

## 🛠️ Arquitectura del Proyecto

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   X/Twitter     │────>│  OCI Language   │────>│   Spark Delta   │
│   API (Replies) │     │  (Sentimiento)  │     │   Tables        │
└─────────────────┘     └─────────────────┘     └────────┬────────┘
                                                         │
                        ┌─────────────────┐     ┌─────────────────┐
                        │   Autonomous    │<────│   Data Sync     │
                        │   Database      │     │   (MERGE)       │
                        └─────────────────┘     └─────────────────┘
```

---

## 🔧 Prerrequisitos

### 1. Cuenta de Oracle Cloud Infrastructure (OCI)
- Compartment configurado
- Políticas de acceso a los servicios de AI

### 2. Cuenta de Desarrollador en X/Twitter (se asume que ya la tienes)
- Bearer Token, Client ID y Client Secret listos para usar en el notebook

### 3. Oracle Autonomous Database
- Instancia aprovisionada
- Usuario y contraseña configurados
- Wallet descargado (opcional)

---

## 📚 Laboratorio Paso a Paso

> Se asume que ya tienes cuenta de desarrollador en X/Twitter (Bearer Token, Client ID y Client Secret). Si no, créala en [developer.twitter.com](https://developer.twitter.com).

---

### Paso 1: Configurar Compartment y Políticas en OCI

**Step 1**

En la consola de Oracle Cloud, busca `compartments` en el menú lateral y haz clic en la opción `Compartments` de `Identity`.

![img01](./img/img01.png)

**Step 2**

En la consola de Oracle Cloud, dentro de la página de `Compartments`, haz clic en `Create compartment` para crear el compartment.

![img02](./img/img02.png)

**Step 3**

En la consola de Oracle Cloud, en `Create compartment`, completa el formulario con valores como estos: `Name: ora26ai`, `Description: demo` y `Parent compartment: oci_doc_agent (root)`; después haz clic en `Create compartment`.

![img03](./img/img03.png)

**Step 4**

En la consola de Oracle Cloud, busca `Policies` y haz clic en `Policies` dentro de `Identity`.

![img04](./img/img04.png)

**Step 5**

En la consola de Oracle Cloud, dentro de `Policies`, revisa el compartment seleccionado y haz clic en `Create Policy`.

![img05](./img/img05.png)

**Step 6**

En la consola de Oracle Cloud, en `Create Policy`, completa el formulario con valores como estos: `Name: ora26ai`, `Description: demo` y `Compartment: oci_doc_agent (root)`; luego haz clic en `Show manual editor` para escribir la política manualmente.

![img06](./img/img06.png)

**Step 7**

En la consola de Oracle Cloud, con la política ya escrita en el editor manual, revisa el texto y haz clic en `Create`.

Puedes copiar y pegar en el editor manual una política como esta (ajusta el compartment a tu nombre):

```sql
Allow any-user to manage all-resources in compartment ora26ai
```

![img07](./img/img07.png)

---

### Paso 2: Crear Oracle AI Data Platform Workbench

**Step 8**

En la consola de Oracle Cloud, busca `ai data platform` y haz clic en `AI Data Platform Workbench`.

![img08](./img/img08.png)

**Step 9**

En la consola de Oracle Cloud, dentro de `AI Data Platform Workbenches`, si no ves tu compartment, primero recarga la página y luego abre el selector de `Compartment`.

![img09](./img/img09.png)

**Step 10**

En la consola de Oracle Cloud, con el compartment `ora26ai` ya seleccionado, haz clic en `Create AI Data Platform Workbench`.

![img10](./img/img10.png)

**Step 11**

En la consola de Oracle Cloud, en `Create AI Data Platform Workbench`, completa el formulario con valores como estos: `AI Data Platform Workbench name: aidp` y `Workspace name: analytics`; en `Add policies`, en `Choose access level`, selecciona `Advanced - Control access settings at the compartment level`.

![img11](./img/img11.png)

**Step 12**

En la consola de Oracle Cloud, la sección muestra las políticas faltantes; usa el botón `Add` para agregarlas.

![img12](./img/img12.png)

**Step 13**

En la consola de Oracle Cloud, cuando aparezca `Policies added`, verifica ese estado y luego haz clic en `Create`.

![img13](./img/img13.png)

**Step 14**

En la consola de Oracle Cloud, verifica el mensaje de creación exitosa y confirma que el workbench `aidp` quede en estado `Creating`.

![img14](./img/img14.png)

**Step 15**

En la consola de Oracle Cloud, cuando el workbench `aidp` aparezca en estado `Active`, haz clic en su nombre.

![img15](./img/img15.png)

**Step 16**

En la pantalla de acceso de Oracle Cloud, se indica que se envió una notificación MFA al iPhone; en el móvil, el botón importante es `Permitir`, que debes pulsar para entrar.

![img16](./img/img16.png)

---

### Paso 3: Workspace, Cluster y Librerías

Puedes usar el archivo de dependencias [files/requirements.txt](files/requirements.txt) (oci, pandas, requests) para instalar en el cluster.

**Step 17**

En Oracle AI Data Platform Workbench, haz clic en `Workspace` en el menú lateral.

![img17](./img/img17.png)

**Step 18**

En Oracle AI Data Platform Workbench, en la lista de workspaces, haz clic en `analytics`.

![img18](./img/img18.png)

**Step 19**

En Oracle AI Data Platform Workbench, abre el menú `Create` y elige la opción `Cluster`.

![img19](./img/img19.png)

**Step 20**

En Oracle AI Data Platform Workbench, en `Create cluster`, usa valores como estos: `Cluster name: cluster`, `Runtime version: Spark 3.5.0`, `Cluster configuration: Quickstart`, `Run duration: Idle timeout` y `Idle timeout in minutes: 120`; al final, haz clic en `Create`.

![img20](./img/img20.png)

**Step 21**

En Oracle AI Data Platform Workbench, en el menú lateral del workspace, haz clic en `Compute` para revisar los clusters.

![img21](./img/img21.png)

**Step 22**

En Oracle AI Data Platform Workbench, dentro de `Compute`, verifica que el cluster `cluster` aparezca en estado `Active`; luego ábrelo.

![img22](./img/img22.png)

**Step 23**

En Oracle AI Data Platform Workbench, dentro del cluster, abre la pestaña `Library`.

![img23](./img/img23.png)

**Step 24**

En Oracle AI Data Platform Workbench, dentro de `Library`, haz clic en el botón `+` para instalar una librería.

![img24](./img/img24.png)

**Step 25**

En Oracle AI Data Platform Workbench, en `Install Library`, usa `Choose a source: Upload file to workspace`, selecciona [requirements.txt](files/requirements.txt), deja sin marcar `Overwrite files with same name` si no necesitas sobrescribir, y luego pulsa `Install`.

![img25](./img/img25.png)

**Step 26**

En Oracle AI Data Platform Workbench, verifica que [requirements.txt](files/requirements.txt) aparezca con estado `Pending`; eso indica que la instalación sigue en proceso.

![img26](./img/img26.png)

**Step 27**

En Oracle AI Data Platform Workbench, cuando termine, confirma que [requirements.txt](files/requirements.txt) cambie a `Installed`; esa es la validación importante de este paso.

![img27](./img/img27.png)

---

### Paso 4: Autonomous AI Database y Usuario

**Step 28**

En Oracle AI Data Platform Workbench, en la lista de workspaces, identifica `analytics`; ese es el workspace que debes abrir o usar.

![img28](./img/img28.png)

**Step 29**

En la consola de Oracle Cloud, busca `Autonomous AI Database` y haz clic en el resultado del mismo nombre.

![img29](./img/img29.png)

**Step 30**

En la consola de Oracle Cloud, dentro de `Autonomous AI Databases`, verifica que el `Compartment` sea `ora26ai`; si está correcto, haz clic en `Create Autonomous AI Database`.

![img30](./img/img30.png)

**Step 31**

En la consola de Oracle Cloud, en `Create Autonomous AI Database Serverless`, completa el formulario con valores como estos: `Display name: ora26ai_name`, `Database name: ora26ainame`, `Compartment: ora26ai`. En `Workload type`, selecciona `Transaction Processing`. Luego revisa la configuración de la base de datos (versión `26ai`, ECPU, Storage y auto scaling), crea la contraseña del usuario `ADMIN`, en `Network access` selecciona `Secure access from everywhere`, y al final haz clic en `Create`.

![img31](./img/img31.png)

**Step 32**

En la consola de Oracle Cloud, después de crear la base de datos, verifica que el estado quede en `Provisioning`.

![img32](./img/img32.png)

**Step 33**

En la consola de Oracle Cloud, dentro de la ficha de Autonomous Database, abre `Database actions` y haz clic en `SQL`.

![img33](./img/img33.png)

**Step 34**

En Oracle Autonomous Database > Database Actions > SQL Worksheet, pega el script y ejecuta `Run Statement`.

Puedes copiar y ejecutar este SQL en Database Actions (cambia `<tu-contraseña>` por la contraseña del usuario):

```sql
-- 1. Crear usuario
CREATE USER APP_AIDP IDENTIFIED BY "<tu-contraseña>";
GRANT CONNECT, RESOURCE TO APP_AIDP;
GRANT UNLIMITED TABLESPACE TO APP_AIDP;
GRANT CREATE TABLE TO APP_AIDP;
GRANT CREATE VIEW TO APP_AIDP;
GRANT CREATE SEQUENCE TO APP_AIDP;
GRANT CREATE PROCEDURE TO APP_AIDP;
GRANT CREATE SESSION TO APP_AIDP;

-- 2. Crear tabla para almacenar resultados
CREATE TABLE APP_AIDP.INTERACTIONS_ANALYSIS (
    reply_id VARCHAR2(50) PRIMARY KEY,
    username VARCHAR2(100),
    user_name VARCHAR2(200),
    text CLOB,
    category VARCHAR2(50),
    severity_level VARCHAR2(20),
    toxicity_score NUMBER(10,2),
    oci_sentiment VARCHAR2(20),
    oci_negative NUMBER(10,4),
    oci_positive NUMBER(10,4),
    oci_neutral NUMBER(10,4),
    created_at VARCHAR2(50)
);
```

![img34](./img/img34.png)

**Step 35**

En Oracle Autonomous Database > Database Actions > SQL Worksheet, revisa `Script Output` y confirma los mensajes de creación del usuario y de la tabla; esta pantalla sirve para validar que el SQL terminó bien.

![img35](./img/img35.png)

---

### Paso 5: Master catalog y Catálogo externo (Autonomous DB)

**Step 36**

En Oracle AI Data Platform Workbench, entra a `Master catalog` y luego haz clic en `Create catalog`.

![img36](./img/img36.png)

**Step 37**

En Oracle AI Data Platform Workbench, en `Create catalog in Master catalog`, usa valores como estos: `Catalog name: ora26ai`, `Catalog type: External catalog`, `External source type: Oracle Autonomous AI Transaction Processing`, `External source method: Choose ATP instance`, `Region: us-chicago-1`, `Compartment: ora26ai`, `ATP instance: ora26ai_name`, `Service: ora26ainame_high`, `Username: app_aidp`; luego ejecuta `Test connection`, verifica `Connection status: Successful` y haz clic en `Create`.

![img37](./img/img37.png)

**Step 38**

En Oracle AI Data Platform Workbench, en la lista de catálogos, revisa que `ora26ai` aparezca; el estado `Creating` indica que aún se está aprovisionando.

![img38](./img/img38.png)

---

### Paso 6: Configurar OCI SDK y subir archivos al workspace

**Step 39**

En Oracle AI Data Platform Workbench, en el menú lateral, selecciona el workspace `analytics`.

![img39](./img/img39.png)

**Step 40**

En Oracle AI Data Platform Workbench, abre `Create` y elige `Folder` para crear una carpeta dentro del workspace `analytics`.

![img40](./img/img40.png)

**Step 41**

En Oracle AI Data Platform Workbench, escribe `oci` como nombre de la carpeta y luego haz clic en `Create`.

![img41](./img/img41.png)

**Step 42**

En Oracle AI Data Platform Workbench, en el contenido del workspace, verifica que la carpeta `oci` ya exista.

![img42](./img/img42.png)

**Step 43**

En la consola de Oracle Cloud, haz clic en el icono de perfil; el menú desplegado muestra el correo y la cuenta, que son la referencia visual de que abriste el lugar correcto.

![img43](./img/img43.png)

**Step 44**

En la consola de Oracle Cloud, dentro de `My profile`, haz clic en `Tokens and keys`.

![img44](./img/img44.png)

**Step 45**

En la consola de Oracle Cloud, dentro de `API keys`, haz clic en `Add API key`.

![img45](./img/img45.png)

**Step 46**

En la consola de Oracle Cloud, en `Add API key`, selecciona `Generate API key pair`; luego descarga `key.pem` como clave privada y `key_public.pem` como clave pública, y finalmente confirma con `Add`.

![img46](./img/img46.png)

**Step 47**

En la consola de Oracle Cloud, en `Configuration file preview`, copia el contenido del archivo [config](files/oci/config), que incluye valores como `user`, `fingerprint`, `tenancy` y `region`; fíjate especialmente en `key_file=<path to your private keyfile>` porque después tendrás que reemplazarlo por la ruta real de la clave privada.

![img47](./img/img47.png)

**Step 48**

En tu equipo, fuera de Oracle Cloud, verifica que dentro de la carpeta `oci` estén [config](files/oci/config), [key.pem](files/oci/key.pem) y [key_public.pem](files/oci/key_public.pem).

![img48](./img/img48.png)

**Step 49**

En Oracle AI Data Platform Workbench, dentro de la carpeta `oci` del workspace, haz clic en `Upload`.

![img49](./img/img49.png)

**Step 50**

En Oracle AI Data Platform Workbench, en `Upload file in oci`, selecciona [config](files/oci/config), [key.pem](files/oci/key.pem) y [key_public.pem](files/oci/key_public.pem); si no necesitas reemplazar archivos, deja sin marcar `Overwrite files with same name`, y luego haz clic en `Upload`.

![img50](./img/img50.png)

**Step 51**

En Oracle AI Data Platform Workbench, abre el menú contextual de [key.pem](files/oci/key.pem) y haz clic en `Copy path`.

![img51](./img/img51.png)

**Step 52**

En Oracle AI Data Platform Workbench, verifica que en la carpeta `oci` ya aparezcan [config](files/oci/config), [key.pem](files/oci/key.pem) y [key_public.pem](files/oci/key_public.pem).

![img52](./img/img52.png)

**Step 53**

En Oracle AI Data Platform Workbench, abre [config](files/oci/config) y ubica la línea `key_file`; y pegar el Path que se copio en el `Step 51`.

![img53](./img/img53.png)

**Step 54**

En Oracle AI Data Platform Workbench, actualiza [config](files/oci/config) para que `key_file` apunte a `/Workspace/oci/key.pem`; esa línea es la referencia clave del paso.

![img54](./img/img54.png)

**Step 55**

En Oracle AI Data Platform Workbench, después de guardar [config](files/oci/config), verifica la notificación `File saved successfully`.

![img55](./img/img55.png)

**Step 56**

En Oracle AI Data Platform Workbench, seleccionamos `Workspaces > analytics`.

![img56](./img/img56.png)

---

### Paso 7: Crear y ejecutar el Notebook

**Step 57**

En Oracle AI Data Platform Workbench, en `Upload file in analytics`, selecciona [notebook.ipynb](files/notebook.ipynb) para cargarlo dentro del workspace `analytics`.

![img57](./img/img57.png)

**Step 58**

En Oracle AI Data Platform Workbench, verifica que [notebook.ipynb](files/notebook.ipynb) ya aparezca en el contenido del workspace.

![img58](./img/img58.png)

**Step 59**

En Oracle AI Data Platform Workbench, dentro del notebook, abre el menú `Cluster`, elige `Attach existing cluster` y selecciona el cluster `cluster`, que debe estar en estado `Active`.

![img59](./img/img59.png)

**Step 60**

En Oracle AI Data Platform Workbench, mientras se conecta el cluster, revisa el estado `Cluster: cluster Attaching`; ese texto indica que todavía está en proceso.

![img60](./img/img60.png)

**Step 61**

En Oracle AI Data Platform Workbench, cuando el notebook muestre `Cluster: cluster (Active)`, ya puedes continuar; ese estado es la confirmación importante del paso.

![img61](./img/img61.png)

**Step 62**

En Oracle AI Data Platform Workbench, la imagen muestra las primeras celdas ejecutadas correctamente; confirma especialmente la salida con la región `us-chicago-1`.

![img62](./img/img62.png)

**Step 63**

En Oracle AI Data Platform Workbench, la celda escribe en `ora26ai.app_aidp.interactions_analysis`; verifica que ese sea el destino donde se guardarán los datos.

![img63](./img/img63.png)

---

### Paso 8: Verificar datos en Autonomous Database

**Step 64**

En Oracle Autonomous Database > Database Actions, ejecuta `select * from app_aidp.interactions_analysis`; la imagen muestra la consulta y la tabla de resultados como validación final del flujo.

Puedes copiar y ejecutar esta consulta para verificar los datos:

```sql
SELECT * FROM app_aidp.interactions_analysis;
```

![img64](./img/img64.png)

---

#### Estructura del Notebook

Notebook de referencia: [files/notebook.ipynb](files/notebook.ipynb).

| Celda | Descripción |
|-------|-------------|
| **0** | Importa librerías: `oci`, `pandas`, `requests`, `AIServiceLanguageClient` |
| **1** | Función `analyze_sentiment_oci()` con OCI Language |
| **2** | Funciones API X/Twitter: `get_tweet()`, `get_replies()`, `anonymize_text()` |
| **3** | Función `analyze_tweet_replies()` y clasificación por toxicidad |
| **4** | Ejecución del análisis y DataFrame de resultados |
| **5** | Conversión a Spark DataFrame |
| **6** | Catálogo y schema Spark (analysis.tweets) |
| **7** | Tabla Delta con clustering e inserción |
| **8** | Estadísticas por categoría y toxicidad |
| **9** | Lectura desde tabla externa en Autonomous Database |
| **10** | Sincronización con MERGE por `reply_id` |

---

## 📊 Clasificación de Toxicidad

| Categoría | Score Negativo | Nivel de Severidad |
|-----------|----------------|-------------------|
| Hate Speech | > 0.7 | Alto |
| Offensive Language | > 0.5 | Medio |
| Harassment | > 0.3 | Bajo |
| Acceptable | ≤ 0.3 | Bajo |

---

## 📁 Estructura del Repositorio

```
.
├── files/                              # Archivos de referencia para la guía
│   ├── notebook.ipynb                  # Notebook principal del laboratorio (subir al workbench)
│   ├── requirements.txt                # Dependencias (oci, pandas, requests)
│   ├── oracle26ai.txt                 # Notas / referencia
│   └── oci/
│       ├── config                      # Configuración OCI SDK (ejemplo)
│       ├── key.pem                     # Clave privada API Key (ejemplo)
│       └── key_public.pem              # Clave pública API Key (ejemplo)
├── img/                                # Imágenes de la guía (img01.png … img64.png)
└── README.md                           # Este documento
```

---

## 📚 Referencias de Desarrollo

### Oracle AI Data Platform
- [Documentación Oficial AI Data Platform](https://docs.oracle.com/en-us/iaas/data-platform/index.html)

### OCI AI Language
- [OCI AI Language Service](https://docs.oracle.com/en-us/iaas/language/using/overview.htm)
- [Sentiment Analysis API](https://docs.oracle.com/en-us/iaas/api/#/en/language/latest/)

### Oracle Cloud Infrastructure
- [OCI Python SDK](https://github.com/oracle/oci-python-sdk)
- [OCI Config File](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdkconfig.htm)

### Oracle Autonomous Database
- [Conexión con oracledb](https://python-oracledb.readthedocs.io/en/latest/user_guide/connection_handling.html)
- [Delta Tables](https://docs.oracle.com/en-us/iaas/data-platform/using/delta-tables.htm)

### Agent Factory
- [Agent Factory Documentation](https://docs.oracle.com/en-us/iaas/agent-factory/index.html)

### X/Twitter API
- [Twitter API v2](https://developer.twitter.com/en/docs/twitter-api)
- [Search Tweets](https://developer.twitter.com/en/docs/twitter-api/tweets/search/introduction)

---

<!-- MARKDOWN LINKS & IMAGES -->
[issues-shield]: https://img.shields.io/github/issues/othneildrew/Best-README-Template.svg?style=for-the-badge
[issues-url]: https://github.com/jganggini/oracle-ai-data-platform-workbench/issues
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/jgangini/
