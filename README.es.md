# TVAppInstaller

TVAppInstaller conecta un teléfono Android con una Android TV o TV Box por ADB en la red local. Sirve para instalar APKs, administrar aplicaciones y elegir el launcher de inicio de la TV. Se comunica directamente por ADB usando Java: no ejecuta `adb.exe` ni necesita Platform-Tools en el teléfono.

[English](README.md) | [Español](README.es.md)

## Versión actual

| | |
|---|---|
| Versión | `1.0` |
| Version code | `1` |
| APK | [TVAppInstaller_v1.0-vc1_20261120.010954.apk](app/build/outputs/apk/debug/TVAppInstaller_v1.0-vc1_20261120.010954.apk) |

Es una compilación **debug**, firmada con la clave de prueba del proyecto. Sirve para probar e instalar manualmente, pero no es una versión final para distribuir.

## Funciones

- Conectarse y desconectarse de una TV por IP y puerto ADB (`5555` por defecto).
- Instalar un APK elegido con el explorador de archivos de Android.
- Listar aplicaciones de terceros y desinstalar las seleccionadas con confirmación.
- Buscar launchers disponibles y pedirle a la TV que establezca uno como predeterminado.
- Ver el estado y los resultados de las operaciones en el registro de la app.
- Usar pantallas separadas para conexión, instalación, aplicaciones y launchers.

## Requisitos y compatibilidad

- El teléfono donde corre la app debe tener Android **5.0 (API 21) o posterior** (`minSdk 21`).
- La TV debe permitir **ADB TCP clásico**, estar accesible por la red y aceptar la clave RSA de la app.
- El teléfono y la TV deben poder comunicarse por el puerto elegido.
- Los comandos pueden variar según la versión de Android y el fabricante; la gestión de launchers y la instalación podrían comportarse distinto entre modelos.

**No admite el emparejamiento de Wireless debugging.** Esta app usa ADB TCP clásico, normalmente en el puerto `5555`; no puede emparejarse con el código o QR del modo moderno de Wireless debugging.

## Cómo habilitar las opciones de desarrollador y ADB en la TV

Los nombres de las opciones cambian según la versión de Android TV y el fabricante. Estos son los pasos habituales:

1. Abrí **Configuración** y entrá en **Preferencias del dispositivo** (o **Sistema**) > **Información**.
2. Buscá **Compilación** o **Número de compilación** y presioná **Seleccionar/OK** siete veces, hasta que la TV avise que las opciones de desarrollador están habilitadas.
3. Volvé a Configuración y abrí **Opciones de desarrollador**.
4. Habilitá la opción que permita **ADB por red**. Puede llamarse **Depuración ADB**, **Depuración de red** o **Depuración por red**; el nombre exacto depende de la TV.
5. Si la TV solo ofrece **Wireless debugging** con código o QR de emparejamiento, esta app no puede conectarse con ese modo. Habilitar las opciones de desarrollador o la depuración USB no siempre habilita ADB TCP clásico.
6. Buscá la dirección IP de la TV en la configuración de red. Dejá el puerto en `5555`, salvo que la TV indique otro.
7. Conectá el teléfono y la TV a una red que permita que se comuniquen.

Si no encontrás una opción para ADB TCP clásico, revisá las instrucciones del modelo de tu TV. La app no puede habilitar ADB en la TV de forma remota.

## Primera conexión

1. Abrí TVAppInstaller e ingresá la IP y el puerto ADB de la TV.
2. Tocá **Conectar**.
3. Aceptá en la TV el aviso de autorización RSA. La app no puede conectarse hasta que lo autorices.
4. Elegí **Instalar APK**, **Aplicaciones** o **Launchers**. La app pide confirmación antes de desinstalar una aplicación o cambiar el launcher.

Si no conecta, revisá la IP, el puerto, la red y la opción de ADB por TCP; después, fijate si aparece el aviso de autorización en la TV.

## Cómo funciona ADB en la app

