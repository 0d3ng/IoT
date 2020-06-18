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



## Tugas
Terdapat sebuah dusun di desa tertentu yang sudah menerapkan IoT, contoh penerapan tersebut di gang-gang ketika sudah beranjak malam lampu yang terdapat pada gang tersebut akan menyala. Pada dusun tersebut juga terdapat kebun rumah kaca, dimana suhu dan kelembaban sangat diperhatikan untuk menjaga produktivitas sayur-sayur di dalam kebun. Semua sensor yang terdapat pada dusun tersebut juga dapat dimonitoring dan semua lampu yang terdapat pada gang-gang dapat dinyalakan melalui server.

Dari kasus di atas, buat program untuk kebutuhan tersebut baik dari sisi controller (ESP8266) atau dari sisi server.