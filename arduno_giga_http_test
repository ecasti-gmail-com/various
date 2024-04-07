#include <WiFi.h>
#include <ArduinoHttpClient.h>
#include "Arduino_GigaDisplay_GFX.h"
#include "Arduino_GigaDisplayTouch.h"
#include <JPEGDecoder.h>
#define minimum(a, b) (((a) < (b)) ? (a) : (b))
#define WHITE 0xffff
#define BLACK 0x0000
#define BUFSIZE 10240
#define BUFSIZ 10240
#define LOGGING true
const char *ssid = "<YOUR-SSID>";
const char *password = "<YOUR_KEY>";

uint8_t buff[40000] = { 0 };
int len;
String buff_str;
GigaDisplay_GFX display;
WiFiClient client;
HttpClient httpclient = HttpClient(client, "<YOUR-HTTP-IMAGE>", 80);


void copy_data() {
  int i;
  for (i = 0; i <= len; i++) {
    buff[i] = (int)buff_str[i];
  }
}
void get_data() {
  uint32_t initime = millis();
  uint32_t time1, time2, time3, fintime;
  Serial.println("making GET request");
  httpclient.get("/static/images/logo-text.svg");
  httpclient.connectionKeepAlive();
  time1 = millis();
  // read the status code and body of the response
  int statusCode = httpclient.responseStatusCode();
  time2 = millis();
  Serial.print("Status Code:");
  Serial.println(statusCode);
  buff_str = httpclient.responseBody();
  fintime = millis();
  Serial.println("=====================================");
  Serial.println("Total download time was    : ");
  Serial.print("Get: ");
  Serial.println(time1 - initime);
  Serial.print("Status: ");
  Serial.println(time2 - time1);
  Serial.print("Body: ");
  Serial.println(fintime - time2);
  Serial.print("Total: ");
  Serial.println(fintime - initime);
  Serial.println("=====================================");
  httpclient.completed();
  copy_data();
  
  len = httpclient.contentLength();
   Serial.println(len);
}

void drawjpg() {
  boolean decoded = JpegDec.decodeArray(buff, len);
  if (decoded) {
    jpegRender(0, 0);
    // jpegInfo();
  } else {

    Serial.println("Jpeg file format not supported!");
  }
}

void jpegRender(int xpos, int ypos) {

  // retrieve infomration about the image
  uint16_t *pImg;
  uint16_t mcu_w = JpegDec.MCUWidth;
  uint16_t mcu_h = JpegDec.MCUHeight;
  uint32_t max_x = JpegDec.width;
  uint32_t max_y = JpegDec.height;

  // Jpeg images are draw as a set of image block (tiles) called Minimum Coding Units (MCUs)
  // Typically these MCUs are 16x16 pixel blocks
  // Determine the width and height of the right and bottom edge image blocks
  uint32_t min_w = minimum(mcu_w, max_x % mcu_w);
  uint32_t min_h = minimum(mcu_h, max_y % mcu_h);

  // save the current image block size
  uint32_t win_w = mcu_w;
  uint32_t win_h = mcu_h;

  // record the current time so we can measure how long it takes to draw an image
  uint32_t drawTime = millis();

  // save the coordinate of the right and bottom edges to assist image cropping
  // to the screen size
  max_x += xpos;
  max_y += ypos;

  // read each MCU block until there are no more
  while (JpegDec.read()) {

    // save a pointer to the image block
    pImg = JpegDec.pImage;

    // calculate where the image block should be drawn on the screen
    int mcu_x = JpegDec.MCUx * mcu_w + xpos;
    int mcu_y = JpegDec.MCUy * mcu_h + ypos;

    // check if the image block size needs to be changed for the right edge
    if (mcu_x + mcu_w <= max_x) win_w = mcu_w;
    else win_w = min_w;

    // check if the image block size needs to be changed for the bottom edge
    if (mcu_y + mcu_h <= max_y) win_h = mcu_h;
    else win_h = min_h;

    // copy pixels into a contiguous block
    if (win_w != mcu_w) {
      for (int h = 1; h < win_h - 1; h++) {
        memcpy(pImg + h * win_w, pImg + (h + 1) * mcu_w, win_w << 1);
      }
    }


    // draw image MCU block only if it will fit on the screen
    //  if ( ( mcu_x + win_w) <= tft.width() && ( mcu_y + win_h) <= tft.height())
    if ((mcu_x + win_w) <= 320 && (mcu_y + win_h) <= 240) {
      display.drawRGBBitmap(mcu_x, mcu_y, pImg, win_w, win_h);
    }

    // Stop drawing blocks if the bottom of the screen has been reached,
    // the abort function will close the file
    else if ((mcu_y + win_h) >= 240)
      JpegDec.abort();
  }

  // calculate how long it took to draw the image
  drawTime = millis() - drawTime;

  // print the results to the serial port
  Serial.print("Total render time was    : ");
  Serial.print(drawTime);
  Serial.println(" ms");
  Serial.println("=====================================");
}

//====================================================================================
//   Send time taken to Serial port
//====================================================================================
void jpegInfo() {
  Serial.println(F("==============="));
  Serial.println(F("JPEG image info"));
  Serial.println(F("==============="));
  Serial.print(F("Width      :"));
  Serial.println(JpegDec.width);
  Serial.print(F("Height     :"));
  Serial.println(JpegDec.height);
  Serial.print(F("Components :"));
  Serial.println(JpegDec.comps);
  Serial.print(F("MCU / row  :"));
  Serial.println(JpegDec.MCUSPerRow);
  Serial.print(F("MCU / col  :"));
  Serial.println(JpegDec.MCUSPerCol);
  Serial.print(F("Scan type  :"));
  Serial.println(JpegDec.scanType);
  Serial.print(F("MCU width  :"));
  Serial.println(JpegDec.MCUWidth);
  Serial.print(F("MCU height :"));
  Serial.println(JpegDec.MCUHeight);
  Serial.println(F("==============="));
}

void setup() {
  Serial.begin(115200);
  display.begin();
  display.setRotation(1);
  display.fillScreen(BLACK);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.println("Connecting...");
  }
  Serial.println("Connected!");
  // print the SSID of the network you're attached to:
  Serial.print("SSID: ");
  Serial.println(WiFi.SSID());
  //Serial.println(httpclient.BUFSIZ );
  // print your WiFi shield's IP address:
  IPAddress ip = WiFi.localIP();
  Serial.print("IP Address: ");
  Serial.println(ip);
  httpclient.connectionKeepAlive();
}

void loop() {
  get_data();
  drawjpg();
  // delay(2000);
}