`AdbRunner` usa **AdbLib** para conectarse directamente a la TV. La app guarda sus claves RSA en su espacio privado y usa la conexión autorizada para listar o desinstalar aplicaciones de terceros, instalar APKs y consultar o cambiar el launcher de inicio.

Para instalar, el APK se copia temporalmente a la TV y la app intenta borrarlo al terminar. Las líneas `adb ...` del registro son descripciones de las operaciones; no son comandos ejecutados con un programa ADB externo.

## Privacidad y uso seguro

- La app se conecta solo a la dirección que ingresás; no busca dispositivos en la red.
- La TV debe aprobar la clave RSA. La app no evita esa autorización.
- El APK elegido se guarda temporalmente durante la transferencia y la app intenta borrar las copias temporales al terminar.
- Usala solo con TVs que tengas permiso para administrar. Revisá y confirmá cada desinstalación o cambio de launcher.

## Compilar desde el código fuente

En Windows, instalá Android SDK y ejecutá esto desde la carpeta raíz del proyecto:

```powershell
.\gradlew.bat :app:assembleDebug --console=plain
```

La APK debug queda en `app/build/outputs/apk/debug/`. El proyecto usa Android Gradle Plugin `7.2.1`, Gradle `7.4`, `compileSdk 33` y `targetSdk 33`.

> La configuración release también usa la clave de firma debug. Antes de distribuir una versión, configurá y protegé tu propia clave de firma.

## Dependencias principales

- [AdbLib](https://github.com/tananaev/adblib) — `com.tananaev:adblib:1.3`, implementación Java del protocolo ADB.
- AndroidX Core KTX `1.9.0`.
- AndroidX AppCompat `1.6.1`.
- Material Components `1.9.0`.
- ConstraintLayout `2.1.4`.

## Estructura del proyecto

- `app/src/main/java/com/example/tvappinstaller/MainActivity.java`: pantallas, navegación, validación de datos y acciones de la interfaz.
- `app/src/main/java/com/example/tvappinstaller/AdbRunner.java`: conexión ADB autenticada y operaciones con la TV.
- `app/src/main/res/layout/`: diseños de la actividad y las pantallas.
- `app/src/main/AndroidManifest.xml`: manifiesto y permiso de red.
- `app/build.gradle`: configuración y dependencias del módulo Android.

## Novedades de la versión 1.0

- Conexión por IPv4, puerto `5555` predeterminado y autorización RSA.
- Listado y desinstalación confirmada de aplicaciones de terceros.
- Selección, transferencia e instalación de APKs.
- Pantallas separadas, registro persistente y selección del launcher de inicio.

## Ideas para más adelante

El sueño a largo plazo es que TVAppInstaller pueda conectarse a una TV a distancia y ofrecer algo parecido a TeamViewer: ver la pantalla y manejar la tele desde el teléfono. VNC podría ser una opción para investigar, pero todavía no está decidido ni implementado. Va a depender de la compatibilidad de cada TV y Android, de la autorización del usuario y de contar con una conexión segura.

## Pruebas y limitaciones conocidas

El código Java compiló durante el desarrollo, pero los comandos ADB pueden variar según el fabricante. Probá especialmente la instalación de APKs y el cambio de launcher en el modelo de TV que vayas a usar.

## Agradecimientos y licencias

Gracias a [Anton Tananaev (@tananaev)](https://github.com/tananaev) por [AdbLib](https://github.com/tananaev/adblib), y a [Cameron Gutman (@cgutman)](https://github.com/cgutman), creador del proyecto original. También, gracias al [Android Open Source Project](https://android.googlesource.com/platform/system/core/+/master/adb/protocol.txt) por la documentación de ADB y a quienes probaron la app y compartieron comentarios.

AdbLib usa la licencia **BSD 3-Clause**; podés consultar su [texto](https://github.com/tananaev/adblib/blob/master/LICENSE). El código propio de TVAppInstaller todavía no tiene una licencia definida.

Para más información, consultá las guías oficiales de Android sobre [opciones de desarrollador](https://developer.android.com/studio/debug/dev-options) y [ADB por Wi-Fi](https://developer.android.com/tools/adb#connect-to-a-device-over-wi-fi).
