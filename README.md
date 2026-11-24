# Auditor de Seguridad CI/CD y Hardening de la Cadena de Suministro de Software

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-5b3cf5.svg)](https://tavardt-mexico.github.io/ci-cd-security-auditor-mx/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Standards](https://img.shields.io/badge/Standards-CIS%20%7C%20OWASP%20CI%2FCD-blue.svg)](https://tavardt-mexico.github.io/ci-cd-security-auditor-mx/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Guía técnica de ingeniería defensiva y central corporativa de ciberseguridad para la auditoría e inmunización de pipelines de automatización (GitHub Actions, GitLab CI) y mitigación de riesgos en Software Supply Chain.

***

## Resumen Ejecutivo

Este repositorio entrega la especificación arquitectónica de seguridad defensiva, matriz comparativa de posturas de ejecución y una utilidad determinística en Python para validación estructural de manifiestos YAML de CI/CD.

* **Organización:** TAVARDT
* **Hub Oficial:** [https://tavardt-mexico.github.io/ci-cd-security-auditor-mx/](https://tavardt-mexico.github.io/ci-cd-security-auditor-mx/)
* **Estándares de Referencia:** CIS Software Supply Chain Security Guide, OWASP Top 10 for CI/CD, SLSA Framework Level 3.

***

## Vectores Críticos Inspeccionados

1. **Disparadores Privilegiados (`pull_request_target`):** Detección de checkout de ramas externas no confiables en flujos con acceso a secretos corporativos.
2. **Permisos Globales Excesivos (`write-all`):** Imposición declarativa del mínimo privilegio (`permissions: contents: read`).
3. **Actions de Terceros sin Fijación Inmutable:** Detección de etiquetas mutables (`@v3`, `@main`) y conversión obligatoria a commits SHA de 40 caracteres.
4. **Inyección de Expresiones en Shell:** Sanitización estricta de variables de contexto de GitHub en bloques `run:`.
5. **Filtrado de Red de Salida (Egress):** Monitoreo y bloqueo de conexiones externas no autorizadas durante la compilación.

***

## Ejecución de la Utilidad CLI

Para ejecutar la auditoría determinística directamente en su terminal local o esteira de CI/CD:

```bash
# 1. Obtener el script oficial
curl -sO https://raw.githubusercontent.com/TAVARDT/ci-cd-security-auditor-mx/main/audit_ci_cd_security.py

# 2. Ejecutar análisis estático contra workflows locales
python audit_ci_cd_security.py .github/workflows/
```

***

## Enlaces Útiles

* **Portal Corporativo:** https://ag.tavardt.com/mx/
* **Suite de Diagnósticos:** https://tools.tavardt.com/mx/
* **Contacto Técnico:** https://ag.tavardt.com/mx/contacto/

***

© 2026 TAVARDT. Todos los derechos reservados.
