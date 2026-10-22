# Política de seguridad

Nos tomamos la seguridad de Hilvana en serio. Si crees haber encontrado una vulnerabilidad:

- Escribe a **hola@hilvana.cloud** con los detalles y los pasos para reproducirla.
- Confirmamos la recepción y trabajamos contigo en la corrección.
- Por favor, no la divulgues públicamente hasta que haya una corrección disponible.

Visión general de los controles de seguridad del producto (arquitectura, aislamiento multiinquilino,
cifrado de credenciales, ejecución de flujos, anti-SSRF, despliegue seguro): **[docs/seguridad.md](docs/seguridad.md)**.

Versiones con soporte de seguridad: la **última** imagen `juankanh/hilvana` publicada.

Cada versión se publica tras `npm audit` sin hallazgos y un análisis Trivy de la imagen; el cloud se
revisa con OWASP ZAP (baseline). Última revisión: 9 de septiembre de 2026.
