# Cumplimiento de la Normativa HIPAA

Si el proyecto manipula **información de salud protegida (PHI, por sus siglas en inglés)**, debe cumplir con la **Ley de Portabilidad y Responsabilidad de Seguros de Salud** de EE. UU. (HIPAA). Esta guía resume consideraciones importantes para el desarrollo y operación de aplicaciones que se ajustan a HIPAA.

## 1. ¿Qué es PHI?

PHI incluye cualquier dato individual sobre salud o atención médica, historial médico, resultados de análisis, diagnósticos, datos demográficos o cualquier información que pueda utilizarse para identificar a un paciente. El manejo de PHI está regulado por las reglas de Privacidad y Seguridad de HIPAA.

## 2. Cifrado y seguridad de datos

- **En tránsito**: utilice siempre HTTPS/TLS para proteger los datos que circulan entre clientes, servidores y bases de datos. Configure certificados SSL válidos (por ejemplo, mediante Google Managed Certificates en Cloud Run).
- **En reposo**: cifre todas las bases de datos y copias de seguridad. En Cloud SQL, active el cifrado automático de Google o utilice claves gestionadas (Customer‑Managed Encryption Keys) cuando sea necesario.
- **Almacenamiento de secretos**: use **Secret Manager** para guardar contraseñas, claves JWT y cualquier secreto. No almacene claves o tokens en el código fuente.
- **Logs**: no registre PHI ni datos sensibles. Enmascare o excluya campos como nombres, identificadores y resultados médicos de los logs. Verifique que el `metadata` de los logs no incluya información de salud.

## 3. Control de acceso y autenticación

- **Autenticación robusta**: utilice JWT con expiración limitada, autenticación multifactor (MFA) y/o proveedores de identidad federada (por ejemplo, Auth0, Okta, Cloud Identity).
- **Autorización basada en roles**: implemente un control de acceso que asigne permisos según roles (p. ej., paciente, médico, administrador). Solo los usuarios con permisos adecuados deben acceder a PHI.
- **Principio de privilegio mínimo**: las cuentas de servicio y usuarios humanos deben tener únicamente los permisos necesarios. Revise periódicamente los roles asignados en GCP (IAM).

## 4. Auditoría y monitoreo

- **Logs de auditoría**: registre eventos de seguridad y accesos a PHI en sistemas de auditoría inmutables (Cloud Audit Logs). Incluya quién accedió, qué datos consultó y cuándo. Conserve estos logs según los requerimientos legales.
- **Alertas y detección de intrusiones**: configure alertas en Cloud Monitoring para detectar accesos inusuales o picos de error. Integre herramientas de detección de amenazas si es necesario.
- **Revisión periódica**: audite regularmente la configuración de seguridad y los logs para identificar posibles brechas de cumplimiento.

## 5. Gestión de datos y retención

- **Principio de minimización de datos**: almacene solo la información necesaria para cumplir con el propósito del sistema. Evite retener PHI más allá del tiempo requerido.
- **Eliminación segura**: cuando se deba borrar PHI, realice borrados seguros en bases de datos y copias de seguridad. Para Cloud SQL, utilice procesos de eliminación que garanticen que los datos ya no son accesibles.
- **Transferencia de datos**: si la PHI se exporta o transfiere a otros sistemas, asegúrese de que los canales estén cifrados y de que el destinatario cumpla con HIPAA.

## 6. Contratos y acuerdos

Si utiliza servicios en la nube o proveedores externos para procesar o almacenar PHI, asegúrese de firmar un **Business Associate Agreement (BAA)** con esos proveedores. Google Cloud ofrece un BAA que cubre servicios como Cloud Run, Cloud SQL, Cloud Storage y Pub/Sub.

## 7. Desarrollo seguro

- **Análisis estático**: utilice herramientas como SonarQube para detectar vulnerabilidades (inyecciones SQL, XSS). Aplique parches y actualizaciones de dependencias.
- **Revisiones de código**: establezca una política de revisión para cambios que afecten funciones relacionadas con PHI.
- **Pruebas de seguridad**: además de pruebas funcionales, realice pruebas de penetración y escaneo de vulnerabilidades.

## 8. Documentación y entrenamiento

- Proporcione documentación clara al equipo sobre qué se considera PHI y cómo manejarla.  
- Entrene a los desarrolladores y operadores en prácticas seguras y en los requisitos de HIPAA.  
- Mantenga un plan de respuesta a incidentes en caso de fuga de datos o violación de seguridad.

> **Nota:** Esta guía es un resumen y no sustituye una consulta legal. Para asegurar el cumplimiento pleno de HIPAA, se recomienda trabajar con profesionales de seguridad y asesores legales especializados.