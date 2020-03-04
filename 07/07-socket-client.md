# Membuat program socket client

## Tujuan
- Mampu memahami protokol komunikasi socket client TCP/IP
- Mampu membuat program komunikasi antara MCU sebagai client socket dan komputer sebagai server socket dengan C#
- Mampu membuat program untuk mengirim data sensor dari MCU ke komputer secara real-time dengan protokol komunikasi socket TCP/IP

## Capaian
- Menjelaskan konsep komunikasi TCP/IP
- Menjelaskan konsep protokol Socket TCP/IP agar MCU dapat berkomunikasi dengan perangkat lainnya
- Membuat program socket client di sisi MCU yang bertugas mengirim data sensor ke Socket Server C# Desktop GUI secara real-time

## Praktikum
```python
import socket
from threading import Thread


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
                # MESSAGE = input("Input response:")
                MESSAGE = "OK"
                conn.send(MESSAGE.encode("utf8"))  # echo
            except Exception as e:
                print(e)
                break


TCP_IP = "192.168.43.85"
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

```cpp
#include <Arduino.h>
#include <ESP8266WiFi.h>

#define LED D4

const char *ssid = "***";
const char *password = "***";
const uint16_t port = 2004;
const char *host = "192.168.43.85";

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
}

void connect_server()
{
  WiFiClient client;

  Serial.printf("\n[Connecting to %s ... ", host);
  if (client.connect(host, port))
  {
    Serial.println("connected]");

    Serial.println("[Sending a request]");
    client.print("Hai from ESP8266");

    Serial.println("[Response:]");
    String line = client.readStringUntil('\n');
    Serial.println(line);
    client.stop();
    Serial.println("\n[Disconnected]");
  }
  else
  {
    Serial.println("connection failed!]");
    client.stop();
  }
  delay(3000);
}

void setup()
{
  Serial.begin(115200);
  connect_wifi();
}

void loop()
{
  connect_server();
}
```

## Tugas
Buatlah program yang dapat mengirimkan data sensor DHT dan dapat membaca perintah untuk menyalakan LED dari socket server. Skenario dari program tersebut adalah sebagai berikut, terlebih dahulu silakan buat rangkaiannya.
1. Secara berkala program akan mengirimkan data sensor DHT, Ultrasonic, dan cahaya.
2. Ketika menerima perintah untuk menyalakan LED. LED built-in akan berkedip sekali.
3. Tambahkan skenario untuk menerima perintah menyalakan lampu berdasarkan perangkat tertentu. Simulasikan lebih dari satu perangkat dalam sebuah socket server. 