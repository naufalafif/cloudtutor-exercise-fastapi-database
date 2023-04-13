#### Menggunakan PostgreSQL Database

Pada halaman sebelumnya kita sudah mencoba menggunakan API yang terhubung ke-database sqlite. Sekarang kita akan mencoba mengubah database yang digunakan menjadi PostgreSQL

Pertama kita perlu keluar dari proses `uvicorn`, gunakan tombol `CTRL + C` untuk menghentikan proses.

Selanjutnya kita bisa menjalankan database postgres. Kita akan menjalankan PostgreSQL menggunakan Docker dengan perintah dibawah

```{.bash .copy}
docker run \
    --name myPostgresDb \
    -p 5432:5432 \
    -e POSTGRES_USER=postgresUser \
    -e POSTGRES_PASSWORD=postgresPW \
    -e POSTGRES_DB=postgresDB \
    -d \
    postgres
```

Tunggu hingga selesai, proses ini bisa memakan waktu hingga 1 menit atau lebih.

Setelah berhasil menjalankan PostgreSQL, kita akan bisa menjalankan kembali uvicorn dan meng-update kode FastAPI

```{.bash .copy}
uvicorn main:app --host 0.0.0.0 --port 80 --reload
```

Untuk mengubah database dari SQLite menjadi PostgreSQL, kita hanya perlu mengubah db_url pada file `main.py`. Ubah db_url dari

```
sqlite://db.sqlite
```

menjadi url berikut

```{.txt .copy}
postgres://postgresUser:postgresPW@localhost:5432/postgresDB
```

hasil akhir dari function integrasi tortoise akan menjadi seperti dibawah

```{.python .copy}
register_tortoise(
    app,
    db_url="postgres://postgresUser:postgresPW@localhost:5432/postgresDB",
    modules={"models": ["models"]},
    generate_schemas=True,
    add_exception_handlers=True,
)
```

Selanjutnya mari kita coba kembali API pada dokumentasi FastAPI di url berikut

```{.bash .copy}
https://#ID#.host.cloudtutor.io/docs
```
