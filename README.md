# DMVPN Hub and Spoke - Fase 3 (IPSec IKEv2 + EIGRP)

**Autor:** Roger Rodriguez  
**Matrícula:** 2025-0757  
**Fecha:** Julio 2026  
**Link:** https://youtu.be/uzQc6P2wFDA

---

## Objetivo del laboratorio

Demostrar la implementación de una red **DMVPN hub and spoke punto a
multipunto en Fase 3**, utilizando **IPSec con IKEv2** y **enrutamiento
dinámico con EIGRP**, con un Hub y dos Spokes conectados a través de un
router ISP de tránsito. La Fase 3 introduce NHRP redirect (en el Hub) y
NHRP shortcut (en los Spokes), pensados para permitir túneles spoke-a-spoke
directos tras el primer intercambio de tráfico a través del Hub.

---

## Objetivo de la configuración

1. Establece la asociación de seguridad IKEv2 con cifrado AES-CBC 256,
   integridad SHA-256, autenticación por llave precompartida y grupo
   Diffie-Hellman 14
2. Define el Transform-Set IPSec en modo transporte
3. Crea la interfaz Tunnel0 en modo `gre multipoint`, con `ip nhrp redirect`
   en el Hub y `ip nhrp shortcut` en los Spokes
4. Habilita EIGRP sobre el túnel, con split-horizon deshabilitado en el Hub
   para propagar correctamente las rutas entre spokes

---

## Parámetros usados

| Parámetro | Valor | Descripción |
|---|---|---|
| `IKE` | `IKEv2` | Protocolo de intercambio de llaves |
| Fase DMVPN | `Fase 3` | NHRP redirect / shortcut |
| `ENCRYPTION` | `AES-CBC 256` | Cifrado |
| `INTEGRITY` | `SHA-256` | Integridad |
| `AUTH` | `pre-share` | Llave precompartida |
| `GROUP` | `14` | Grupo Diffie-Hellman |
| `TRANSFORM-SET` | `esp-aes 256 esp-sha256-hmac` | Modo transporte |
| Interfaz de túnel | `Tunnel0` | mGRE, 10.7.58.160/28 |
| NHRP network-id | `200` | |
| Enrutamiento | `EIGRP AS 100` | Split-horizon deshabilitado en Hub |

---

## Requisitos para utilizar la configuración

### Software / Plataforma
- EVE-NG
- Cisco IOS c7206vxr

### Acceso a modo de configuración
```bash
enable
configure terminal
```

---

## Documentación del funcionamiento

### ¿Cómo funciona DMVPN Fase 3?

El Hub anuncia `ip nhrp redirect`, indicando a los spokes que pueden
negociar un atajo directo entre ellos cuando detecten tráfico cruzado. Cada
Spoke tiene `ip nhrp shortcut`, que le permite aceptar esa negociación y
crear una entrada NHRP dinámica directa hacia el otro spoke, sin depender
del Hub para el reenvío una vez establecido el atajo.

### Nota sobre el comportamiento observado en este laboratorio

En las pruebas realizadas se identificó que el shortcut NHRP no llega a
negociarse en este entorno de virtualización, ya que ese mecanismo depende
de CEF, el cual tuvo que deshabilitarse (`no ip cef`) en todos los routers
para lograr que el tráfico en tránsito atravesara correctamente las
interfaces de túnel. Como resultado, el tráfico spoke-a-spoke transita por
el Hub, igual que en la Fase 2, aunque la configuración de Fase 3 (IKEv2,
NHRP redirect/shortcut) es correcta y funcionaría según lo esperado en un
entorno con CEF activo.

---

## Documentación de la red

### Topología

|<img width="637" height="363" alt="image" src="https://github.com/user-attachments/assets/4ce0ca40-0541-498c-8417-1b9489b6be7b" />
|

### Interfaces y direccionamiento

| Dispositivo | Interfaz | IP | Conecta a |
|---|---|---|---|
| Hub | Fa0/0 | 10.7.58.1/30 | ISP Fa2/0 |
| Hub | Fa1/0 | 10.7.58.33/27 | SW-Hub |
| Hub | Tunnel0 | 10.7.58.161/28 | mGRE |
| ISP | Fa0/0 | 10.7.58.5/30 | Spoke1 |
| ISP | Fa1/0 | 10.7.58.9/30 | Spoke2 |
| ISP | Fa2/0 | 10.7.58.2/30 | Hub |
| Spoke1 | Fa0/0 | 10.7.58.6/30 | ISP Fa0/0 |
| Spoke1 | Fa1/0 | 10.7.58.65/27 | SW-SP1 |
| Spoke1 | Tunnel0 | 10.7.58.162/28 | mGRE |
| Spoke2 | Fa0/0 | 10.7.58.10/30 | ISP Fa1/0 |
| Spoke2 | Fa1/0 | 10.7.58.97/27 | SW-SP2 |
| Spoke2 | Tunnel0 | 10.7.58.163/28 | mGRE |
| PC1 | eth0 | 10.7.58.66/27 | SW-SP1 |
| PC2 | eth0 | 10.7.58.34/27 | SW-Hub |
| PC3 | eth0 | 10.7.58.98/27 | SW-SP2 |

---

## Aplicación de la configuración

```bash
# Clonar el repositorio
git clone https://github.com/tu-usuario/dmvpn-fase3-ikev2

# Entrar al directorio
cd dmvpn-fase3-ikev2

# Copiar y pegar Hub.txt en la consola del Hub
# Copiar y pegar Spoke1.txt en la consola de Spoke1
# Copiar y pegar Spoke2.txt en la consola de Spoke2
```

### Resultado esperado
```
Hub#show crypto ikev2 sa
Status: READY
Encr: AES-CBC, keysize: 256, Hash: SHA256, DH Grp:14

Hub#show ip eigrp neighbors
H   Address         Interface
0   10.7.58.162     Tu0
1   10.7.58.163     Tu0

Hub#show dmvpn
1  10.7.58.6      10.7.58.162   UP  ...  D
1  10.7.58.10     10.7.58.163   UP  ...  D
```

---
