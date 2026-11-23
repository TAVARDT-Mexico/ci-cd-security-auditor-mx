# Auditor de Seguridad CI/CD & Detector de Fuga de Secretos | TAVARDT México

[![Licence](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![CI/CD Security](https://img.shields.io/badge/CI%2FCD-Security%20Hardened-00d26a.svg)](README.md)
[![Language](https://img.shields.io/badge/Language-Español-green.svg)](README.md)
[![Engineering](https://img.shields.io/badge/TAVARDT-Cybersecurity-5b3cf5.svg)](https://ag.tavardt.com/mx/ciberseguridad/)

Auditor estático defensivo y linter de seguridad para flujos de integración y despliegue continuo (GitHub Actions). Identifica configuraciones de alto riesgo de supply chain, disparadores vulnerables a pull requests externos, permisos excesivos en GITHUB_TOKEN y fuga de secretos corporativos.

## Aplicación Web Interactiva

Acceda a la herramienta interactiva para auditar manifiestos YAML en tiempo real directamente en el navegador:

- [https://tavardt-mexico.github.io/ci-cd-security-auditor-mx/](https://tavardt-mexico.github.io/ci-cd-security-auditor-mx/)

---

## 1. Visión General del Auditor de Seguridad CI/CD

Las esteiras de automatización e integración continua se han convertido en el objetivo primordial de campañas avanzadas de supply chain. Los atacantes comprometen credenciales de desarrolladores y disparadores mal configurados para ejecutar código arbitrario en los runners y exfiltrar variables de entorno protegidas, tokens de nube (AWS, GCP, Azure) y credenciales de despliegue.

El Auditor de Seguridad CI/CD de TAVARDT analiza manifiestos YAML de forma determinista y pasiva, validando cada instrucción frente a los riesgos del OWASP Top 10 para CI/CD Security y el marco SLSA.

---

## 2. Vulnerabilidades y Vectores Auditados

| Vector de Riesgo | Escenario en el Pipeline CI/CD | Impacto Técnico |
| :--- | :--- | :--- |
| **pull_request_target con Checkout Inseguro** | El flujo ejecuta en el contexto de la rama principal y descarga código del PR externo (`ref: ${{ github.event.pull_request.head.sha }}`). | Ejecución remota de código (RCE) inmediata en el runner con acceso total a los secretos corporativos del repositorio. |
| **Permisos Excesivos write-all** | El pipeline especifica `permissions: write-all` u omite restricciones explícitas de permisos. | El token hereda autorización para alterar etiquetas, sobrescribir releases y publicar artefactos maliciosos. |
| **Inyección de Expresiones Shell en run:** | Variables de contexto de usuarios externos (ej: `${{ github.event.issue.title }}`) se concatenan directamente en scripts inline. | Permite quebra de sintaxis y ejecución de comandos remotos arbitrarios en el sistema operativo del runner. |
| **Acciones Externas Fijadas por Tag Mutable** | Instrucciones `uses:` apuntan a referencias mutables como `@v3` o `@main` en lugar de commit SHAs inmutables. | Expone el pipeline a envenenamiento de supply chain si la cuenta de un mantenedor de la acción es vulnerada. |
| **Exfiltración y Credenciales Estáticas** | Presencia de claves privadas codificadas directamente o comandos `curl` enviando secretos a servidores externos. | Compromiso inmediato de la infraestructura conectada y violación de los estándares de gobernanza y seguridad. |

---

## 3. Guía de Instalación y Ejecución Local (CLI)

El scanner opera en la terminal sin dependencias externas, utilizando exclusivamente la biblioteca estándar de Python.

### Clonar el Repositorio

```bash
git clone https://github.com/TAVARDT-Mexico/ci-cd-security-auditor-mx.git
cd ci-cd-security-auditor-mx
```

### Auditar un Manifiesto YAML Específico

```bash
python scanner.py .github/workflows/deploy.yml
```

### Auditar Todo el Directorio con Salida JSON

```bash
python scanner.py .github/workflows/ --json
```

---

## 4. Blindaje Profesional y Soporte Técnico

Si su auditoría detectó vulnerabilidades en sus pipelines, credenciales expuestas o requiere gobernanza Zero Trust con proveedores cloud:

- **Portal Institucional:** [TAVARDT México](https://ag.tavardt.com/mx/ciberseguridad/)
- **Central de Herramientas:** [Herramientas Defensivas TAVARDT](https://tools.tavardt.com/mx/)
- **Atención Técnica:** [Soporte Técnico y Respuesta a Incidentes](https://ag.tavardt.com/mx/contato/)

---

## Licencia

Distribuido bajo la licencia de código abierto MIT. Consulte el archivo [`LICENSE`](LICENSE) para más detalles.  
Copyright (c) 2026 TAVARDT México.
