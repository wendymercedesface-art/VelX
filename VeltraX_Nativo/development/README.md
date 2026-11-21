# Firma de desarrollo

`veltrax-debug.keystore` conserva la firma del APK de prueba 2.1 para que las próximas compilaciones de este proyecto puedan instalarse como actualizaciones.

Es una clave de depuración, con las credenciales estándar de Android (`android`, alias `androiddebugkey`). No es una clave de publicación. Para distribuir una versión de producción debe configurarse una firma privada propia.

En GitHub Actions la clave se restaura desde el secreto `VELTRAX_DEBUG_KEYSTORE_B64` (ver `CI_GITHUB.md`). No la subas al repositorio.
