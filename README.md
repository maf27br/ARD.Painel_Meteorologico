# Painel Meteorológico
Objetivo: Painel que mostre data, ora, temperatura, umidad, pressão e altitude, já praticando os aprendizados de utilização do display TFT 2.4.

Material:
  - Arduino Mega
  - Shield TFT 2.4 ILI9341, SD + Touch
  - RTC 1302
  - BMP180
  - DHT11
  - Protoboard e cabos
Montagem:
![20240414_165715](https://github.com/maf27br/ARD.Painel_Meteorologico/assets/68168344/ea04d255-1176-4c90-9e26-70adf3f0cca0)

![20240414_165709](https://github.com/maf27br/ARD.Painel_Meteorologico/assets/68168344/d5d2ae7b-3800-4b60-a58f-120c4203bab8)


Código:

```
#include <Adafruit_BMP085.h>
#include <Ds1302.h>
#include <DHT.h>
#include <Adafruit_GFX.h>
#include <SD.h>
#include <MCUFRIEND_kbv.h>
#include <SFE_BMP180.h>  //Biblioteca para sensor BMP180
#include <Wire.h>        //Biblioteca para comunicação I2C

#define SD_CS 10 // Card select for shield use
uint8_t spi_save;

MCUFRIEND_kbv tft;
//TEMPORIZADORES
unsigned long startMillis;
unsigned long currentMillis;

//DEFINIÇÃO DHT
const byte DHTPIN = A8;
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

//INICIALIZA SENSOR PRESSÃO/ALTITUDE BMP180
Adafruit_BMP085 bmp;

//DEFINIÇÃO CORES DISPLAY
#define BLACK 0x0000
#define NAVY 0x000F
#define DARKGREEN 0x03E0
#define DARKCYAN 0x03EF
#define MAROON 0x7800
#define PURPLE 0x780F
#define OLIVE 0x7BE0
#define LIGHTGREY 0xC618
#define DARKGREY 0x7BEF
#define BLUE 0x001F
#define GREEN 0x07E0
#define CYAN 0x07FF
#define RED 0xF800
#define MAGENTA 0xF81F
#define YELLOW 0xFFE0
#define WHITE 0xFFFF
#define ORANGE 0xFD20
#define GREENYELLOW 0xAFE5
#define PINK 0xF81F
//RTC
Ds1302 rtc(28, 26, 24);  // IO, SCLK, CE
// inicia estrutura de tempo (t)
const static char *DiasdaSemana[] = { "Segunda", "Terca", "Quarta", "Quinta", "Sexta", "Sabado", "Domingo" };

//DEFINIÇÃO RODAPÉ
const int rt_x = 120;
const int rt_y = 230;
const byte rt_tam = 1;
const int rt_cor = WHITE;
const int rt_tempo_refresh = 1000;

//DEFINIÇÃO PONTOS DIA DA SEMANA
const int dow_x = 16;
const int dow_y = 10;
const byte dow_tam = 2;
const int dow_cor = WHITE;
const int dow_tempo_refresh = 1000;

//DEFINIÇÃO PONTOS DATA
const int dt_x = 200;
const int dt_y = 10;
const byte dt_tam = 2;
const int dt_cor = WHITE;
const int dt_tempo_refresh = 1000;

//DEFINIÇÃO PONTOS MATRIZ HORA
const int hora_x = 20;
const int hora_y = 80;
const byte hora_tam = 6;
const int hora_cor = GREENYELLOW;
const int hora_tempo_refresh = 1000;

//DEFINIÇÃO PONTOS TEMP
const int temp_x = 40;
const int temp_y = 160;
const byte temp_tam = 3;
const int temp_cor = WHITE;
const int temp_tempo_refresh = 1000;

//DEFINIÇÃO PONTOS HUMIDADE
const int hum_x = 155;
const int hum_y = 160;
const byte hum_tam = 3;
const int hum_cor = WHITE;
const int hum_tempo_refresh = 1000;

//DEFINIÇÃO PONTOS PRESSAO
const int pre_x = 235;
const int pre_y = 160;
const byte pre_tam = 2;
const int pre_cor = WHITE;
const int pre_tempo_refresh = 1000;

//DEFINIÇÃO PONTOS ALT
const int alt_x = 235;
const int alt_y = 180;
const byte alt_tam = 2;
const int alt_cor = WHITE;
const int alt_tempo_refresh = 1000;

void setup(void) {
  Serial.begin(9600);
  startMillis = millis();
  //INICIALIZA RELOGIO
  rtc.init();
  /*
  Ds1302::DateTime dt;
  dt.year = 24;
  dt.month = 4;
  dt.day = 7;
  dt.dow = 7;
  dt.hour = 22;
  dt.minute = 21;
  dt.second = 0;
  rtc.setDateTime(&dt);
*/
  //BMP180
  if (!bmp.begin()) {                                                              //SE O SENSOR NÃO FOR INICIALIZADO, FAZ
    Serial.println("Sensor BMP180 não foi identificado! Verifique as conexões.");  //IMPRIME O TEXTO NO MONITOR SERIAL
  }
  //DHT
  dht.begin();
  uint16_t ID = tft.readID();
  Serial.print("TFT ID = 0x");
  Serial.println(ID, HEX);
  if (ID == 0xD3D3) ID = 0x9486;  // write-only shield
  tft.begin(ID);
  tft.setRotation(1);  //PORTRAIT
  Serial.print(F("Initializing SD card..."));
  //FUNDO
  tft.fillScreen(BLACK);
  //BORDA
  tft.drawRoundRect(2, 2, 316, 238, 15, GREENYELLOW);
  if (!SD.begin(SD_CS)) {
    Serial.println(F("failed!"));
    return;
  }
  Serial.println(F("OK!"));
  spi_save = SPCR;
  //Serial.println(tft.width());
  //Serial.println(tft.height());
}

void loop(void) {
  currentMillis = millis();
  if (currentMillis - startMillis >= hora_tempo_refresh) {
    exibe_diadasemana(dow_x, dow_y, dow_tam, dow_cor, exibe_data_hora(2));
    exibe_dia(dt_x, dt_y, dt_tam, dt_cor, exibe_data_hora(1));
    exibe_hora(hora_x, hora_y, hora_tam, hora_cor, exibe_data_hora(0));
    exibe_vlrtemp(temp_x, temp_y, temp_tam, temp_cor);
    exibe_vlrhum(hum_x, hum_y, hum_tam, hum_cor);
    exibe_vlrPressao(pre_x, pre_y, pre_tam, pre_cor);
    exibe_vlrAlt(alt_x, alt_y, alt_tam, alt_cor);
    exibe_RunningTime(rt_x, rt_y, rt_tam, rt_cor, runTime());
    //Serial.println();  //QUEBRA DE LINHA NA SERIAL
    startMillis = currentMillis;
  }
}

void exibe_hora(int x, int y, byte tam, int cor, String texto) {
  tft.setCursor(x, y);
  tft.setTextColor(cor, BLACK);
  tft.setTextSize(tam);
  tft.print(texto);
}

void exibe_RunningTime(int x, int y, byte tam, int cor, String texto) {
  tft.setCursor(x, y);
  tft.setTextColor(cor, BLACK);
  tft.setTextSize(tam);
  tft.print(texto);
}

void exibe_diadasemana(int x, int y, byte tam, int cor, String texto) {
  tft.setCursor(x, y);
  tft.setTextColor(cor, BLACK);
  tft.setTextSize(tam);
  tft.print(texto);
}

void exibe_dia(int x, int y, byte tam, int cor, String texto) {
  tft.setCursor(x, y);
  tft.setTextColor(cor, BLACK);
  tft.setTextSize(tam);
  tft.print(texto);
}

String runTime(void) {
  unsigned long runMillis = millis();
  unsigned long allSeconds = millis() / 1000;
  int runHours = allSeconds / 3600;
  int secsRemaining = allSeconds % 3600;
  int runMinutes = secsRemaining / 60;
  int runSeconds = secsRemaining % 60;
  int runDays = runHours / 24;

  char buf[45];
  sprintf(buf, "tempo ligado: %d dias, %02d:%02d:%02d", runDays, runHours, runMinutes, runSeconds);
  return (buf);
}

void exibe_vlrtemp(int x, int y, byte tam, int cor) {
  tft.setCursor(x, y);
  tft.setTextColor(cor, BLACK);
  tft.setTextSize(tam);
  int temp = dht.readTemperature();
  char buf[2];
  sprintf(buf, "%d", temp);
  tft.print(buf);
  tft.setTextSize(2);
  tft.write(0xF7);
  tft.setTextSize(tam);
  tft.print("C");
  //barra de temperatura
  int temp_range = map(temp, 5, 45, 0, 3);
  switch (temp_range) {
    case 0:
      tft.fillRect(x, y + 24, 72, 12, WHITE);
      break;
    case 1:
      tft.fillRect(x, y + 24, 72, 12, YELLOW);
      break;
    case 2:
      tft.fillRect(x, y + 24, 72, 12, ORANGE);
      break;
    case 3:
      tft.fillRect(x, y + 24, 72, 12, RED);
      break;
    default:
      tft.fillRect(x, y + 24, 72, 12, NAVY);
      break;
  }
  bmpDraw("t.bmp", x - 36, y );
}

void exibe_vlrhum(int x, int y, byte tam, int cor) {
  tft.setCursor(x + 8, y);
  tft.setTextColor(cor, BLACK);
  tft.setTextSize(tam);
  int hum = dht.readHumidity();
  char buf[2];
  sprintf(buf, "%d", hum);
  tft.print(buf);
  tft.setTextSize(2);
  tft.print("%");
  //barra de temperatura
  //Serial.println(hum);
  //Serial.println(hum_range);

  if (hum <= 40) {
    tft.fillRect(x, y + 24, 72, 12, ORANGE);
  } else if (hum <= 70) {
    tft.fillRect(x, y + 24, 72, 12, GREEN);
  } else {
    tft.fillRect(x, y + 24, 72, 12, BLUE);
  }
  bmpDraw("h.bmp", x - 36, y);  
}

void exibe_vlrPressao(int x, int y, byte tam, int cor) {
  tft.setCursor(x, y);
  tft.setTextColor(cor, BLACK);
  tft.setTextSize(tam);
  int pressao = bmp.readPressure() / 100;
  char buf[8];
  //Serial.println(pressao);
  sprintf(buf, "%d hPa", pressao);
  tft.print(buf);
}

void exibe_vlrAlt(int x, int y, byte tam, int cor) {
  tft.setCursor(x, y);
  tft.setTextColor(cor, BLACK);
  tft.setTextSize(tam);
  int altitude = bmp.readAltitude(101325);
  char buf[8];
  //Serial.println(altitude);
  sprintf(buf, "%d m", altitude);
  tft.print(buf);
}

String exibe_data_hora(byte tp_retorno) {
  //pega a data
  Ds1302::DateTime now;
  char buf[8];
  rtc.getDateTime(&now);
  switch (tp_retorno) {
    case 0:
      sprintf(buf, "%02d:%02d:%02d", now.hour, now.minute, now.second);
      return buf;
      break;
    case 1:
      sprintf(buf, "%02d/%02d/%02d", now.day, now.month, now.year);
      return buf;
      break;
    case 2:
      return DiasdaSemana[now.dow - 1];
      break;
  }
}

void exibe_data_serial() {
  //pega a data
  Ds1302::DateTime now;
  rtc.getDateTime(&now);
  Serial.print("Data: ");
  if (now.day < 10) Serial.print('0');  //Se o dia do mês for menor que 10, imprime 0 à frente
  Serial.print(now.day);                //Imprime o dia do mês (1-31)
  Serial.print('/');
  if (now.month < 10) Serial.print('0');  //Se o mês for menor que 10, imprime 0 à frente
  Serial.print(now.month);                // Imprime o mês (1-12)
  Serial.print('/');
  Serial.print("20");      //Imprime 20
  Serial.print(now.year);  //Imprime o ano (0-99)
  Serial.print(' ');
  //Imprime o dia da semana (Segunda, Terça,..., Domingo)
  Serial.print("- Dia: ");
  Serial.print(DiasdaSemana[now.dow - 1]);  //Imprime o dia da semana
  Serial.print(' ');
  //Imprime a hora atual (hh:mm:ss)
  Serial.print("- Hora: ");
  if (now.hour < 10) Serial.print('0');  //Se a hora for menor que 10,  imprime 0 à frente
  Serial.print(now.hour);                //Imprime hora atual(0 – 23)
  Serial.print(':');
  if (now.minute < 10) Serial.print('0');  //Se o minuto for menor que 10, imprime 0 à frente
  Serial.print(now.minute);                //Imprime os segundos (0-59)
  Serial.print(':');
  if (now.second < 10) Serial.print('0');  //Se o segundo for menor que 10, imprime 0 à frente
  Serial.print(now.second);                //Imprime os segundos (0-59)
  Serial.println();
}

//Função desenho BMP
#define BUFFPIXEL 20

void bmpDraw(char *filename, int x, int y) {
  File bmpFile;
  int bmpWidth, bmpHeight;             // W+H in pixels
  uint8_t bmpDepth;                    // Bit depth (currently must be 24)
  uint32_t bmpImageoffset;             // Start of image data in file
  uint32_t rowSize;                    // Not always = bmpWidth; may have padding
  uint8_t sdbuffer[3 * BUFFPIXEL];     // pixel in buffer (R+G+B per pixel)
  uint16_t lcdbuffer[BUFFPIXEL];       // pixel out buffer (16-bit per pixel)
  uint8_t buffidx = sizeof(sdbuffer);  // Current position in sdbuffer
  boolean goodBmp = false;             // Set to true on valid header parse
  boolean flip = true;                 // BMP is stored bottom-to-top
  int w, h, row, col;
  uint8_t r, g, b;
  uint32_t pos = 0, startTime = millis();
  uint8_t lcdidx = 0;
  boolean first = true;

  if ((x >= tft.width()) || (y >= tft.height())) return;

  Serial.println();
  Serial.print("Loading image '");
  Serial.print(filename);
  Serial.println('\'');
  // Open requested file on SD card
  SPCR = spi_save;
  if ((bmpFile = SD.open(filename)) == NULL) {
    Serial.print("File not found");
    return;
  }

  // Parse BMP header
  if (read16(bmpFile) == 0x4D42) {  // BMP signature
    Serial.print(F("File size: "));
    Serial.println(read32(bmpFile));
    (void)read32(bmpFile);             // Read & ignore creator bytes
    bmpImageoffset = read32(bmpFile);  // Start of image data
    Serial.print(F("Image Offset: "));
    Serial.println(bmpImageoffset, DEC);
    // Read DIB header
    Serial.print(F("Header size: "));
    Serial.println(read32(bmpFile));
    bmpWidth = read32(bmpFile);
    bmpHeight = read32(bmpFile);
    if (read16(bmpFile) == 1) {    // # planes -- must be '1'
      bmpDepth = read16(bmpFile);  // bits per pixel
      Serial.print(F("Bit Depth: "));
      Serial.println(bmpDepth);
      if ((bmpDepth == 24) && (read32(bmpFile) == 0)) {  // 0 = uncompressed

        goodBmp = true;  // Supported BMP format -- proceed!
        Serial.print(F("Image size: "));
        Serial.print(bmpWidth);
        Serial.print('x');
        Serial.println(bmpHeight);

        // BMP rows are padded (if needed) to 4-byte boundary
        rowSize = (bmpWidth * 3 + 3) & ~3;

        // If bmpHeight is negative, image is in top-down order.
        // This is not canon but has been observed in the wild.
        if (bmpHeight < 0) {
          bmpHeight = -bmpHeight;
          flip = false;
        }

        // Crop area to be loaded
        w = bmpWidth;
        h = bmpHeight;
        if ((x + w - 1) >= tft.width()) w = tft.width() - x;
        if ((y + h - 1) >= tft.height()) h = tft.height() - y;

        // Set TFT address window to clipped image bounds
        SPCR = 0;
        tft.setAddrWindow(x, y, x + w - 1, y + h - 1);

        for (row = 0; row < h; row++) {  // For each scanline...
          // Seek to start of scan line.  It might seem labor-
          // intensive to be doing this on every line, but this
          // method covers a lot of gritty details like cropping
          // and scanline padding.  Also, the seek only takes
          // place if the file position actually needs to change
          // (avoids a lot of cluster math in SD library).
          if (flip)  // Bitmap is stored bottom-to-top order (normal BMP)
            pos = bmpImageoffset + (bmpHeight - 1 - row) * rowSize;
          else  // Bitmap is stored top-to-bottom
            pos = bmpImageoffset + row * rowSize;
          SPCR = spi_save;
          if (bmpFile.position() != pos) {  // Need seek?
            bmpFile.seek(pos);
            buffidx = sizeof(sdbuffer);  // Force buffer reload
          }

          for (col = 0; col < w; col++) {  // For each column...
            // Time to read more pixel data?
            if (buffidx >= sizeof(sdbuffer)) {  // Indeed
              // Push LCD buffer to the display first
              if (lcdidx > 0) {
                SPCR = 0;
                tft.pushColors(lcdbuffer, lcdidx, first);
                lcdidx = 0;
                first = false;
              }
              SPCR = spi_save;
              bmpFile.read(sdbuffer, sizeof(sdbuffer));
              buffidx = 0;  // Set index to beginning
            }

            // Convert pixel from BMP to TFT format
            b = sdbuffer[buffidx++];
            g = sdbuffer[buffidx++];
            r = sdbuffer[buffidx++];
            lcdbuffer[lcdidx++] = tft.color565(r, g, b);
          }  // end pixel
        }    // end scanline
        // Write any remaining data to LCD
        if (lcdidx > 0) {
          SPCR = 0;
          tft.pushColors(lcdbuffer, lcdidx, first);
        }
        Serial.print(F("Loaded in "));
        Serial.print(millis() - startTime);
        Serial.println(" ms");
      }  // end goodBmp
    }
  }

  bmpFile.close();
  if (!goodBmp) Serial.println("BMP format not recognized.");
}

// These read 16- and 32-bit types from the SD card file.
// BMP data is stored little-endian, Arduino is little-endian too.
// May need to reverse subscript order if porting elsewhere.

uint16_t read16(File f) {
  uint16_t result;
  ((uint8_t *)&result)[0] = f.read();  // LSB
  ((uint8_t *)&result)[1] = f.read();  // MSB
  return result;
}

uint32_t read32(File f) {
  uint32_t result;
  ((uint8_t *)&result)[0] = f.read();  // LSB
  ((uint8_t *)&result)[1] = f.read();
  ((uint8_t *)&result)[2] = f.read();
  ((uint8_t *)&result)[3] = f.read();  // MSB
  return result;
}

```
