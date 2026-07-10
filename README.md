# Laboratorio FortiGate 7 – Configuración GUI en PNETLab


---

## 📖 Introducción

En este laboratorio se implementó un firewall **FortiGate** utilizando **PNETLab**, con el propósito de configurar políticas de seguridad, controlar el tráfico entre redes, aplicar perfiles de seguridad y administrar el acceso a los servicios, todo a través de la interfaz gráfica (GUI).

---

## 🎯 Objetivos

### Objetivo general
Configurar un firewall FortiGate completamente desde la interfaz gráfica para implementar servicios de red y políticas de seguridad.

### Objetivos específicos
- Toda la configuración debe realizarse por la GUI
- Acceso a Internet
- LAN de usuarios (`/25`)
- LAN de servidores (`/28`)
- Configuración de IP en interfaces
- DHCP en la LAN de usuarios
- Ruta por defecto
- NAT
- Permitir únicamente tráfico HTTP desde la LAN de usuarios hacia la LAN de servidores, bloqueando todo lo demás
- Bloquear redes sociales
- Bloquear llamadas por WhatsApp
- Bloquear dominios y sub-dominios de `itla.edu.do`
- Detectar y bloquear escaneos de red
- Aplicar WAF al servidor Web

---

## 🗺️ Topología de la red

```
![image](https://github.com/thesixcay/Fortigate-7/blob/671e90907615a82851b1931821a26ef02d680b2a/screenshots/Screenshot%202026-07-10%20161641.png)
```

| Dispositivo   | Interfaz | Rol             |
|---------------|----------|-----------------|
| FortiGate     | port1    | WAN (Internet)  |
| FortiGate     | port2    | LAN_USUARIOS    |
| FortiGate     | port3    | LAN_SERVIDORES  |
| sw2           | e0/0-e0/1| Switch usuarios |
| sw3           | e0/0-e0/1| Switch servidores|
| Win 7         | e0       | Cliente         |
| Winserver     | e0       | Servidor Web    |

---

## ⚙️ Configuración de interfaces

Se configuraron las interfaces del FortiGate para separar la red de usuarios, la red de servidores y la conexión hacia Internet.

| Interfaz             | Alias           | IP/Netmask                  | Modo       | DHCP Server                  |
|-----------------------|-----------------|------------------------------|------------|-------------------------------|
| port1 (WAN)           | WAN             | Obtenida por DHCP (192.168.111.0/24) | DHCP | — |
| port2 (LAN_USUARIOS)  | LAN_USUARIOS    | 23.6.1.1/255.255.255.128 (/25) | Manual   | Rango 23.6.1.10 – 23.6.1.120 |
| port3 (LAN_SERVIDORES)| LAN_SERVIDORES  | 23.6.1.129/255.255.255.240 (/28) | Manual | — |

**DHCP en LAN de usuarios**
- Rango: `23.6.1.10 – 23.6.1.120`
- Máscara: `255.255.255.128`
- Gateway: mismo que la interfaz
- DNS: `8.8.8.8` / `1.1.1.1`
- Lease time: `604800` segundos

---

## 🌐 Acceso a Internet

La interfaz WAN (port1) fue configurada en modo DHCP, obteniendo conectividad hacia Internet, verificada mediante acceso a sitios externos desde la máquina cliente (Win 7).

---

## 🔒 Políticas de firewall

### Tráfico HTTP permitido (usuarios → servidores)
- **Nombre:** `HTTP_to_Server`
- **Origen:** LAN_USUARIOS (port2)
- **Destino:** LAN_SERVIDORES (port3) — WEB_SERVER
- **Servicio:** HTTP
- **Acción:** ACCEPT
- **Perfiles de seguridad:** WAF_WEB, certificate-inspection

### Bloqueo del resto del tráfico
- **Nombre:** `Block_to_Server`
- **Origen:** LAN_USUARIOS
- **Destino:** LAN_SERVIDORES
- **Servicio:** ALL
- **Acción:** DENY

---

## 🚫 Bloqueo de redes sociales y aplicaciones

Se configuró un **Application Control Sensor** (`APP_CONTROL`) para bloquear categorías como:
- Social Media
- Aplicaciones P2P
- Proxy
- Remote Access

Esto incluye el bloqueo de llamadas por WhatsApp mediante el control de aplicaciones.

---

## 🌍 Bloqueo de dominios (itla.edu.do)

Se creó un **Web Filter Profile** (`WEB_FILTER`) con un filtro de URL estático:

| URL           | Tipo      | Acción | Estado   |
|---------------|-----------|--------|----------|
| itla.edu.do   | Wildcard  | Block  | Enable   |
| *.itla.edu.do | Wildcard  | Block  | Enable   |

---

## 🛡️ WAF en el servidor Web

Se habilitó el perfil **Web Application Firewall** (`WAF_WEB`) junto con inspección SSL (`certificate-inspection`) en la política `HTTP_to_Server`, protegiendo el servidor IIS de la LAN de servidores.

---

## ✅ Verificación

- Acceso exitoso a Internet desde Win 7 (ej. Wikipedia)
- Acceso HTTP exitoso desde el cliente hacia el servidor IIS (`23.6.1.130`)
- Confirmación del bloqueo de tráfico no autorizado (pruebas de ping fallidas hacia el servidor cuando corresponde)

---



---

## 🧰 Herramientas utilizadas

- **PNETLab** – Simulación de la topología de red
- **FortiGate VM64-KVM** – Firewall / UTM
- **Windows 7** – Máquina cliente
- **Windows Server (IIS)** – Servidor Web
