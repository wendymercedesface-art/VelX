# VeltraX en GitHub Actions

## Qué hace cada workflow

- `android-apk.yml`: compila el APK de depuración en Ubuntu con JDK 17 y SDK 35, y lo publica como artefacto. Se ejecuta con cada push a `main` o a mano (*Actions → Run workflow*).
- `windows-msi.yml`: genera el instalador MSI del módulo de escritorio en Windows. Se ejecuta a mano.

## Antes de subir el proyecto

1. **La clave de firma no debe llegar al repositorio.** `*.keystore` está en `.gitignore`, así que git no la sube. Compruébalo con `git ls-files | grep -i keystore`: no debe devolver nada. Ojo: arrastrar archivos a la web de GitHub ignora `.gitignore`, y entonces la clave quedaría subida (y pública si el repositorio lo es).
2. **`.github/` debe estar en la raíz del repositorio**, no dentro de `VeltraX_Nativo/`. Si no, GitHub no ejecuta los workflows.
3. **La rama principal debe llamarse `main`**. El push solo dispara el build en esa rama.

## Crear el secreto de firma (una vez)

Convierte la clave local a texto:

- PowerShell: `[Convert]::ToBase64String([IO.File]::ReadAllBytes("development\veltrax-debug.keystore")) | Set-Clipboard`
- Linux: `base64 -w0 development/veltrax-debug.keystore`
- macOS: `base64 -i development/veltrax-debug.keystore | tr -d '\n'`

En GitHub: **Settings → Secrets and variables → Actions → New repository secret**. Nombre: `VELTRAX_DEBUG_KEYSTORE_B64`. Valor: el texto generado.

## Por qué el paso de firma se detiene a propósito

El paso «Restaurar y verificar la clave de firma» decodifica el secreto y compara la huella del certificado con la de VeltraX 2.1. Si falta el secreto o es otra clave, el build se detiene con un mensaje claro. Firmar con otra clave compilaría sin errores, pero el APK no se instalaría como actualización de la 2.1. Eso sería peor que un error explícito.

## Errores frecuentes

| Mensaje | Causa | Solución |
|---|---|---|
| `Falta el secreto` | No creaste `VELTRAX_DEBUG_KEYSTORE_B64` | Paso 2 |
| `Clave incorrecta` | El secreto contiene otra clave | Vuelve a generar el base64 desde `development/veltrax-debug.keystore` |
| `Permission denied` al ejecutar `./gradlew` | Git no guardó el permiso de ejecución | El paso `chmod` lo corrige. A largo plazo: `git update-index --chmod=+x gradlew` |
| `bash\r: No such file or directory` | `gradlew` con finales de línea de Windows | `.gitattributes` lo evita. Si aparece, convierte el archivo a LF |
| `Could not find or load main class ... GradleWrapperMain` | Falta `gradle/wrapper/gradle-wrapper.jar` | Súbelo al repositorio |

## Qué está verificado y qué no

Verificado en el entorno de trabajo:
- El keystore abre con el alias `androiddebugkey` y su huella SHA-256 coincide con la de la 2.1.
- El paso de firma se ejecutó con seis casos: correcto, correcto con espacios y CRLF, vacío, solo saltos de línea, otra clave y texto que no es un keystore. En todos se comporta como se describe arriba.
- git no rastrea el keystore, y el wrapper contiene su clase principal.
- Los dos YAML son válidos.

No verificado: la compilación de Gradle y los workflows en GitHub. El entorno de trabajo no permite descargar dependencias de Maven ni de Gradle. Tampoco el módulo de escritorio, que no aparece en el log de compilación (`verificacion/compilacion.txt`). Si el MSI falla por WiX, instálalo en el workflow, por ejemplo con `choco install wixtoolset`.
