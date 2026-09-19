# Windows RDP temporal con GitHub Actions + Tailscale

Este proyecto reproduce el flujo del video: GitHub Actions crea un runner Windows temporal, habilita Remote Desktop, crea un usuario local, conecta el runner a Tailscale y mantiene el job activo para poder conectarse desde **Windows App** por RDP.

## Importante

Esto **no es una VM persistente que tú administras**. En GitHub-hosted runners, la VM pertenece al job y se elimina cuando termina. El límite actual de un job en runners hospedados por GitHub es de 6 horas. La configuración por defecto de este proyecto mantiene la sesión 60 minutos para evitar dejar un job consumiendo minutos accidentalmente.

## 1. Crear el repositorio

Copia estos archivos a un repositorio de GitHub, preferiblemente privado si contiene configuraciones propias.

## 2. Configurar Tailscale

En tu tailnet crea la etiqueta `tag:ci` y una identidad federada/OAuth para GitHub Actions.

La opción recomendada actualmente por Tailscale es Workload Identity Federation. Guarda en GitHub Actions estos secrets:

- `TS_OAUTH_CLIENT_ID`
- `TS_AUDIENCE`

La acción de Tailscale es `tailscale/github-action@v4`.

## 3. Crear la regla de acceso

Usa `tailnet-policy.example.hujson` como plantilla. Sustituye `tu-correo@ejemplo.com` por tu identidad de Tailscale.

La regla permite únicamente TCP 3389 desde ese usuario hacia dispositivos con `tag:ci`.

## 4. Crear la contraseña RDP

En GitHub:

`Settings -> Secrets and variables -> Actions -> New repository secret`

Crea:

`RDP_PASSWORD`

No pongas la contraseña directamente en el archivo YAML. El workflow la toma como secret y nunca la imprime.

## 5. Lanzar la VM temporal

Ve a:

`Actions -> Windows RDP + Tailscale -> Run workflow`

Selecciona `duration_minutes` entre 5 y 330.

Cuando el job llegue a `Verify Tailscale and RDP`, abre el **Summary** del workflow. Ahí aparecerá la IPv4 de Tailscale y el usuario RDP.

## 6. Conectar desde Microsoft Windows App

En Android, iPhone, Windows o macOS abre **Windows App** y agrega un PC.

En **PC name** usa:

`100.x.y.z`

o el nombre DNS de Tailscale indicado en el Summary.

En la cuenta usa:

`rdpuser`

Si la aplicación pide una cuenta con dominio, usa:

`\\rdpuser`

La contraseña es la que guardaste en `RDP_PASSWORD`.

El dispositivo desde el que conectas también debe estar conectado a tu tailnet.

## 7. Terminar la sesión

Pulsa **Cancel workflow** en GitHub Actions. Al finalizar el job, el runner temporal se destruye y Tailscale elimina el nodo efímero creado por la acción.

## Archivos

- `.github/workflows/windows-rdp.yml` — workflow completo.
- `tailnet-policy.example.hujson` — ejemplo de política Tailscale.
