# Traspaso de TVAppInstaller a Cline + Ollama

Este archivo resume el estado del proyecto y contiene un prompt para continuar el trabajo con Cline y el modelo local. Úsalo como contexto inicial; los archivos fuente actuales del workspace son la referencia definitiva.

## Cómo usar este traspaso

1. Abre en VS Code la carpeta `F:\Users\Abdal\Desktop\TVAppInstaller`.
2. Copia el bloque **Prompt para Cline** de abajo en Cline.
3. Asegúrate de que Cline tenga acceso a lectura de archivos y pídele explícitamente que lea las rutas antes de proponer o hacer cambios.
4. No pegues ni envíes el ejecutable `adb` de la raíz a servicios externos: no forma parte del backend Android actual.

## Prompt para Cline

```text
Continúa el proyecto Android existente TVAppInstaller en:
F:\Users\Abdal\Desktop\TVAppInstaller

Antes de responder o modificar nada, usa las herramientas read-file de este
workspace y lee completos estos archivos:

- app/src/main/java/com/example/tvappinstaller/MainActivity.java
- app/src/main/java/com/example/tvappinstaller/AdbRunner.java
- app/src/main/res/layout/activity_main.xml
- app/src/main/AndroidManifest.xml
- app/build.gradle
- build.gradle
- settings.gradle
- gradle/wrapper/gradle-wrapper.properties
- gradle.properties

Comprueba también los archivos presentes bajo app/src/main (especialmente
assets), porque el estado puede haber cambiado desde este traspaso. No asumas
que el texto de este documento sustituye la lectura del código.

OBJETIVO DEL PRODUCTO

Es una app Android llamada TVAppInstaller, paquete com.example.tvappinstaller,
que corre en un teléfono/emulador Android y administra una TV Android que el
usuario controla y autoriza mediante ADB TCP clásico en la red local.
La interfaz debe permitir:

1. Ingresar IPv4 y puerto opcional (5555 por defecto), conectar y desconectar.
2. Al conectar, alternar el botón de conexión a “Desconectar” y abrir un menú.
3. Tener pantallas separadas para instalar, listar/desinstalar apps y gestionar
   launchers; cada pantalla de función tiene un botón “Atrás” arriba a la
   izquierda.
4. Mostrar el resultado y los errores en una consola persistente dentro de la app.
5. Listar paquetes de terceros de la TV y mostrarlos como checkbox.
6. Seleccionar una o más apps, confirmar y desinstalarlas.
7. Pulsar “Instalar APK”, elegir un APK con el selector de archivos de Android,
   enviarlo a la TV e instalarlo.
8. Consultar actividades HOME/launcher, seleccionar una y solicitar que la TV
   la establezca como predeterminada, con confirmación.

La app debe utilizar solo TVs autorizadas por el usuario. No agregues escaneo
de red, descubrimiento automático de objetivos, bypass de autenticación ni
funciones ocultas. La autorización RSA que solicita la TV debe permanecer
activa y visible.

ARQUITECTURA ACTUAL

- MainActivity.java administra las pantallas, valida IPv4/puerto y maneja los
  botones, el selector de APK, la lista de paquetes y la selección de launcher.
- `activity_main.xml` contiene el host de pantallas y la consola persistente.
  Las pantallas individuales son `screen_connection.xml`, `screen_menu.xml`,
  `screen_install.xml`, `screen_apps.xml` y `screen_launchers.xml`.
- La pantalla de conexión solo contiene IP, puerto y el botón toggle; si el
  puerto está vacío, se usa 5555. Al conectar se abre el menú; atrás desde el
  menú vuelve a conexión y el botón muestra “Desconectar”.
- AdbRunner.java es el backend asíncrono. Usa la biblioteca Java AdbLib
  (com.tananaev:adblib:1.3), Socket y el protocolo ADB clásico; no lanza un
  proceso adb.exe ni un binario nativo.
- La clave RSA se genera/lee dentro del almacenamiento privado de la app
  (filesDir/adbkey y adbkey.pub). La primera conexión puede mostrar en la TV el
  diálogo de autorización de esa clave.
- Para listar usa el servicio shell ADB con `pm list packages -3`.
- Para desinstalar usa el servicio shell ADB con `pm uninstall <package>`.
- Para consultar launchers, AdbRunner intenta `cmd package query-activities`
  con el intent MAIN/HOME y usa `pm query-activities` como fallback cuando el
  subcomando no está disponible. Para cambiar el launcher intenta
  `cmd package set-home-activity` y tiene un fallback `pm set-home-activity`.
- Para instalar, el código actual selecciona un Uri con
  ACTION_OPEN_DOCUMENT, copia el contenido temporalmente a cache, valida que el
  ZIP tenga AndroidManifest.xml, transfiere el archivo por el servicio `sync:`
  a `/data/local/tmp/`, ejecuta `pm install -r` y luego intenta borrar el
  temporal remoto.
- Las líneas `adb ...` de la consola son trazas legibles; no son comandos
  ejecutados en CMD/PowerShell. Los datos se envían mediante el protocolo ADB.
- La interfaz principal está en activity_main.xml. Conserva los IDs que usa
  MainActivity; si cambias la UI, actualiza ambos lados de forma consistente.

ARCHIVOS IMPORTANTES

- `app/src/main/java/com/example/tvappinstaller/MainActivity.java`
- `app/src/main/java/com/example/tvappinstaller/AdbRunner.java`
- `app/src/main/res/layout/activity_main.xml`
- `app/src/main/res/layout/screen_connection.xml`
- `app/src/main/res/layout/screen_menu.xml`
- `app/src/main/res/layout/screen_install.xml`
- `app/src/main/res/layout/screen_apps.xml`
- `app/src/main/res/layout/screen_launchers.xml`
- `app/src/main/AndroidManifest.xml`
- `app/src/main/res/values/strings.xml`
- `app/build.gradle`
- `build.gradle`
- `settings.gradle`
- `gradle/wrapper/gradle-wrapper.properties`

CONFIGURACIÓN ACTUAL

- applicationId y namespace: `com.example.tvappinstaller`
- minSdk 21, targetSdk 33, compileSdk 33
- Android Gradle Plugin 7.2.1 y Gradle wrapper 7.4
- Dependencia ADB: `com.tananaev:adblib:1.3`
- El manifiesto declara INTERNET. La selección de documentos usa Storage Access
  Framework; no debería requerir READ_EXTERNAL_STORAGE.
- `local.properties` tiene una ruta de SDK local específica de esta máquina.
  No la reemplaces por una ruta inventada ni la agregues a control de versiones.

LIMITACIONES CONOCIDAS Y COSAS QUE HAY QUE VALIDAR

- La biblioteca elegida implementa ADB TCP clásico con autenticación RSA. No
  implementa el emparejamiento moderno de “Wireless debugging” por código/TLS.
  No anuncies compatibilidad con ese modo sin reemplazar/verificar el backend.
- La TV debe permitir conexiones ADB TCP, normalmente en el puerto 5555, estar
  accesible en la red local y aceptar la clave de esta app.
- Los comandos para enumerar y fijar actividades HOME pueden variar entre
  fabricantes/versiones de Android TV. La compilación no valida esos comandos;
  probar la pantalla de launcher en una TV real y reportar la salida completa
  antes de afirmar compatibilidad.
- El archivo `adb` que había en la raíz del workspace es un ELF Linux x86_64
  que requiere `/lib64/ld-linux-x86-64.so.2`; no es un ejecutable Android.
  Se quitó la copia de `app/src/main/assets/adb`. No vuelvas a copiar ese ELF a
  assets ni a ejecutar binarios descargados desde filesDir.
- El código Java compiló correctamente con
  `.\gradlew.bat :app:compileDebugJavaWithJavac --console=plain`.
  No se generó APK en esa validación y no se probó la instalación de APK contra
  una TV real. El usuario compila/instala el APK por su cuenta.
- La transferencia ADB `sync:` y el flujo `pm install -r` deben probarse en una
  TV real. Si fallan, inspecciona el log de la app, verifica el protocolo
  SEND/DATA/DONE/OKAY, el modo/ruta del archivo remoto y la respuesta real de
  `pm install`; no informes éxito si la TV no lo confirma.
- `MainActivity` usa `startActivityForResult`, que está deprecated pero puede
  funcionar. Migrarlo a Activity Result API es una mejora opcional, no requisito
  para cambiarlo sin necesidad.
- El build `release` está configurado con firma debug: sirve para desarrollo,
  no para distribuir una versión final.

MODO DE TRABAJO

- Primero inspecciona; luego explica brevemente el problema encontrado.
- Haz cambios pequeños y coherentes, siguiendo el Java/XML/Gradle existente.
- Conserva todas las operaciones de red fuera del hilo principal.
- No concatentes datos del usuario en un comando de shell genérico. Mantén las
  operaciones acotadas (conectar, listar, instalar, desinstalar) y valida los
  nombres de paquete/rutas.
- No elimines archivos de usuario ni cambies paquetes/dependencias salvo que
  sea necesario y lo expliques.
- No afirmes que la app funciona en una TV solo porque compile.
- Después de cambios, ejecuta el chequeo más acotado que los valide. Para la
  compilación Java usa:
  `.\gradlew.bat :app:compileDebugJavaWithJavac --console=plain`
- No ejecutes `assembleDebug` si el usuario solo pidió cambios fuente o dijo
  que él generará el APK.
- Responde en español claro, ya que el usuario no es programador.
```

## Historial breve

- Se creó la UI para IP/puerto, conexión, consola y listado de apps.
- Se corrigió el color del texto de los paquetes: antes era blanco sobre fondo
  blanco y parecía que los nombres estaban vacíos.
- Se agregó la dependencia AdbLib y se reemplazó el uso del ELF Linux por un
  cliente Java del protocolo ADB clásico.
- Se agregó selección de paquetes con checkbox y desinstalación confirmada.
- Se agregó “Instalar APK”, selector del sistema, transferencia temporal ADB y
  llamada a `pm install -r`.
- Se reorganizó la navegación: conexión, menú, instalación, apps y launchers
  tienen pantallas separadas, con consola persistente y navegación Atrás.
- Se añadió la consulta de actividades HOME y selección para intentar cambiar
  el launcher predeterminado de la TV.
- La compilación Java pasó después de esos cambios; falta validar la instalación
  de APK y el cambio de launcher con una TV real y generar el APK final desde la
  máquina del usuario.
