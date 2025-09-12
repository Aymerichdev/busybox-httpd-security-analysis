# busybox-httpd-security-analysis
# BusyBox HTTPD - Análisis de seguridad (SAST)

## Resumen

Proyecto: Análisis estático de seguridad sobre `networking/httpd.c` de BusyBox (ej. v1.37.0). Se usan herramientas libres: `flawfinder`, `cppcheck`, `splint` y `Frama-C`. El objetivo es identificar debWilidades, clasificarlas con MITRE CWE y realizar post-análisis con un LLM para filtrar falsos positivos.

---

## Estructura del repositorio

```
busybox-httpd-security-analysis/
├─ report.word # Trabajo final donde vendra la tabla de resultados y conclusiones 
├─ busybox-source/                # carpeta con el cód0igo fuente descargado
│  └─ busybox-1.37.0/
├─ analysis_results/               # salidas de las herramientas
├─ scripts/                        # scripts útiles, TODO despues de testear todas las herramientas por separado crear script universal
└─ README.md                       # este archivo
```

---

## Requisitos

* Debian/Ubuntu (o derivado) con `apt-get`.
* Privilegios para instalar paquetes (sudo).

Herramientas que usaremos:

```bash
sudo apt-get update
sudo apt-get install -y flawfinder cppcheck splint frama-c-base build-essential
```

### 1) Descargar el código fuente de BusyBox

```bash
sudo apt-get update
apt-get source busybox
# ejemplo: busybox-1.37.0/
```

### 2) Contar líneas del archivo objetivo

```bash
wc -l busybox-1.37.0/networking/httpd.c
```

### 3) Ejecutar Flawfinder

```bash
mkdir -p analysis_results
flawfinder --minlevel=1 busybox-1.37.0/networking/httpd.c > analysis_results/flawfinder_httpd.txt
# opción menos ruidosa:
flawfinder --minlevel=2 --columns busybox-1.37.0/networking/httpd.c > analysis_results/flawfinder_httpd_level2.txt
```

### 4) Ejecutar Cppcheck

```bash
cppcheck --enable=all --inconclusive --force --inline-suppr --std=c11 busybox-1.37.0/networking/httpd.c 2> analysis_results/cppcheck_httpd.txt
# para suprimir advertencias por includes del sistema:
# cppcheck --enable=all --inconclusive --suppress=missingIncludeSystem busybox-1.37.0/networking/httpd.c 2> analysis_results/cppcheck_filtered.txt
```

### 5) Ejecutar Splint

```bash
splint +bounds +null +charint busybox-1.37.0/networking/httpd.c 2> analysis_results/splint_httpd.txt
```

### 6) Ejecutar Frama-C

#### Opción A — intento directo:

```bash
frama-c -eva busybox-1.37.0/networking/httpd.c -then-on 'msg' -report > analysis_results/frama_httpd_direct.txt 2>&1
```

