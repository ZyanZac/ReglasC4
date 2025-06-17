# Documentación Técnica – CERTISEP

## 1. Propósito del Sistema
CERTISEP es una plataforma web y móvil que permite a universidades con RVOE federal generar certificados y títulos en formato XML conforme a la SEP (MET/MEC), firmarlos electrónicamente y garantizar su validez mediante biometría facial y certificados digitales del SAT (.CER/.KEY).

---

## 2. Usuarios y Roles

| Rol              | Plataforma    | Acciones                                                                 |
|------------------|---------------|--------------------------------------------------------------------------|
| Usuario Operativo | Web           | - Descargar plantilla Excel<br>- Cargar datos<br>- Generar XML<br>- Empaquetar documentos |
| Usuario Firmante | Web y Móvil   | - Todo lo que realiza el usuario operativo<br>- Firmar paquetes o documentos individualmente<br>- Validación biométrica con foto |

- El usuario operativo **no puede firmar**.
- El usuario firmante tiene **todas las capacidades operativas más la firma** electrónica.
- Solo se permite **un dispositivo móvil por usuario firmante**.

---

## 3. Flujo del Sistema

1. Descarga plantilla Excel  
2. Captura datos  
3. Sube a plataforma  
4. Genera XML  
5. Agrupa en paquete  
6. Notifica al usuario firmante  
7. Validación biométrica  
8. Firma  
9. Documentos disponibles

---

## 4. Plantillas y Estructura de Datos

### Títulos

Campos requeridos:  
Matrícula, Nombre, Apellidos, Email, CURP, Servicio Social, Entidad, Fechas (inicio, término, examen, expedición), Modalidad, Carrera, RVOE, Antecedentes, Responsable.

### Certificados

Campos requeridos:  
Matrícula, Nombre, Apellidos, CURP, Género, Fecha de Nacimiento, Campus, Tipo de certificado, Asignaturas, Calificaciones, Responsable.

- Solo se acepta una hoja Excel llamada `"Registros"`.
- Plantillas son personalizadas pero con estructura común.
- **Los valores permitidos dependen de cada institución. CERTISEP no impone catálogos cerrados.**

---

## 5. Validaciones

### General
- Errores se detectan al intentar cargar el XML en el MEC/MET.
- El proceso debe repetirse si el archivo es rechazado.

### Certificados
- CURP válida, matrícula sin "/", email válido.
- Fecha de expedición no mayor a 90 días.
- Asignaturas complementarias (ID 266) se excluyen del XML.
- Validación entre asignaturas cursadas y asignadas.

### Títulos
- Fechas cronológicas (Examen < Expedición, etc.).
- CURP extranjero → entidad "33 extranjero".
- Cédula antecedente obligatoria para posgrado.
- Servicio social obligatorio para licenciatura.

---

## 6. Firma Electrónica

- Requiere: archivos `.CER` y `.KEY` del SAT.
- Entrenamiento biométrico: 3 fotos con gestos.
- Firma biométrica: una foto frontal.
- Si los documentos están empaquetados, **una sola firma es válida para todos**. Si no, se firma uno por uno.
- Firma compatible con certificados X.509 v2/v3.

---

## 7. Arquitectura Técnica

- **Frontend Web**: Vue 3, TypeScript, Bootstrap.
- **Móvil**: NativeScript (JavaScript Vanilla).
- **Backend**: Firebase (Auth, Firestore, Storage, Cloud Functions).
- **Reconocimiento facial**: Amazon Rekognition.

---

## 8. Infraestructura y Seguridad

- Plataforma SaaS en la nube.
- Certificaciones: ISO 27001, 27017, 27018.
- HTTPS, autenticación por Firebase.
- Firmas cifradas en el dispositivo móvil.
- Respaldo diario (3:00 AM) en Firestore.
- Alta disponibilidad, pero sin protección ante errores humanos.

---

## 9. Limitaciones

- Firma por sesión: hasta 400 documentos.
- Android no disponible (deuda técnica).
- Solo la última versión del Excel es guardada.
- Solo se acepta hoja llamada `"Registros"`.
- Sin límite de usuarios por institución (máximo observado: 5).
- No existen pruebas unitarias ni automatizadas.

---

## 10. Automatizaciones

- Triggers en Firebase Functions: notifican al usuario firmante de paquetes XML pendientes.
- Todo el sistema se ejecuta sin backend personalizado (usando funciones de Firebase).

## 11. Modelado Arquitectónico – Estándar C4

CERTISEP utiliza el enfoque de **modelo C4** para documentar visualmente su arquitectura. Este enfoque permite representar el sistema desde diferentes niveles de abstracción, facilitando la comprensión técnica y funcional del mismo.

### 11.1 Nivel de Contexto

El **diagrama de contexto** presenta una visión general del sistema, sus usuarios y sistemas externos involucrados. Para mantener claridad y simplicidad:

- Se representa una única persona: **"Representante del Instituto"**, quien agrupa los roles de **usuario operativo** y **usuario firmante**.
- Las interacciones con sistemas externos como **SEP, SAT, Firebase y Amazon Rekognition** se muestran según los flujos autorizados.
- No se modelan interacciones directas entre el sistema y el SAT, en cumplimiento con las reglas internas.

### 11.2 Nivel de Contenedores

El **diagrama de contenedores** descompone el sistema `CertificadosService` en los siguientes elementos clave:

- **WebApp**: Aplicación web hecha en Vue 3 para usuarios operativos y firmantes.
- **MobileApp**: Aplicación móvil hecha en NativeScript, usada exclusivamente por firmantes para validar biometría y firmar.
- **Backend**: Ejecutado completamente mediante **Firebase Functions**, que orquesta la lógica del sistema.
- **Database**: Almacenamiento en **Firebase Firestore** para plantillas, usuarios, XML y firmas.

Los contenedores interactúan con sistemas externos conforme a flujos definidos y respetando clarificaciones del modelo C4 (e.g., verificación biométrica, lineamientos educativos, autenticación).

### 11.3 Nivel de Componentes

Se adoptó un estándar interno para la descomposición de componentes dentro de los contenedores, siguiendo las mejores prácticas del modelo C4 y compatible con Structurizr DSL:

- El backend se descompone en componentes especializados como `CertController`, `XMLGenerator`, `ValidationService`, `BioValidator`, `NotificationService`, etc.
- Cada componente tiene una **responsabilidad única** y relaciones claras con otros elementos del sistema o servicios externos.
- Este nivel de detalle se adopta como **estándar institucional para futuros proyectos**, asegurando trazabilidad, mantenibilidad y cumplimiento con reglas de modelado formales.

Los componentes del **WebApp** y **MobileApp** también se modelan por separado, destacando elementos como `ExcelUploader`, `XMLViewer`, `SignerDashboard`, `BiometricCapture` y `SignatureExecutor`.

### 11.4 Notas de modelado

- Se sigue el estilo y reglas del archivo `ReglasTodo.json`, con validación de nombres, tags, verbos y grupos.
- El motor de modelado utilizado es **Structurizr DSL**.
- Se emplea `autoLayout lr` para mantener consistencia visual entre vistas, sin intervención manual en posicionamiento.