---
layout: post
read_time: true
show_date: true
title:  Time Elapsed inside a Grid Cell
date:   2023-07-09 13:04:00 
description: Pengolahan Data Tracking GPS. Menghitung Waktu yang dihabiskan didalam suatu grid cell.  Counting time elapsed inside a grid cell of gps tracked animal movement data
img: posts/GPS Tracking.jpg 
tags: [Thoughts, Research]
author: Shulhan
github: 
---


Time Elapsed inside a Grid Cell  
Menghitung Waktu yang dihabiskan didalam suatu grid cell.  

Biasanya dilakukan dalam mengolah data tracking GPS.
Hal ini juga sangat penting apabila inging melakukan analisis dalam pergerakan hewan yang ditrack menggunakan GPS.
Ada beberapa langkah yang bisa dilakukan:

## Filtering
Filtering berguna untuk menghilangkan data-data yang error atau rusak.
Contohnya seperti apabila hewan yang ditag adalah hewan aquatik, akan tetapi data yang diperoleh menunjukkan koordinat pada daratan. Ini merupakan suatu hal yang tidak mungkin, sehingga perlu dilakukan filtering atau mgenhilangkan data error tersebut.
Contoh lainnya apabila ada pecatatan waktu yang sama dalam koordinat yang berbeda. Ini juga merupakan suatu hal yang tidak mungkin, kecuali hewan tersebut membelah diri, dan gpsnya juga membelah diri jadi dua.
Apabila ada data dengan waktu yang sama dengan koordinat yang sama, ini juga harus dihapus salah satunya.

### Spatial filtering   
atau filtering terhadap ruang (contoh 1) dapat dilakukan dengan menggunakan software GIS seperti Qgis atau Arcgis, dan melihat apakah titik koordinat pencatatan sudah sesuai atau belum (hewan laut berada di laut, hewan darat berada di darat).
Jika belum, maka data-data yang tidak sesuai tersebut bisa dihilangkan.

### Temporal filtering  
atau filtering terhadap waktu (contoh 2) dapat dilakukan dengan menggunakan software pengolah data angka seperti MS Excell, dan melihat apakah ada waktu yang sama pada koordinat yang berbeda atau pada koordinat yang sama.
Apabila pada koordinat yang sama, maka gunakan salah satunya.
Contoh:
| No   | Time      | Latitude  | Longitude |
|------|-----------|-----------|-----------|
| 1    | 09:05 AM  | 34.0522° N | 118.2437° W |
| 2    | 09:10 AM  | 40.7128° N | 74.0060° W  |
| 3    | 09:20 AM  | 35.6895° N | 139.6917° W |
| 4    | 09:05 AM  | 34.0522° N | 118.2437° W |
| 5    | 09:00 AM  | 37.7749° N | 122.4194° W |
| 6    | 09:15 AM  | 51.5074° N | 0.1278° W   |


Bisa dilihat pada tabel tersebut bahwa data nomor 1 memiliki waktu yang sama dengan data nomer 4. Maka salah satunya harus dihapus

Filtering ini berguna agar dapat meminimalisir kesalahan pada pengolahan dalam tahap selanjutnya.


## Sorting
Setelah data tracking difilter, tahap selanjutnya adalah dengan melakukan sorting atau pengurutan. 
Data diurutkan dari waktu paling awal hingga paling akhir. 

| No   | Time      | Latitude  | Longitude |
|------|-----------|-----------|-----------|
| 1    | 09:00 AM  | 37.7749° N | 122.4194° W |
| 2    | 09:05 AM  | 35.0478° N | 110.2437° W |
| 3    | 09:05 AM  | 34.0522° N | 118.2437° W |
| 4    | 09:10 AM  | 40.7128° N | 74.0060° W  |
| 5    | 09:15 AM  | 51.5074° N | 0.1278° W   |
| 6    | 09:20 AM  | 35.6895° N | 139.6917° W |

## Kalkulasi 1
Data yang telah disorting, kini dapat estimasi waktu yang diperlukan oleh hewan untuk bergerak dari satu titik ke titik selanjutnya.
Penghitungan dilakukan dengan cara mengurangi waktu yang lebih akhir dengan waktu yang lebih awal.
Contoh: 
| No   | Time      | Latitude  | Longitude |
|------|-----------|-----------|-----------|
| 1    | 09:00 AM  | 37.7749° N | 122.4194° W |
| 2    | 09:05 AM  | 34.0522° N | 118.2437° W |
| 3    | 09:10 AM  | 40.7128° N | 74.0060° W  |
| 4    | 09:15 AM  | 51.5074° N | 0.1278° W   |
| 5    | 09:20 AM  | 35.6895° N | 139.6917° W |

setelah dihitung lama waktunya:
| ID   | Time      | Latitude  | Longitude | Elapsed Time |
|------|-----------|-----------|-----------|--------------|
| 1    | 09:00 AM  | 37.7749° N | 122.4194° W |              |
| 2    | 09:05 AM  | 34.0522° N | 118.2437° W |   00:05      |
| 3    | 09:10 AM  | 40.7128° N | 74.0060° W  |   00:05      |
| 4    | 09:15 AM  | 51.5074° N | 0.1278° W   |   00:05      |
| 5    | 09:22 AM  | 35.6895° N | 139.6917° E |   00:07      |

penghitungan lebih lanjut dapat dilakukan untuk menghitung jarak yang ditempuh maupun kecepatan saat hewan berpindah dari satu titik ke titik yang lain

## Point to Line
Tahap selanjutnya adalah membuat garis yang menghubungkan antara dua 2 titik dalam waktu yang berurutan.
Ini dilakukan untuk menghitung jarak antar titik, dan digunakan untuk tahapan pengolahan selanjtunya.
Pembuatan garis ini dilakukan menggunakan software GIS.
![Point to Line](./assets/img/posts/20230709/pointtoline.jpg)


## Kalkulasi 2
Kalkulasi ini menghitung jumlah waktu yang dihabiskan dalam suatu grid (suatu area).
1. Buat shapefile grid denan luas tertentu, misal 10x10 km atau 1x1 derajat
2. Potong shapefile garis yang tadi telah dibuat dengan shapefile grid (intersect)
![Kalkulasi Rasio Line](./assets/img/posts/20230709/Linegridratio.jpg)
3. Hitung rasio dari garis yang berada dalam suatu grid dengan panjang garis. Dalam contoh ini, garis L3 berada dalam 2 grid, yaitu grid A dan Grid B.
L3 yang berada dalam grid A adalah x, dan L3 yang berada dalam grid B adalah y. Maka untuk mengitung rasio x adalah x/l3 dan y adalah y/l3. 
4. Hitung rasio waktu dengan mengalikan waktu yang ditempuh dengan rasio pada langkah 3
5. Lakukan summary statistics untuk menghitung jumlah waktu yang ada dalam grid
