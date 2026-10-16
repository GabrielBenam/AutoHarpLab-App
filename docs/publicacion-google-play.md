# Publicación de AutoHarpLab en Google Play

Este repositorio conserva el workflow actual que produce APK de depuración y añade un workflow **manual** para preparar un AAB de producción.

## 1. Revisiones de código obligatorias

El proyecto Android se conserva actualmente dentro de `autoarpa-android.zip`. Antes de distribuirlo, extraerlo y comprobar:

- `app/build.gradle.kts`: `compileSdk >= 36`, `targetSdk >= 36` para teléfonos Android en octubre de 2026, `versionCode` positivo y único, y `versionName` significativo.
- Compatibilidad entre Gradle, Android Gradle Plugin, NDK y SDK 36.
- `applicationId` definitivo y único (no se podrá modificar como actualización de la misma app).
- Manifiesto, permisos, bibliotecas externas, seguridad y política de privacidad.
- Que `bundleRelease` produzca realmente un `.aab`, y que funcione la versión de producción en equipos reales.

**Nota:** la instalación de SDK 36 en el workflow no modifica el `targetSdk` declarado por la aplicación. Si el ZIP contiene código o configuración incompatible, se debe actualizar el proyecto y volver a comprimirlo antes de usar el workflow.

## 2. Crear una clave de subida propia (en Windows)

En Android Studio: **Build > Generate Signed Bundle / APK > Android App Bundle > Create new**. Crea un archivo `.jks` privado y conserva un respaldo fuera del repositorio. Utiliza una clave RSA. No publiques la clave ni sus contraseñas.

Para codificar el archivo para GitHub, abre PowerShell y ejecuta, cambiando la ruta:

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\\ruta\\privada\\autoharplab-upload.jks")) | Set-Clipboard
```

Pega el contenido en el secreto indicado abajo. No compartas el valor por mensajes ni archivos públicos.

## 3. Registrar secretos en GitHub

En el repositorio: **Settings > Secrets and variables > Actions > New repository secret**. Crea estos cuatro secretos:

| Nombre exacto | Valor |
| --- | --- |
| `PLAY_UPLOAD_KEYSTORE_BASE64` | Base64 del `.jks`, generado localmente |
| `PLAY_UPLOAD_STORE_PASSWORD` | Contraseña del almacén de claves |
| `PLAY_UPLOAD_KEY_PASSWORD` | Contraseña de la clave |
| `PLAY_UPLOAD_KEY_ALIAS` | Alias de la clave dentro del JKS |

La acción comprueba que existan los cuatro antes de firmar. La clave se reconstruye en un runner temporal, no se incluye en el artefacto.

## 4. Generar el AAB

1. Verifica que el proyecto Android actualizado compile y esté listo para publicación.
2. Abre **Actions > Google Play - AAB firmado > Run workflow**.
3. Entra a la ejecución, revisa los resultados del trabajo `android-play-bundle`.
4. Descarga el artefacto **AutoHarpLab-GooglePlay-AAB**.
5. Extrae de él `AutoHarpLab-release.aab`.

**El workflow no sube ni publica automáticamente el AAB en Play Console.** No es una garantía de cumplimiento de las políticas de Google Play: el paquete debe revisarse y probarse.

## 5. Operaciones que deben hacerse personalmente

1. Crear y verificar una cuenta de Play Console, pagando la tarifa de registro.
2. Crear AutoHarpLab dentro de Play Console y confirmar el identificador de paquete.
3. Activar Play App Signing y registrar la clave de subida conforme a las instrucciones de Google.
4. Añadir icono, gráfico promocional, capturas de pantalla reales, descripción, categoría y contacto.
5. Publicar la política de privacidad correcta y declarar prácticas reales de datos, publicidad y permisos.
6. Subir el AAB a pruebas internas/cerradas y solucionar cualquier error de la consola.
7. Si la cuenta personal es nueva, realizar la prueba cerrada obligatoria con 12 participantes inscritos durante al menos 14 días continuos.
8. Solicitar acceso a producción y enviar la versión para revisión.

Referencias:
- https://support.google.com/googleplay/android-developer/answer/11926878?hl=es
- https://support.google.com/googleplay/android-developer/answer/14151465?hl=es
- https://developer.android.com/studio/publish/app-signing
