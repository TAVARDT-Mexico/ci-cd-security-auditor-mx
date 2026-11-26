<p align="center">
  <a href="https://ag.tavardt.com/mx/">
    <img src="https://ag.tavardt.com/img/tavardt-banner.png" alt="TAVARDT" width="1024" />
  </a>
</p>

# Auditor de Seguridad CI/CD y Hardening de la Cadena de Suministro de Software

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-5b3cf5.svg)](https://tavardt-mexico.github.io/ci-cd-security-auditor-mx/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Standards](https://img.shields.io/badge/Standards-CIS%20%7C%20OWASP%20CI%2FCD-blue.svg)](https://tavardt-mexico.github.io/ci-cd-security-auditor-mx/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Guía corporativa de ingeniería defensiva, estándares de inmunización para canales de automatización y utilidad determinista de auditoría CLI para la protección de la cadena de suministro en GitHub Actions y plataformas modernas de CI/CD.

***

## Panorama Ejecutivo: El Escenario de Amenazas en CI/CD

Los canales de Integración Continua y Despliegue Continuo (CI/CD) constituyen el núcleo de la ingeniería de software actual. Al poseer acceso a credenciales de despliegue, permisos de escritura en repositorios y conexiones a la red corporativa, las automatizaciones se han convertido en el vector predilecto de ataque para vulnerar la cadena de suministro de software.

Cuando los equipos priorizan la velocidad de entrega sin controles de seguridad perimetral, los flujos de trabajo heredan vulnerabilidades arquitectónicas graves:
* Disparadores de ejecución compilan código no verificado procedente de forks dentro de entornos privilegiados.
* Dependencias y actions externas emplean etiquetas mutables expuestas al secuestro de cuentas.
* Los tokens de ejecución heredan permisos globales de escritura sobre el código fuente y artefactos.
* Los ejecutores (runners) carecen de controles de red en el tráfico de salida, posibilitando la exfiltración de secretos.

Este repositorio define un estándar corporativo de defensa en profundidad. Combina configuraciones validadas para producción con un analizador estático determinista en Python (`audit_ci_cd_security.py`) diseñado para operar sin telemetría externa.

***

## Superficies Críticas de Ataque en Flujos de Automatización

### 1. El Riesgo Crítico del Disparador `pull_request_target`
Los flujos activados por `pull_request_target` se ejecutan en el contexto de la rama base del repositorio, accediendo directamente a secretos protegidos:
* **Mecanismo de la Falla:** Cuando el flujo descarga el código del pull request (`github.event.pull_request.head.sha`), ejecuta código suministrado por terceros en el entorno privilegiado.
* **Impacto Operativo:** Colaboradores externos pueden alterar scripts de compilación, ejecutar comandos arbitrarios en el runner y sustraer credenciales de producción sin autorización previa.
* **Directriz Defensiva:** Nunca descargue código no confiable bajo el evento `pull_request_target`. Isole las rutinas de validación en eventos ordinarios de `pull_request`.

### 2. Dependencias y Actions Externas sin Fijación por Commit SHA
Invocar actions de terceros mediante etiquetas de versión (como `@v3` o `@main`) introduce riesgos directos de suplantación:
* **Mecanismo de la Falla:** Las etiquetas en Git son punteros mutables que pueden modificarse si la cuenta del desarrollador original es vulnerada.
* **Impacto Operativo:** Un commit malicioso introducido en una action popular infecta automáticamente miles de pipelines corporativos de forma simultánea.
* **Directriz Defensiva:** Fije todas las actions externas obligatoriamente mediante el hash SHA inmutable de 40 caracteres, anotando la versión correspondiente como comentario.

### 3. Tokens con Permisos Excesivos (`permissions: write-all`)
Las automatizaciones operan frecuentemente con permisos globales heredados de la configuración general de la organización:
* **Mecanismo de la Falla:** La omisión de un bloque declarativo `permissions:` confiere al token acceso de escritura sobre el repositorio, paquetes y lanzamientos.
* **Impacto Operativo:** Una dependencia comprometida durante el build puede alterar el código fuente en ramas protegidas o publicar paquetes adulterados.
* **Directriz Defensiva:** Declare explícitamente `permissions: contents: read` en el encabezado del archivo, limitando privilegios de escritura únicamente a jobs aislados que lo requieran.

### 4. Conectividad de Salida No Restringida en Ejecutores
Los runners de compilación suelen disponer de acceso irrestrito a internet:
* **Mecanismo de la Falla:** Scripts maliciosos pueden abrir conexiones reversas y transmitir variables de entorno hacia servidores remotos no autorizados.
* **Directriz Defensiva:** Implemente filtrado de tráfico de salida (egress filtering) en el runner, bloqueando toda comunicación no dirigida a registros oficiales.

***

## Modelos de Mitigación Defensiva en Producción

El repositorio proporciona plantillas técnicas validadas para su despliegue en entornos de producción:

### 1. Flujo Protegido para GitHub Actions (`hardened-workflow.yml`)
Aplica el principio de mínimo privilegio, fijación de hashes SHA y aislamiento de secretos:

```yaml
name: Enterprise Hardened Pipeline

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  audit-and-build:
    runs-on: ubuntu-latest
    steps:
      - name: Hardened Checkout
        uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
        with:
          persist-credentials: false

      - name: Inicializacion Segura de Python
        uses: actions/setup-python@0a5c61591373683505ea898e09a3ea4f39ef2b9c # v5.0.0
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Ejecucion de Analisis Estatico
        run: |
          python audit_ci_cd_security.py .github/workflows/
```

### 2. Script de Verificación de Fijación de Actions (`action-pinning.sh`)
Inspecciona los manifiestos de automatización y verifica el uso de hashes inmutables:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Revisa los flujos de trabajo y comprueba la fijacion por SHA de 40 caracteres
find .github/workflows -type f \( -name "*.yml" -o -name "*.yaml" \) | while read -r workflow; do
  echo "[*] Auditando workflow: ${workflow}"
  grep -E "uses: [a-zA-Z0-9_\-\.\/]+@[a-f0-9]{40}" "${workflow}" || echo "[!] Actions no fijadas detectadas"
done
```

### 3. Filtrado de Red de Salida del Runner (`step-security-harden.yml`)
Bloquea conexiones no autorizadas originadas en el entorno de ejecución:

```yaml
- name: Proteger Red de Salida
  uses: step-security/harden-runner@63c24ba6bd7ba022e95695ff85de572c04a18142 # v2.7.0
  with:
    egress-policy: block
    allowed-endpoints: >
      github.com:443
      api.github.com:443
      pypi.org:443
      files.pythonhosted.org:443
```

***

## Herramienta Oficial de Auditoría CLI

Los entornos de navegador web aplican restricciones de aislamiento que impiden acceder al sistema de archivos local y analizar directorios de configuración. Con el fin de proporcionar diagnósticos reproducibles y sin telemetría externa, TAVARDT desarrolló un analizador estático en Python diseñado para terminales locales y tuberías de CI/CD.

### Fases del Análisis Estático

La utilidad (`audit_ci_cd_security.py`) ejecuta cuatro comprobaciones deterministas:
* **Auditoría de Disparadores:** Examina la definición de eventos para señalar combinaciones inseguras de `pull_request_target`.
* **Verificación de Mínimo Privilegio:** Identifica directivas `write-all` o la ausencia de bloques restrictivos de permisos.
* **Inspección de Dependencias:** Valida las declaraciones `uses:` mediante expresiones regulares para verificar hashes SHA de 40 caracteres.
* **Escaneo de Secretos:** Analiza bloques de comandos en terminal y variables para detectar credenciales expuestas en texto plano.

### Instrucciones de Ejecución

Descargue y ejecute la herramienta directamente contra los manifiestos del repositorio:

```bash
# 1. Obtener el script oficial
curl -sO https://raw.githubusercontent.com/TAVARDT/ci-cd-security-auditor-mx/ci-cd-security-auditor-mx/main/audit_ci_cd_security.py

# 2. Ejecutar analisis estatico contra workflows locales
python audit_ci_cd_security.py .github/workflows/
```

***

## Preguntas Frecuentes sobre Arquitectura (FAQ Técnico)

### 1. ¿Por qué el evento pull_request_target se considera una de las mayores vulnerabilidades en GitHub Actions?
El evento `pull_request_target` se ejecuta en el contexto privilegiado de la rama base con acceso directo a secretos y permisos de escritura. Si el flujo descarga el código enviado en un pull request desde un fork externo, atacantes no autorizados pueden ejecutar código arbitrario y exfiltrar credenciales críticas.

### 2. ¿Cómo previene la fijación por commit SHA de 40 caracteres los ataques a la cadena de suministro?
Las etiquetas como `@v3` o `@main` son referencias mutables en Git que pueden reasignarse si la cuenta del autor es comprometida. Un hash commit SHA de 40 caracteres es criptográficamente inmutable, asegurando que únicamente se ejecute el código previamente auditado.

### 3. ¿Qué impacto operativo produce omitir el bloque de permisos en los manifiestos de automatización?
Al omitir la declaración explícita de `permissions:`, el token del runner hereda los privilegios configurados a nivel de organización, que comúnmente incluyen permisos amplios de escritura. Un componente infectado puede aprovechar estos privilegios para adulterar versiones o modificar ramas protegidas.

### 4. ¿Cómo impide el filtrado de red de salida (egress filtering) la exfiltración de credenciales?
El filtrado de red restringe las conexiones salientes del runner únicamente a dominios oficiales previamente autorizados. Si una dependencia maliciosa intenta establecer una shell reversa o transmitir secretos hacia servidores de comando externos, la conexión es descartada de inmediato a nivel de red.

### 5. ¿La herramienta determinista en Python requiere conexión a internet o permisos de administrador?
No. La utilidad opera de manera completamente pasiva y fuera de línea, leyendo los archivos locales YAML directamente desde el disco sin enviar telemetria y sin requerir privilegios de superusuario.

***

## Marcos de Cumplimiento y Estándares Empresariales

Arquitectura defensiva y controles de seguridad alineados con la Ley Federal de Proteccion de Datos Personales en Posesion de los Particulares (LFPDPPP) y las guias de ciberseguridad del INAI.

***

## Enlaces Útiles

* **Portal Corporativo:** https://ag.tavardt.com/mx/
* **Suite de Diagnósticos:** https://tools.tavardt.com/mx/
* **Contacto Técnico:** https://ag.tavardt.com/mx/contacto/

***

© 2026 TAVARDT. Todos los derechos reservados.
