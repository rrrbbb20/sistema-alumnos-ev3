# Registro de seguridad · Sistema Alumnos

Estudiante: ____________________   Repositorio: ____________________   Fecha: __________

Completa este registro a medida que avanzas. Cada fila debe poder seguirse hasta un **build de Jenkins**
y un **commit** (usa el identificador `VULN-xx` en el mensaje del commit).

## 1. Revisiones de seguridad realizadas

| # | Etapa del SDLC | Herramienta / técnica | Build | Resultado (resumen) |
|---|---|---|---|---|
| R1 | Código (SAST) | Bandit en el pipeline | #__ | __ hallazgos High / __ Medium / __ Low |
| R2 | Dependencias (SCA) | pip-audit en el pipeline | #__ | __ vulnerabilidades en __ paquetes |
| R3 | Pruebas dinámicas (DAST) | OWASP ZAP sobre staging | #__ | __ High / __ Medium / __ Low |
| R4 | Revisión manual | Lectura de `app.py` | — | |
| R5 | Producción | Grafana + Prometheus | — | |

## 2. Vulnerabilidades identificadas y corregidas

| ID | Vulnerabilidad (CWE) | Detectada por | Ubicación | Riesgo | Corrección aplicada | Commit | Build que lo verifica |
|---|---|---|---|---|---|---|---|
| VULN-01 | | | `app.py:__` | | | | #__ |
| VULN-02 | | | | | | | |
| VULN-03 | | | | | | | |

(Agrega las filas que necesites.)

## 3. Resultados de OWASP ZAP

| Alerta de ZAP | Riesgo | URL de ejemplo | VULN relacionada | Cómo se mitiga | ¿Mitigada? Build |
|---|---|---|---|---|---|
| | | | | | |

## 4. Monitoreo e incidentes

| Hora | Panel / métrica de Grafana | Qué se observó | Entorno | Clasificación | Acción de respuesta | Verificación |
|---|---|---|---|---|---|---|
| | | | | | | |

## 5. Gestión de dependencias

| Paquete | Versión inicial | Vulnerabilidades (IDs) | Versión final | Origen del cambio (PR de Dependabot / manual) | ¿Pruebas, pip-audit y ZAP siguen OK? Build |
|---|---|---|---|---|---|
| Flask | 2.2.2 | | | | |
| Werkzeug | 2.2.2 | | | | |
| Jinja2 | 3.1.2 | | | | |
| gunicorn | 21.2.0 | | | | |
| prometheus-client | 0.17.1 | | | | |

## 6. Trazabilidad

Pega aquí (o adjunta) el `TRAZABILIDAD.md` del último build y completa un recorrido por cada actividad del pipeline:

| Actividad | Hallazgo | Build que lo detectó | Commit / PR que lo resolvió | Build que lo verificó | Versión en producción |
|---|---|---|---|---|---|
| Revisión de seguridad (Bandit) | | #__ | | #__ | |
| Prueba automatizada (OWASP ZAP) | | #__ | | #__ | |
| Gestión de dependencias (Dependabot) | | #__ | PR #__ | #__ | |



# Registro de Seguridad — Sistema Alumnos (OCY1102)

## 1. Revisiones Continuas de Seguridad (SAST, SCA y DAST)
- **SAST (Bandit):** Integrado en la etapa `Security Review (SAST & SCA)` sobre `app.py`. Detectó inicialmente 5 hallazgos (2 High, 2 Medium, 1 Low).
- **SCA (pip-audit):** Integrado en la etapa `Security Review (SAST & SCA)` sobre `requirements.txt`. Detectó 19 CVEs en `Flask 2.2.2`, `Werkzeug 2.2.2`, `Jinja2 3.1.2` y `Gunicorn 21.2.0`.
- **DAST (OWASP ZAP 2.16.1):** Integrado en la etapa `Deploy Staging & DAST (OWASP ZAP)` contra `http://jenkins:5001`. Detectó inicialmente 3 alertas High (SQL Injection, XSS Reflected, XSS DOM Based), 3 Medium (Anti-CSRF, CSP, X-Frame-Options) y 1 Low (X-Content-Type-Options).

## 2. Vulnerabilidades Identificadas y Mitigaciones Aplicadas
| ID | Severidad | Hallazgo | Archivo / Ruta | Mitigación Aplicada | Commit |
|---|---|---|---|---|---|
| VULN-01 | High / Low | Secretos fijos (`SECRET_KEY`) y hash débil MD5 (CWE-259, CWE-327) | `app.py` (`/login`) | Lectura por variables de entorno, `secrets.token_hex(32)` y PBKDF2-SHA256 (`werkzeug.security`) con `secrets.compare_digest`. | `VULN-01` |
| VULN-02 | High / Medium | Inyección SQL y fuga de error SQLite (CWE-89) | `app.py` (`/alumnos/buscar`) | Consulta parametrizada (`LIKE ?`) con tupla de parámetros y mensaje de error genérico HTTP 500. | `VULN-02` |
| VULN-03 | High | Cross-Site Scripting (XSS) reflejado y DOM (CWE-79) | `app.py` (`/hello`) | Sanitización estricta del parámetro `name` mediante `markupsafe.escape()`. | `VULN-03` |
| VULN-04 | Medium / Low | Ausencia de token anti-CSRF y cabeceras HTTP de seguridad (CWE-352, CWE-693) | `app.py` (`/login`, `after_request`) | Token aleatorio por sesión (`csrf_token`) validado con `compare_digest` y cabeceras `CSP`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff` y `Referrer-Policy`. | `VULN-04` |
| VULN-05 | High / Medium | Modo depuración activo (`debug=True`) y bind `0.0.0.0` (CWE-94, CWE-605) | `app.py` (`__main__`) | `debug=False` y enlace por defecto a `127.0.0.1` configurable por `FLASK_HOST`. | `VULN-05` |