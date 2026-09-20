# 🛡️ Wazuh Docker - SIEM + XDR Autohospedado

[![GitHub](https://img.shields.io/badge/GitHub-wazuh--docker-blue?logo=github)](https://github.com/JLalib/wazuh-docker)
[![Docker](https://img.shields.io/badge/Docker-wazuh%2Fwazuh-blue?logo=docker)](https://hub.docker.com/r/wazuh/wazuh)
[![License](https://img.shields.io/badge/License-AGPL--3.0-orange)](https://www.gnu.org/licenses/agpl-3.0.html)

## 📋 Descripción general

**Wazuh en Docker** es una solución de seguridad completamente autohospedada y open source que proporciona **SIEM (Security Information and Event Management) + XDR (Extended Detection and Response)** integrado. Incluye agentes multi-plataforma (Linux, Windows, macOS), análisis de logs centralizado, file integrity monitoring (FIM), intrusion detection (IDS), detección de anomalías, malware y rootkits, escaneo de vulnerabilidades, monitorización de cumplimiento normativo (HIPAA, PCI-DSS, GDPR, NIST), dashboards Kibana intuitivos, API REST completa, integraciones webhook, respuesta activa automatizada, reglas de detección personalizadas, workflows de respuesta a incidentes, escalable a miles de endpoints, 100% privado sin vendor lock-in.

## ✨ Características principales

- **Log analysis centralizado**: Recopila logs de múltiples fuentes, correlación basada en reglas, alerting en tiempo real
- **File Integrity Monitoring (FIM)**: Detecta cambios en archivos críticos, monitorea permisos, alertas automáticas
- **Intrusion Detection (IDS)**: Detección de intrusiones, network anomaly detection, threat signatures
- **Malware detection**: Detección por comportamiento de malware, rootkit detection, identificación de PUP
- **Vulnerability scanning**: CVE scanning automático, inventario de paquetes, tracking de remediación
- **Compliance reporting**: HIPAA, PCI-DSS, GDPR, NIST 800-53, TSC, CIS compliance reporting
- **Kibana dashboards**: Visualización intuitiva, dashboards personalizados, métricas en tiempo real
- **REST API**: API completa, documentación OpenAPI, integraciones personalizadas, acceso programático
- **Webhook integrations**: Slack, Teams, email alerting, webhooks personalizados, automatización event-driven
- **Active response**: Respuesta a incidentes automatizada, ejecución de scripts, automatización de reglas de firewall
- **Threat intelligence**: Integración VirusTotal, threat feeds, IOC ingestion y alerting
- **Multi-platform agents**: Linux, Windows, macOS, Amazon Linux, CentOS, Debian, Ubuntu, etc.

## 📋 Requisitos del sistema

- **Docker & Docker Compose v2+**
- **RAM**: 2 GB - 8 GB mínimo (recomendado 4GB+ para producción)
- **Espacio en disco**: 20 GB - 200GB+ (según volumen de logs)
- **Puertos TCP**: 443 (UI/API), 1514-1515 (agents), 514 (syslog)
- **Elasticsearch storage**: SQLite o Elasticsearch remoto
- **CPU**: 2 cores mínimo (4+ recomendado para producción)
- **Red**: Acceso outbound para threat intelligence feeds
- **Opcional**: Reverse proxy nginx/Caddy para HTTPS

> ⚠️ **Resource-intensive**: Wazuh consume más recursos que alternativas ligeras. Mínimo 2-4 GB RAM. Monitorea consumo.
> 💡 **Scalability**: Para grandes deployments (1000+ agents), considera multi-node Elasticsearch + Wazuh cluster.

## 🐳 Instalación

### Paso 1: docker-compose.yml (completo con Elasticsearch)

```yaml
version: '3.8'
services:
  wazuh:
    image: wazuh/wazuh:latest
    container_name: wazuh
    restart: unless-stopped
    ports:
      - "443:443"
      - "1514:1514"
      - "1515:1515"
      - "514:514/udp"
    environment:
      - INDEXER_URL=https://wazuh.indexer:9200
      - INDEXER_USERNAME=admin
      - INDEXER_PASSWORD=SecurePass123!
      - FILEBEAT_SSL_VERIFICATION_MODE=full
    volumes:
      - wazuh_api_cert:/etc/certs/wazuh
      - wazuh_etc:/var/ossec/etc
      - wazuh_var:/var/ossec/var
    depends_on:
      - wazuh.indexer
      - wazuh.dashboard

  wazuh.indexer:
    image: wazuh/wazuh-indexer:latest
    container_name: wazuh.indexer
    restart: unless-stopped
    environment:
      - OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g
    volumes:
      - wazuh_indexer_data:/usr/share/wazuh-indexer/data

  wazuh.dashboard:
    image: wazuh/wazuh-dashboard:latest
    container_name: wazuh.dashboard
    restart: unless-stopped
    ports:
      - "5601:5601"
    environment:
      - INDEXER_URL=https://wazuh.indexer:9200
    depends_on:
      - wazuh.indexer

volumes:
  wazuh_api_cert:
  wazuh_etc:
  wazuh_var:
  wazuh_indexer_data:
```

### Paso 2: Iniciar Wazuh

```bash
docker compose up -d
# Espera ~60 segundos para que inicie (Elasticsearch indexing)
docker compose logs -f wazuh
```

### Paso 3: Acceder a Wazuh

| Servicio | URL |
|----------|-----|
| 🛡️ **Wazuh Web UI** | `https://localhost:443` |
| 📊 **Kibana Dashboard** | `https://localhost:5601` |
| 📡 **API REST** | `https://localhost:443/api/v1` |

**Credenciales por defecto**: `admin` / `SecureSecurePass123!` **(¡cambiar inmediatamente!)**

## ⚙️ Configuración

1. **Cambiar contraseña por defecto**: Primer login → cambiar password de admin
2. **Configurar HTTPS**: Certificado autofirmado por defecto; usar reverse proxy (nginx/Caddy) para certificado válido
3. **Ajustar recursos Elasticsearch**: Modificar `OPENSEARCH_JAVA_OPTS` según RAM disponible (`-Xms2g -Xmx2g` para 4GB+)
4. **Configurar puertos**: Cambiar mapeo de puertos si hay conflictos (ej: `- "8443:443"`)
5. **Persistencia de datos**: Volúmenes Docker nombrados (`wazuh_etc`, `wazuh_var`, `wazuh_indexer_data`)
6. **Agent enrollment**: Configurar `WAZUH_MANAGER` IP en agentes para registro automático
7. **Webhook integrations**: Management → Configuration → Integrations → Slack/Teams/Custom webhook
8. **Alert threshold tuning**: Management → Groups → Rules → Set alert level threshold (1-15)

## 🚀 Primeros pasos

1. Abre `https://localhost:443` en tu navegador
2. Login con `admin` / `SecureSecurePass123!` **(cambiar credencial inmediatamente)**
3. El dashboard principal carga con overview de seguridad
4. Agrega agents en **"Add agent"** para monitorear endpoints
5. Agents instalados en Linux/Windows comienzan a reportar logs automáticamente
6. Kibana dashboard muestra análisis de logs en tiempo real
7. Configura rules, alerts, webhooks según necesidad

> ⚠️ **Cambiar contraseña por defecto**: Primero login, cambia admin password. HTTPS con certificado autofirmado.
> 💡 **Agent deployment**: Windows: MSI installer. Linux: RPM/DEB packages. macOS: DMG installer. Deploy vía Ansible/Puppet para escala.

## 💡 Casos de uso

- **Centralized security monitoring**: Monitorea 100+ endpoints centralizadamente, threat detection automático
- **Compliance monitoring**: HIPAA, PCI-DSS, GDPR, NIST compliance reporting automático
- **Incident response**: Detección rápida de incidentes, automated active response, forensics
- **Threat intelligence**: VirusTotal integration, threat feeds, IOC matching, malware alerts
- **File integrity monitoring**: Detecta cambios no autorizados en archivos críticos, regulatory requirement
- **Network security monitoring**: Firewall logs analysis, IDS/IPS integration, network anomaly detection
- **Enterprise self-hosted**: Full data control, no vendor lock-in, customizable, open source

## 🔒 Acceso remoto seguro

Para exponer Wazuh de forma segura a Internet:

1. **Reverse Proxy** (nginx/Caddy) con certificado TLS válido (Let's Encrypt)
2. **Autenticación**: Configurar LDAP, Active Directory, SAML o API keys
3. **Firewall**: Restringir puertos 1514/1515/514 solo a IPs de agentes conocidos
4. **VPN/Tailscale**: Acceso solo vía VPN para administración
5. **Rate limiting**: Configurar en reverse proxy para API endpoints

## 🛠️ Gestión y mantenimiento

### Ver estado
```bash
docker compose ps
```

### Ver logs
```bash
# Logs del manager
docker compose logs -f wazuh

# Logs de Elasticsearch/Indexer
docker compose logs -f wazuh.indexer
```

### Detener Wazuh
```bash
docker compose down
```

### Actualizar versión
```bash
docker compose pull
docker compose up -d
# Datos persistentes en volúmenes Docker
```

### Backup de datos
```bash
# Snapshot Elasticsearch
docker compose exec wazuh.indexer curl -XGET "https://localhost:9200/_snapshot" -u admin:SecurePass123!

# O backup simple de volúmenes
docker compose down
tar -czf wazuh_backup.tar.gz wazuh_* 2>/dev/null || true
```

### Monitorear consumo
```bash
docker stats wazuh wazuh.indexer wazuh.dashboard
# Típicamente:
# Wazuh CPU: 5-20% (según volumen logs)
# Wazuh RAM: 500MB-2GB
# Indexer RAM: 1-4GB (Elasticsearch memory)
```

## 📝 Licencia

**AGPL-3.0** - Open source (community edition)

- Código fuente: [Wazuh GitHub Repository](https://github.com/wazuh/wazuh)
- Imágenes Docker: [Docker Hub - Wazuh official images](https://hub.docker.com/r/wazuh/wazuh)
- Documentación oficial: [Official Documentation](https://documentation.wazuh.com/)

---

> 📖 **Basado en el tutorial**: [Cómo instalar Wazuh en Docker - Security Monitoring SIEM autohospedado](https://genbyte.blogspot.com/2026/09/como-instalar-wazuh-en-docker-security.html)  
> 🎥 **Vídeo tutorial**: [Canal GENBYTE en YouTube](https://www.youtube.com/@genbyte)  
> ☕ **Apoya el proyecto**: [Ko-fi](https://ko-fi.com/genbyte) | [Newsletter](https://genbyte.blogspot.com/newsletter)