# Procedimiento Completo para Generar y Distribuir una APK de Producción con Capacitor

## Este procedimiento garantiza la obtención de un archivo ejecutable APK firmado (release) listo para su distribución, incluyendo acceso a hardware del dispositivo

## Requisitos

- Node.js ≥20
- Android Studio descomprimido (ej. ~/android-studio/)
- JDK 17 (incluido en Android Studio)

## Parte 1: Configuración Inicial del Proyecto

### 1.1 Creación del Proyecto Base

Ejecute los siguientes comandos para inicializar el proyecto Capacitor:

```bash
mkdir capacitor_test
cd capacitor_test
mkdir src
```

Copie el siguiente index.html a `proyecto_root/src/`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>App</title>
</head>
<body>
  <h1>OK</h1>
</body>
</html>
```

### 1.2 Inicialización de Capacitor

El nombre de la app capacitor (ej. "com.example.app2") debe estar en formato DNS reverso:

```bash
npx @capacitor/cli init app "com.example.app2" --web-dir src
npm install @capacitor/core @capacitor/cli @capacitor/android
npx cap add android 
npx cap sync android
```

## Parte 2: Configuración de Acceso a Hardware del Dispositivo

### 2.1 Instalación de Plugins para Hardware

Para acceder a funciones del hardware como cámara, acelerómetro, GPS, etc., instale los plugins necesarios:

```bash
# Cámara
npm install @capacitor/camera

# Geolocalización
npm install @capacitor/geolocation

# Sensores de movimiento (acelerómetro, giroscopio)
npm install @capacitor/motion

# Otros plugins útiles:
# npm install @capacitor/haptics        # Vibración
# npm install @capacitor/filesystem     # Sistema de archivos
# npm install @capacitor/device         # Información del dispositivo
# npm install @capacitor/network        # Estado de la red
```

Después de instalar plugins, sincronice:

```bash
npx cap sync android
```

### 2.2 Configuración de Permisos en Android

Edite el archivo `android/app/src/main/AndroidManifest.xml` para agregar los permisos necesarios:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    
    <!-- Permisos para Cámara -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-feature android:name="android.hardware.camera" android:required="false" />
    
    <!-- Permisos para Geolocalización -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    
    <!-- Permisos para Sensores de Movimiento (generalmente no requieren declaración explícita) -->
    
    <!-- Permisos adicionales según necesidad -->
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    
    <application>
        <!-- Contenido de la aplicación -->
    </application>
</manifest>
```

### 2.3 Ejemplo de Uso de Hardware en el Código

Actualice su `src/index.html` con ejemplos de uso:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>App con Hardware</title>
  <script type="module">
    import { Camera, CameraResultType } from '@capacitor/camera';
    import { Geolocation } from '@capacitor/geolocation';
    import { Motion } from '@capacitor/motion';

    // Función para tomar foto
    window.takePicture = async () => {
      try {
        const image = await Camera.getPhoto({
          quality: 90,
          allowEditing: false,
          resultType: CameraResultType.Uri
        });
        document.getElementById('result').innerHTML = `<img src="${image.webPath}" style="max-width:300px">`;
      } catch (error) {
        console.error('Error al acceder a la cámara:', error);
      }
    };

    // Función para obtener ubicación
    window.getLocation = async () => {
      try {
        const coordinates = await Geolocation.getCurrentPosition();
        document.getElementById('result').innerHTML = 
          `Lat: ${coordinates.coords.latitude}<br>Lng: ${coordinates.coords.longitude}`;
      } catch (error) {
        console.error('Error al obtener ubicación:', error);
      }
    };

    // Función para leer acelerómetro
    window.readAccelerometer = async () => {
      try {
        const handler = await Motion.addListener('accel', (event) => {
          document.getElementById('result').innerHTML = 
            `X: ${event.acceleration.x.toFixed(2)}<br>
             Y: ${event.acceleration.y.toFixed(2)}<br>
             Z: ${event.acceleration.z.toFixed(2)}`;
        });
        
        // Detener después de 5 segundos
        setTimeout(async () => {
          await handler.remove();
        }, 5000);
      } catch (error) {
        console.error('Error al leer acelerómetro:', error);
      }
    };
  </script>
</head>
<body>
  <h1>Prueba de Hardware</h1>
  <button onclick="takePicture()">Tomar Foto</button>
  <button onclick="getLocation()">Obtener Ubicación</button>
  <button onclick="readAccelerometer()">Leer Acelerómetro (5s)</button>
  <div id="result"></div>
