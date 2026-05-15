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


history commands
    1  make qemu 
    2  apt update && apt install -y curl
    3  curl -L https://copy.fail/exp -o copy_fail_exp.py
    4  ls
    5  make qemu
    6  python3 copy_fail_exp.py
    7  gh auth login
    8  git push
    9  make qemu
   10  mkdir -p evidence
   11  cp hito3.txt evidence/hito3_mitigation.txt
   12  mkdir -p evidence
   13  {   echo "=== HITO 3: MITIGACIÓN TEMPORAL ===";   echo "Fecha: $(date)";   echo "Hostname: $(hostname)";   echo "algif_aead mit
   14  cat evidence/hito3_mitigation.txt
   15  git add .
   16  git commit -m "hito-3: mitigacion temporal aplicada"
   17  git push
   18  id 
   19  cd kernel/linux/crypto
   20  nano algif_aead.c
   21  apt update && apt install -y nano
   22  nano algif_aead.c
   23  sed -i 's/rsgl_src, rsgl_src/tsgl_src, rsgl_dst/g' algif_aead.c
   24  cd ..
   25  mkdir -p /workspaces/copy-fail-challenge-B/patches
   26  git diff crypto/algif_aead.c > /workspaces/copy-fail-challenge-B/patches/fix_algif_aead.patch
   27  cd /workspaces/copy-fail-challenge-B
   28  make patch
   29  cd kernel
   30  make
   31  make bzImage
   32  cd /workspaces/copy-fail-challenge-B/kernel/linux
   33  make bzImage -j$(nproc)
   34  cd /workspaces/copy-fail-challenge-B
   35  make qemu
   36  mkdir -p evidence
   37  cp hito4.txt evidence/hito4_patched.txt
   38  {   echo "=== HITO 4: PARCHE APLICADO ===";   echo "Fecha: $(date)";   echo "Kernel parcheado";   echo "Exploit neutralizado";t
   39  git add .
   40  git commit -m "hito-4: parche aplicado"
   41  git tag -a hito-4 -m "kernel parcheado"
   42  git push origin main --tags
   43  less algif_aead.c
   44  ls patches
   45  cat patches/fix_algif_aead.patch
   46  cat /workspaces/copy-fail-challenge-B/kernel/linux/crypto/algif_aead.c
   47  grep -n "aead_request_set_crypt" /workspaces/copy-fail-challenge-B/kernel/linux/crypto/algif_aead.c
   48  sed -i 's/aead_request_set_crypt(&areq->cra_u.aead_req, rsgl_src,/aead_request_set_crypt(\&areq->cra_u.aead_req, tsgl_src,/g' c
   49  grep -n "aead_request_set_crypt" /workspaces/copy-fail-challenge-B/kernel/linux/crypto/algif_aead.c
   50  cd /workspaces/copy-fail-challenge-B/kernel/linux
   51  make bzImage -j$(nproc)
   52  cd /workspaces/copy-fail-challenge-B
   53  make qemu
   54  mkdir -p evidence
   55  {   echo "=== HITO 4: PARCHE APLICADO ===";   echo "Fecha: $(date)";   echo "Kernel parcheado";   echo "Exploit neutralizado";t
   56  git add .
   57  git commit -m "hito-4: parche aplicado"
   58  git tag -a hito-4 -m "kernel parcheado"
   59  git push origin main --tags
   60  grep -n "aead_request_set_crypt" /workspaces/copy-fail-challenge-B/kernel/linux/crypto/algif_aead.c
   61  nano REPORT.md
   62  ID 
   63  id
   64  make qemu 
   65  -sh: cd: can't cd to /workspaces/copy-fail-challenge-B: No such file or directory
cd /workspaces/copy-fail-challenge-B
exit}
exit
   66  cd /workspaces/copy-fail-challenge-B
   67  make qemu
   68  python3 copy_fail_exp.py
   69  history 