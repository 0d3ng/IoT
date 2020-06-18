# Pengamanan Data IoT

## Tujuan
- Mampu memahami konsep enkripsi data pada arsitektur IoT
- Mampu membuat program untuk menyandikan data di sisi MCU yang akan dikirimkan ke IoT Server
- Mampu menampilkan data tersandikan di sisi IoT Server

## Capaian
- Mengirim data sensor yang dienkripsi dari sisi MCU untuk keamanan data
- Menampilkan hasil data yang telah tersandikan ke IoT Server

## Teori Singkat
Konsep secara umum IoT adalah dengan menghubungkan sebuah objek agar mengerjakan sesuatu lebih mudah dan lebih efisien, membuat hal secara otomatis hal-hal yang membosankan, menggunakan sumber daya yang lebih efisien, dan umumnya digunakan untuk meningkatkan kualitas hidup kita agar lebih baik. Namun, ketika tidak dikelola dengan baik, IoT berpotensi membuat segalanya menjadi buruk dari sisi persepsi privasi dan keamanan data.

Salah satu cara yang dapat digunakan untuk mengamankan data adalah dengan melakukan pengamanan data yang dihasilkan dari perangkat IoT. Misalkan dengan mengacak data agar tidak dapat dengan mudah dibaca ketika data tersebut jatuh pada pihak yang tidak bertanggungjawab. Enkripsi AES dapat digunakan untuk melakukan hal tersebut, AES atau kepanjangan dari Advanced Encryption Standard dengan menggunakan kunci simetris. Kunci simetris berarti kunci untuk melakukan enkripsi(mengubah data dalam bentuk tidak mudah dibaca) dengan proses dekrip sama.

Jenis-jenis AES yang dapat digunakan adalah AES-128, AES-192, dan AES-256. 128, 192, dan 256 adalah menggambarkan ukuran key ketika proses enkripsi.

## Praktikum
Pada praktikum kali ini kita akan mengirimkan data yang terenkripsi AES menggunakan ESP8266, kemudian server yang telah kita bangun sebelumnya melakukan data dekripsi. 

Sebelum membuat program enkripsi, silakan ditambahkan terlebih dahulu file-file berikut ini
- `AES.cpp`
- `AES.h`
- `AES_config.h`
- `Base64.cpp`
- `Base64.h`
> Ketika menggunakan Arduino IDE, masukkan file-file tersebut pada folder installasi arduino - buat folder pada `libraries`

