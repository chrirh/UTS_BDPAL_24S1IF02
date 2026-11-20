Pemantauan Aktivitas Penyuntingan Wikipedia Secara Real-Time

Repositori ini berisi implementasi pipeline Big Data end-to-end untuk memantau aktivitas penyuntingan Wikipedia secara real-time. Proyek ini dikerjakan sebagai bagian dari Ujian Tengah Semester (UTS) mata kuliah Big Data & Predictive Analytics Lanjut (ST153) di Universitas AMIKOM Yogyakarta.

📝 Deskripsi Proyek

1. Komunitas Wikimedia Indonesia membutuhkan sistem untuk:
2. Mengetahui halaman yang sedang ramai disunting (indikasi peristiwa terkini).
3. Memisahkan kontribusi bot dan manusia.
4. Mendeteksi lonjakan penyuntingan yang tidak wajar (indikasi edit war atau vandalisme).

Proyek ini dibangun menggunakan arsitektur terdistribusi dengan Apache Kafka sebagai message broker (Ingestion) dan Apache Spark Klaster (Standalone) untuk pemrosesan data, baik secara Batch maupun Streaming.

🏗️ Arsitektur Sistem

- Sumber Data: API Publik Wikimedia EventStreams (/v2/stream/recentchange) menggunakan format SSE (Server-Sent Events) / JSON.
- Node Master (Data Ingestion): Skrip Producer Python berjalan di PC Master untuk menarik data, memfilter field, menyamarkan IP pengguna (SHA-256), dan mengirimnya ke Kafka Broker.
- Node Worker / Klaster (Data Processing): Terdiri dari 1 Spark Master dan 2 Spark Worker. Pemrosesan dilakukan menggunakan:
- Spark RDD: Parsing JSON, Narrow/Wide Transformations, dan perbandingan kinerja reduceByKey vs groupByKey.
- Spark SQL / DataFrame: Analisis statistik bot vs manusia dan deteksi lonjakan anomali dengan ambang batas Persentil ke-99 (P99 Tumbling Window).
- Structured Streaming: Pemrosesan aliran data real-time dengan window 1 menit, toleransi keterlambatan (watermark) 2 menit, dan checkpointing untuk pemulihan kegagalan (fault-tolerance).

📂 Struktur Repositori

1. 6041_MASTER_Spark_Kafka.ipynb
Notebook dijalankan di PC Master. Berisi Producer Kafka, konfigurasi Topic, uji Decoupling, dan ekspor snapshot data ke HDFS/Shared Folder.

2. 6008_CLUSTER_Spark_Kafka.ipynb 
3. 6035_CLUSTER_Spark_Kafka.ipynb
Notebook dijalankan di Spark Klaster. Berisi seluruh analitik data menggunakan RDD, DataFrame, Spark SQL, dan operasi Structured Streaming.

🚀 Teknologi yang Digunakan

- Bahasa Pemrograman: Python 3.1x
- Message Broker: Apache Kafka & confluent-kafka
- Distributed Processing: Apache Spark 4.1.3 (PySpark)
- Streaming Package: spark-sql-kafka-0-10_2.13:4.1.3
Lainnya: Jupyter Notebook, requests (untuk SSE API)

📊 Highlight Kinerja (Benchmark)

1. Throughput Ingestion: Berhasil memproses >200.000 pesan mentah, dengan 119.372 pesan bersih terekspor ke dalam 3 partisi Kafka.
2. Optimasi Shuffle: Penyesuaian spark.sql.shuffle.partitions dari default 200 menjadi 8 terbukti menekan overhead dan mempercepat eksekusi agregasi SQL sebesar ~5x lipat (dari 4,53 detik menjadi 0,78 detik).
3. Streaming Stability: Trigger interval 30 detik memberikan keseimbangan terbaik antara throughput yang tinggi dengan latensi sistem yang tetap terjaga.