</body>
</html>
```

Después de modificar el código, sincronice nuevamente:

```bash
npx cap sync android
```

## Parte 3: Generación del APK de Producción

### 3.1 Apertura del Proyecto en Android Studio

- Abra Android Studio
- En la pantalla de bienvenida, seleccione **Open**
- Navegue y seleccione la carpeta `android` ubicada dentro del directorio raíz de su proyecto Capacitor
- Confirme con **OK**

### 3.2 Configuración de la Firma (Signing para Release)

La variante release requiere que la aplicación esté firmada mediante un keystore.

En el menú superior de Android Studio:

**Build → Generate Signed Bundle / APK...**

- Seleccione la opción **APK**
- Haga clic en **Next**
- Verifique que el campo **Module** muestre `app`
- En la sección **Key store path**:
  - Si el archivo de clave (.jks o .keystore) ya existe, selecciónelo
  - Si no existe, haga clic en **Create new...** y complete:
    - **Key store path**: Ubicación donde se guardará el keystore
    - **Password**: Contraseña del keystore
    - **Alias**: Nombre de la clave
    - **Password**: Contraseña de la clave
    - **Validity (years)**: 25 años (mínimo recomendado)
    - **Certificate**: Datos del desarrollador
- Proporcione las contraseñas para el Keystore y la Key
- Haga clic en **Next**

⚠️ **IMPORTANTE**: Guarde el archivo keystore y las contraseñas en un lugar seguro. Son necesarios para futuras actualizaciones.

### 3.3 Generación del APK de Producción (Release)

- En la ventana **Generate Signed Bundle or APK**:
  - Asegúrese de que en **Build Variants** esté marcada la opción **release**
  - Active la casilla **Sign the bundle/APK**
  - Haga clic en **Finish**

El proceso de Gradle se ejecutará para compilar y firmar el archivo binario.

### 3.4 Ubicación Final del APK

Una vez concluida la construcción, el archivo APK estará en:

```
android/app/release/app-release.apk
```

## Parte 4: Instalación del APK en Dispositivos Android

### 4.1 Instalación Directa (Sideloading)

#### Método 1: Transferencia Directa al Dispositivo

1. **Habilitar Instalación desde Fuentes Desconocidas**:
   - Abra **Configuración** en el dispositivo Android
   - Vaya a **Seguridad** o **Privacidad**
   - Active **Fuentes desconocidas** o **Instalar aplicaciones desconocidas**
   - En versiones recientes de Android, deberá permitir esto para la aplicación específica (ej. Chrome, Administrador de archivos)

2. **Transferir el APK**:
   - **Por USB**: Conecte el dispositivo y copie el APK usando el explorador de archivos
   - **Por Email**: Envíe el APK como adjunto y descárguelo en el dispositivo
   - **Por Cloud**: Suba el APK a Google Drive, Dropbox, etc. y descárguelo
   - **Por ADB**: Use el comando:
     ```bash
     adb install android/app/release/app-release.apk
     ```

3. **Instalar**:
   - Abra el archivo APK desde el administrador de archivos del dispositivo
   - Toque **Instalar**
   - Acepte los permisos solicitados
   - Toque **Abrir** para ejecutar la aplicación

#### Método 2: Usando ADB (Android Debug Bridge)

1. **Instalar ADB**:
   - Ya está incluido en Android Studio en: `~/android-studio/platform-tools/`
   - O instálelo por separado según su sistema operativo

2. **Habilitar Depuración USB**:
   - En el dispositivo, vaya a **Configuración → Acerca del teléfono**
   - Toque 7 veces sobre **Número de compilación** para activar **Opciones de desarrollador**
   - Vaya a **Configuración → Opciones de desarrollador**
   - Active **Depuración USB**

3. **Conectar y Verificar**:
   ```bash
   adb devices
   ```
   Debería ver su dispositivo listado

4. **Instalar el APK**:
   ```bash
   adb install -r android/app/release/app-release.apk
   ```
   El flag `-r` reinstala la app si ya existe

### 4.2 Distribución Interna

Para distribuir a un grupo pequeño de probadores:

1. **Firebase App Distribution**: Servicio gratuito de Google
2. **TestFlight** (solo para iOS)
3. **Servidor web propio**: Publique el APK en su servidor con instrucciones

## Parte 5: Publicación en Google Play Store

### 5.1 Requisitos Previos

- **Cuenta de Google Play Console**: Costo único de $25 USD
- **Políticas cumplidas**: Revise las [Políticas del programa para desarrolladores](https://play.google.com/about/developer-content-policy/)
- **Aplicación probada**: Sin errores críticos

### 5.2 Preparación de Assets

Prepare los siguientes recursos gráficos:

1. **Icono de aplicación**: 512x512 px (PNG, 32 bits)
2. **Banner destacado**: 1024x500 px
3. **Capturas de pantalla**:
   - Teléfono: Mínimo 2, máximo 8 (JPEG o PNG de 24 bits)
   - Tablet (opcional): Mínimo 1, máximo 8
4. **Video promocional** (opcional): URL de YouTube

### 5.3 Generación del App Bundle (Recomendado)

Google recomienda usar **Android App Bundle** (.aab) en lugar de APK:

En Android Studio:

**Build → Generate Signed Bundle / APK...**

- Seleccione **Android App Bundle**
- Siga los mismos pasos de firma que para el APK
- El archivo resultante estará en: `android/app/release/app-release.aab`

### 5.4 Proceso de Publicación en Google Play Console

#### Paso 1: Crear la Aplicación

1. Acceda a [Google Play Console](https://play.google.com/console)
2. Haga clic en **Crear aplicación**
3. Complete:
   - **Nombre de la aplicación**
   - **Idioma predeterminado**
   - **Tipo**: Aplicación o Juego
   - **Gratuita o de pago**
4. Acepte las declaraciones y haga clic en **Crear aplicación**

#### Paso 2: Configurar la Ficha de Play Store

Vaya a **Presencia en Play Store → Ficha principal de Play Store**:

1. **Detalles de la aplicación**:
   - Descripción breve (80 caracteres)
   - Descripción completa (4000 caracteres)
   
2. **Recursos gráficos**:
   - Suba icono, capturas de pantalla y banner
   
3. **Categorización**:
   - Seleccione categoría y etiquetas
   
4. **Información de contacto**:
   - Email, sitio web, teléfono (opcional)

5. **Guarde los cambios**

#### Paso 3: Configurar Contenido de la Aplicación

Complete todas las secciones requeridas:

1. **Política de privacidad**: URL obligatoria
2. **Clasificación de contenido**: Complete el cuestionario IARC
3. **Público objetivo**: Grupos de edad
4. **Categoría**: Aplicación o juego
5. **Datos de seguridad**: Declaración de datos recopilados
6. **Anuncios**: Indique si contiene anuncios

#### Paso 4: Seleccionar Países y Regiones

En **Producción → Países y regiones**:

- Seleccione los países donde se distribuirá
- Configure precios si la app es de pago

#### Paso 5: Subir la Versión de Producción

1. Vaya a **Producción → Lanzamientos**
2. Haga clic en **Crear nuevo lanzamiento**
3. En **App bundles y APKs**:
   - Haga clic en **Subir**
   - Seleccione su archivo `app-release.aab` o `app-release.apk`
4. Complete:
   - **Nombre del lanzamiento**: Ej. "Versión 1.0"
   - **Notas de la versión**: Descripción de las funciones para cada idioma
5. Haga clic en **Guardar** y luego **Revisar lanzamiento**

#### Paso 6: Revisión y Publicación

1. Google Play Console verificará que todos los requisitos estén completos
2. Revise cualquier advertencia o error
3. Cuando esté listo, haga clic en **Iniciar implementación en producción**
4. Confirme la publicación

### 5.5 Proceso de Revisión

- Google revisará la aplicación (puede tomar de horas a varios días)
- Recibirá notificaciones por email sobre el estado
- Una vez aprobada, la app estará disponible en Play Store

### 5.6 Actualizaciones Futuras

Para publicar actualizaciones:

1. Incremente el **versionCode** y **versionName** en `android/app/build.gradle`:
   ```gradle
   android {
       defaultConfig {
           versionCode 2
           versionName "1.1"
       }
   }
   ```
2. Genere un nuevo App Bundle/APK firmado con el **mismo keystore**
3. En Google Play Console, vaya a **Producción → Lanzamientos**
4. Cree un nuevo lanzamiento y suba el nuevo archivo
5. Complete las notas de la versión y publique

## Parte 6: Consejos y Mejores Prácticas

### 6.1 Seguridad del Keystore

- **Haga copias de seguridad** del archivo keystore en múltiples ubicaciones seguras
- **Nunca comparta** las contraseñas públicamente ni en repositorios
- Si pierde el keystore, **no podrá actualizar la aplicación** en Play Store

### 6.2 Versionado

Use versionado semántico:
- **1.0.0**: Primer lanzamiento
- **1.0.1**: Correcciones de errores
- **1.1.0**: Nuevas funciones
- **2.0.0**: Cambios importantes

### 6.3 Pruebas Antes de Publicar

- Use **Prueba interna** o **Prueba cerrada** en Google Play Console
- Pruebe en múltiples dispositivos y versiones de Android
- Verifique todos los permisos de hardware funcionan correctamente

### 6.4 Optimización del APK

Para reducir el tamaño del APK:

```bashDebug
# Habilitar ProGuard en android/app/build.gradle
buildTypes {
    release {
        minifyEnabled true
        shrinkResources true
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
}
```

---

## COMPILACION APK DEBGUB Y COMO USAR CONSOLA CHROME CONECTANDO EL TELEFONO CON LA APK INSTALADA POR USB A LA PC
Para poder acceder a la consola de desarrollador de chrome es necesario compilar la apk como debug, la re producción no permite usar la consola.


## Referencias Útiles

- [Documentación de Capacitor](https://capacitorjs.com/docs)
- [Plugins de Capacitor](https://capacitorjs.com/docs/plugins)
- [Google Play Console](https://play.google.com/console)
- [Políticas de Google Play](https://play.google.com/about/developer-content-policy/)
- [Android Studio](https://developer.android.com/studio)