Selain library di atas, dibutuhkan juga library `ArduinoJson`. Silakan intall library tersebut menggunakan library manager `Arduino IDE` atau secara manual dengan mengunduh di [https://github.com/bblanchon/ArduinoJson.git](https://github.com/bblanchon/ArduinoJson.git). Setelah berhasil didownload copy-kan pada folder `libraries`.

### Enkripsi AES pada ESP8266
Buatlah kode di bawah ini menggunakan editor Anda
```cpp
#include <Arduino.h>
#include <ESP8266WiFi.h>

#include <ArduinoJson.h>

#include "AES.h"
#include "base64.h"
#include "AES_config.h"

byte my_iv[N_BLOCK] = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};
byte key[] = {0x48, 0x48, 0x48, 0x48, 0x48, 0x48, 0x48, 0x48, 0x48, 0x48, 0x48, 0x48, 0x48, 0x48, 0x48, 0x48};

const char *ssid = "SSID"; //ganti dengan ssid Anda
const char *password = "PASSWORD"; //ganti dengan password ssid Anda
const uint16_t port = 2004; // port server
const char *host = "192.168.0.100"; // host server

WiFiClient client;

void connect_wifi()
{
  Serial.printf("Connecting to %s ", ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
  Serial.println(" connected");
  delay(250);
}

void connect_server()
{
  while (!client.connect(host, port))
  {
    Serial.printf("\n[Connecting to %s ... ", host);
    delay(1000);
    return;
  }
  Serial.println("connected]");
  delay(1000);
}

uint8_t getrnd()
{
  uint8_t really_random = *(volatile uint8_t *)0x3FF20E44;
  return really_random;
}

void gen_iv(byte *iv)
{
  for (int i = 0; i < N_BLOCK; i++)
  {
    iv[i] = (byte)getrnd();
  }
}

String do_encrypt(String msg, byte *key)
{
  size_t encrypt_size_len = 2000;
  DynamicJsonDocument root(1024);

  char *b64data = new char[encrypt_size_len];
  byte *cipher = new byte[encrypt_size_len];

  AES aes;

  aes.set_key(key, sizeof(key));
  gen_iv(my_iv);

  memset(b64data, 0, encrypt_size_len);

  //IVbase64
  base64_encode(b64data, (char *)my_iv, N_BLOCK);
  root["iv"] = String(b64data);

  memset(b64data, 0, encrypt_size_len);
  memset(cipher, 0, encrypt_size_len);

  //msg base64
  int b64len = base64_encode(b64data, (char *)msg.c_str(), msg.length());

  //AES128，IV，CBCpkcs7
  aes.do_aes_encrypt((byte *)b64data, b64len, cipher, key, 128, my_iv);
  aes.clean();

  memset(b64data, 0, encrypt_size_len);

  base64_encode(b64data, (char *)cipher, aes.get_size());
  root["msg"] = String(b64data);

  String JsonBuff;
  serializeJson(root, JsonBuff);
  root.clear();

  delete[] b64data;
  delete[] cipher;

  return JsonBuff;
}

void setup()
{
  Serial.begin(115200);
  Serial.println("Contoh penggunaan socket client");
  connect_wifi();
  connect_server();
}

void loop()
{
  if (client.connected())
  {
    Serial.print("[Sending a request]: ");
    String data = do_encrypt("Polinema Yes", key);
    client.print(data);
    Serial.println(data);
  }
  else
  {
    connect_server();
  }
  delay(3000);
}
```

Kode di atas digunakan untuk mengirimkan message data yang terenkripsi pada sebuah server menggunakan protokol socket yang sebelumnya telah kita praktikan. Data yang dienkripsi sebelumnya dilakukan encode menggunakan base64 dan menggunakan AES-128.

Jalankan kode di atas pada ESP8266, kemudian silakan diamati menggunakan `Serial Monitor`.

### Enkripsi pada Server

Jika pada program yang sebelumnya adalah proses enkripsi pada sisi ESP8266, selanjutnya kita perlu membuat kode program pada sisi server socket. Kode program di bawah ini adalah modifikasi yang sebelumnya tentang server socket. Buat file `aes_example.py` yang isinya adalah sebagai berikut

> Jangan lupa untuk menambahkan library `pycrypto`, untuk installasi library salah satu cara yang dapat dilakukan menggunakan perintah `pip install pycrypto`

```python
import sys
import json
import base64
import random
from Crypto.Cipher import AES

AES128_key = '48484848484848484848484848484848'

class AESEncrypter(object):
    def __init__(self, key, iv=None):
        self.key = key
        self.iv = iv if iv else bytes(key[0:16], 'utf-8')

    def _pad(self, text):
        text_length = len(text)
        padding_len = AES.block_size - int(text_length % AES.block_size)
        if padding_len == 0:
            padding_len = AES.block_size
        t2 = chr(padding_len) * padding_len
        t2 = t2.encode('utf-8')
        # print('text ', type(text), text)
        # print('t2 ', type(t2), t2)
        t3 = text + t2
        return t3

    def _unpad(self, text):
        text_length = len(text)
        padding_len = int(text_length % AES.block_size)
        if padding_len != 0:
            pad = ord(text[-1])
            return text[:-pad]
        else:
            return text

    def _decode_base64(self, data):
        """
        Decode base64, padding being optional.
        :param data: Base64 data as an ASCII byte string
        :returns: The decoded byte string.
        """
        missing_padding = len(data) % 4
        if missing_padding != 0:
            data += b'=' * (4 - missing_padding)
        return base64.b64decode(data)

    def encrypt(self, raw):
        raw = raw.encode('utf-8')
        raw = self._pad(raw)
        cipher = AES.new(self.key, AES.MODE_CBC, self.iv)
        encrypted = cipher.encrypt(raw)
        return base64.b64encode(encrypted).decode('utf-8')

    def decrypt(self, enc):
        enc = enc.encode("utf-8")
        enc = base64.b64decode(enc)
        cipher = AES.new(self.key, AES.MODE_CBC, self.iv)
        decrypted = cipher.decrypt(enc)
        decrypted = self._unpad(decrypted.decode('utf-8'))
        decrypted = self._decode_base64(decrypted.encode('utf-8')).decode('utf-8')
        return decrypted


def JsonToPlaintext(esp8266_json):
    try:
        esp8266_data = json.loads(esp8266_json)
    except Exception as e:
        print('Json，data:{},error：{}'.format(esp8266_data, e))
        return
    iv = base64.b64decode(esp8266_data['iv'])  # base64
    cipher = AESEncrypter(bytes.fromhex(AES128_key), iv)
    return cipher.decrypt(esp8266_data['msg'])


def PlaintextToJson(Plaintext):
    esp8266_iv = ''
    for i in range(32):
        esp8266_iv += (random.choice('0123456789abcdef'))
        # esp8266_iv = '00000000000000000000000000000000'
    iv = bytes.fromhex(esp8266_iv)
    esp8266_iv = str(base64.b64encode(iv), 'utf-8')
    b64msg = base64.b64encode(Plaintext.encode('utf-8')).decode('utf-8')
    print('b64msg: %s' % b64msg)
    cipher = AESEncrypter(bytes.fromhex(AES128_key), iv)
    encrypted = cipher.encrypt(b64msg)
    # print('Encrypted: %s' % encrypted)
    esp8266_send_json = {"iv": "%s" % esp8266_iv, "msg": "%s" % encrypted}
    return json.dumps(esp8266_send_json)


def print_hex(bytes):
    l = [hex(int(i)) for i in bytes]
    print(" ".join(l))


if __name__ == "__main__":
    try:
        # for AES test

        print('decrypt：')
        esp8266_data = '{"iv":"E9Mhq/WDtSZkrfGhHDJSRg==","msg":"XHao0wiSLEwegeKIIfmd6YprWYn4tAKjBHE3zKM9P9I="}'
        print('data: %s' % esp8266_data)
        print('Decrypted: %s' % JsonToPlaintext(esp8266_data))

        print('encrypt：')
        msg = 'Hello Word Hello Word'
        print('esp8266 json: %s' % PlaintextToJson(msg))

    except KeyboardInterrupt:
        sys.exit(0)
```

Kelas di atas merupakan fungsi untuk decrypt, gabungkan pada class yang menghandle server socket dari program yang telah buat sebelumnya.

```python
import socket
from threading import Thread
from time import *
from aes_example import *

# Multithreaded Python server
class ClientThread(Thread):

    def __init__(self, ip, port):
        Thread.__init__(self)
        self.ip = ip
        self.port = port
        print("Incoming connection from " + ip + ":" + str(port))

    def run(self):
        while True:
            try:
                data = conn.recv(2048)
                if len(data) == 0:
                    break
                print("length: " + str(len(data)))
                print("Server received data:", data)
                msg = data.decode("utf-8")
                print('Decrypted: %s' % JsonToPlaintext(msg))

            except Exception as e:
                print(e)
                break
            sleep(0.25)


TCP_IP = "192.168.0.100"
TCP_PORT = 2004
BUFFER_SIZE = 20

tcpServer = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcpServer.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
tcpServer.bind((TCP_IP, TCP_PORT))
threads = []

while True:
    tcpServer.listen(4)
    print("Server started on " + TCP_IP + " port " + str(TCP_PORT))
    (conn, (ip, port)) = tcpServer.accept()
    newthread = ClientThread(ip, port)
    newthread.start()
    threads.append(newthread)

for t in threads:
    t.join()
```

Silakan dijalankan kode python tersebut, seharusnya nanti akan muncul keluaran seperti ditunjukkan di bawah ini
* Keluaran ESP8266, `Serial Monitor` 
    
    <pre>[Sending a request]: {"iv":"+JSiRDx+CHZlHyhjQ0hgLA==","msg":"vz85bdEM4KikxyK1bE2ty5FFAjH7hX8f5MgaGzOX35g="}
    [Sending a request]: {"iv":"Y+FK38uSH2jFxfEbJEVCzQ==","msg":"xDJUyk6SpzWplbEQlD26v1r0mhISlvqMeFEuCSQDS2s="}
    [Sending a request]: {"iv":"qcMgC6bmoS4I2525//E7xg==","msg":"zFXPfsvCRnPb76nnP5dsPhGWTdX1bfkbDsJV1K5ChKs="}
    [Sending a request]: {"iv":"uzEwC+mJqI5H41qw8dqjfQ==","msg":"J87YZCWNPJCRsP5dcdFGMA3Lvos+4BoJTpnRcdECF0E="}
    [Sending a request]: {"iv":"NxyyZ4iuFyn25yulTaZyzA==","msg":"HtV3TmI2CQqdcUxu1gY8UXegncFnZLeg0rcsxenxMIU="}</pre>    

* Keluaran Server, `Socket Server`

    <pre>Server started on 192.168.0.100 port 2004
    Incoming connection from 192.168.0.102:52120
    Server started on 192.168.0.100 port 2004
    length: 86
    Server received data: b'{"iv":"1G/XBpSm74YTOPYeYDDv+Q==","msg":"/Zm65menSxcTsPmsk2PwBhl/EIB9wBAoOW43w5MDgu0="}'
    Decrypted: Polinema Yes
    length: 86
    Server received data: b'{"iv":"ou4v7V2UC8JHzUyUTrj7gA==","msg":"nzfrB+aODLLQPEfrPeor3pwlH+6Xd5TMXocIzr9bLRI="}'
    Decrypted: Polinema Yes
    length: 86
    Server received data: b'{"iv":"L4vYlzrrmor0amidDNSIOw==","msg":"B9BZ/OTE/RKwc21x6K93KtpOhabmgbrKUf+s/4cpaOE="}'
    Decrypted: Polinema Yes
    length: 86
    Server received data: b'{"iv":"vD2IEzQnokG5VgLt+xZ/3w==","msg":"gkWwtJzVde1OSOvl0l3QgcRxrB5NHV4QHb8nz1GIANg="}'
    Decrypted: Polinema Yes
    length: 86
    Server received data: b'{"iv":"gju8v7DzFjhCdEU/kq6Y9Q==","msg":"wss6Vk89oTOWcZ3wIttJ29epQrZfjQiffaDfj8puo6w="}'
    Decrypted: Polinema Yes</pre>

Keluaran tersebut jika dilihat dengan teliti, messagenya tidak selalu berurutan karena stream di socket server ada waktu tunggu sehingga ketika keluarnya tidak berurutan. Silakan dilakukan explorasi program di atas misalkan dengan mengubah key atau parameter yang lain.

> Ketika implementasi di dunia nyata Anda harus mengubah key tersebut di atas agar lebih aman.

## Tugas
Modifikasi program pada praktikum di atas dengan mengenkripsi data sensor cahaya dan sensor suhu. Kemudian ditampilkan juga hasil dekrip pada bagian socket servernya.