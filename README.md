# Despertador Inteligente

App Android (Kotlin + Jetpack Compose) y sketch del ESP32. **El celular no habla directo con el ESP32:** los dos leen y escriben en Firebase (Firestore) con el mismo usuario.

```
Celular (app)  <──►  Firestore  <──►  ESP32
   escribe: alarmas, luces             escribe: estado (cada 15 s), historial (cada 60 s)
   lee: estado, historial              lee: alarmas y luces (cada 20 s)
```

## 1. Poner la app en el celular

1. Firebase Console → Configuración del proyecto → **Tus apps** → app Android `com.despertador.app` → descarga **google-services.json**.
2. Copia ese archivo en `app/google-services.json` (dentro de la carpeta `app`, junto a `build.gradle.kts`).
3. Android Studio → **File > Open** → carpeta `DespertadorInteligente` → espera el *Gradle Sync*.
4. Conecta el celular (depuración USB activada) y pulsa **Run**.
5. Inicia sesión con **el mismo correo y contraseña** que tiene el ESP32 en `secrets.h`.

## 2. Poner el ESP32

1. Abre `esp32/despertador_esp32.ino` en Arduino IDE.
2. Copia `esp32/secrets.h.example` como `secrets.h` (misma carpeta que el `.ino`) y rellena tus datos.
3. Sube el sketch. En el Serial Monitor (115200) debe salir `Sesion Firebase iniciada` y `Alarmas leidas: N`.

## 3. Tarjeta SD del DFPlayer

Crea una carpeta **MP3** en la SD con los sonidos numerados:

```
/MP3/0001.mp3   Suave
/MP3/0002.mp3   Clásico
/MP3/0003.mp3   Campana
/MP3/0004.mp3   Gallo
/MP3/0005.mp3   Digital
```

Para escucharlos también en el celular, copia los mismos archivos a `app/src/main/res/raw/` como `alarma_1.mp3` ... `alarma_5.mp3` (minúsculas, sin espacios). Si faltan, la app funciona igual.

## 4. Datos en Firestore (rutas que usan la app y el ESP32)

| Ruta | Campos | Quién escribe |
|---|---|---|
| `users/{uid}/alarmas/{id}` | hora, minuto, activa, dias (bits L=1 … D=64), sonido (1-5), etiqueta, volumen (0-1), vibracion, posponer | app |
| `users/{uid}/config/luces` | color (0-4), modoNoche, luzNocturna, expresion | app |
| `users/{uid}/config/app` | notificaciones | app |
| `users/{uid}/estado/actual` | temp, hum, luz, aire, movimiento, ts | ESP32 |
| `users/{uid}/lecturas/{id}` | igual que estado | ESP32 |

Las reglas de Firestore que ya publicaste cubren todas estas rutas.

## Limitaciones actuales

- Las alarmas solo suenan con el ESP32 encendido y con WiFi (necesita hora por internet).
- `posponer` se guarda desde la app, pero el ESP32 todavía no lo usa. La cara del robot sí sigue la expresión elegida en la app (o es automática si eliges "Automático").
- Las notificaciones de calidad del aire solo guardan la preferencia; falta enviar el aviso al celular.
