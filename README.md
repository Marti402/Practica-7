# Practica-7 - Reproducción de audio con ESP32
Participants: Alexandre Pascual / Marti Vila

### Objetivo de la pràctica
En esta práctica se pretende trabajar con la reproducción de audio utilizando el microcontrolador ESP32. Se explora el uso de librerías específicas para reproducir archivos de audio tanto embebidos como desde una tarjeta SD.

## Parte 1

```c++
#include "AudioGeneratorAAC.h"
#include "AudioOutputI2S.h"
#include "AudioFileSourcePROGMEM.h"
#include "sampleaac.h"
AudioFileSourcePROGMEM *in;
AudioGeneratorAAC *aac;
AudioOutputI2S *out;
void setup(){
Serial.begin(115200);
in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac));
aac = new AudioGeneratorAAC();
out = new AudioOutputI2S();
out -> SetGain(0.125);
out -> SetPinout(7, 6, 4);
aac->begin(in, out);
}
void loop(){
if (aac->isRunning()) {
aac->loop();
} else {
aac -> stop();
Serial.printf("Sound Generator\n");
delay(1000);
}
}

```
### Explicación
En esta primera parte, el objetivo es reproducir un archivo de audio que está guardado en la memoria del propio microcontrolador utilizando *PROGMEM*. Se utilizan varias librerías específicas para manejar la decodificación AAC y la salida por el bus I2S.

* AudioFileSourcePROGMEM: Permite leer archivos de audio embebidos en la memoria flash.
* AudioGeneratorAAC: Se encarga de decodificar el archivo en formato AAC.
* AudioOutputI2S: Controla la salida de audio usando el protocolo I2S.
* sampleaac: Es el archivo de audio embebido.

El código configura los pines de salida para I2S (7, 6 y 4), inicia la reproducción, y continúa en bucle mientras se reproduce el audio. Al finalizar, se detiene y espera un segundo.


## Parte 2

```c++

#include "Audio.h"
#include "SD.h"
#include "FS.h"
// Digital I/O used
#define SD_CS 10
#define SPI_MOSI 11
#define SPI_MISO 13
#define SPI_SCK 12
#define I2S_DOUT 4
#define I2S_BCLK 7
#define I2S_LRC 6
Audio audio;
void setup(){
pinMode(SD_CS, OUTPUT);
digitalWrite(SD_CS, HIGH);
SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI);
Serial.begin(115200);
SD.begin(SD_CS);
audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT);
audio.setVolume(10); // 0...21
audio.connecttoFS(SD, "prova.mp3");
}
void loop(){
audio.loop();
}
// optional
void audio_info(const char *info){
Serial.print("info "); Serial.println(info);
}
void audio_id3data(const char *info){ //id3 metadata
Serial.print("id3data ");Serial.println(info);
}
void audio_eof_mp3(const char *info){ //end of file
Serial.print("eof_mp3 ");Serial.println(info);
}
void audio_showstation(const char *info){
Serial.print("station ");Serial.println(info);
}
void audio_showstreaminfo(const char *info){
  Serial.print("streaminfo ");Serial.println(info);
  }
  void audio_showstreamtitle(const char *info){
  Serial.print("streamtitle ");Serial.println(info);
  }
  void audio_bitrate(const char *info){
  Serial.print("bitrate ");Serial.println(info);
  }
  void audio_commercial(const char *info){ //duration in sec
  Serial.print("commercial ");Serial.println(info);
  }
  void audio_icyurl(const char *info){ //homepage
  Serial.print("icyurl ");Serial.println(info);
  }
  void audio_lasthost(const char *info){ //stream URL played
  Serial.print("lasthost ");Serial.println(info);
  }
  void audio_eof_speech(const char *info){
  Serial.print("eof_speech ");Serial.println(info);
  }

```

### Explicación
En esta segunda parte, reproducimos un archivo MP3 almacenado en una tarjeta SD. Configuramos manualmente los pines SPI para acceder a la tarjeta y los pines I2S para la salida de audio.

* SD.begin(SD_CS): Inicializa la tarjeta SD.
* connecttoFS(SD, "prova.mp3"): Establece conexión al archivo MP3 llamado prova.mp3.
* audio.loop(): Mantiene la reproducción en ejecución.


### Conclusión
Esta práctica permite familiarizarse con el manejo de audio en sistemas embebidos usando el ESP32. Se trabajan dos formas diferentes de reproducción:

* Desde la memoria flash (PROGMEM): útil para archivos pequeños y rápidos de reproducir sin necesidad de almacenamiento externo.
* Desde una tarjeta SD: ideal para almacenar archivos más grandes y reproducirlos directamente.

Ambos métodos usan el protocolo I2S, que es estándar en la salida de audio digital para microcontroladores.
Además, se implementan varias funciones opcionales para mostrar información adicional como bitrate, título, metadatos y estado del flujo de audio.
