#### Database in FastAPI

FastAPI, sebagai sebuah framework web, tidak dilengkapi dengan sistem manajemen database (DBMS) sendiri. Sebaliknya, FastAPI menyediakan cara yang mudah untuk berinteraksi dengan database melalui dukungannya untuk berbagai ORM (Object-Relational Mapping) Python dan driver database. Kita dapat menggunakan ORM seperti Tortoise ORM, SQLAlchemy, atau Pony ORM, ataupun menggunakan driver database langsung seperti asyncpg untuk PostgreSQL atau databases untuk SQLite.

Pada exercise ini kita akan menggunakan Tortoise ORM

#### Tortoise ORM

Pertama kita mulai dengan meng-clone project dengan perintah berikut

```{.bash .copy}
git clone https://github.com/cloudtutor-io/learn-materials.git /home/admin/learning-materials \
&& cd /home/admin/learning-materials/simple-fastapi-database
```

Gunakan editor disebelah kanan untuk membuka direktori `admin/learning-materials/simple-fastapi-database`, klik icon refresh bila folder belum tampil

Sekarang kita sudah memiliki aplikasi yang menggunakan FastAPI dan Tortoise ORM

#### Model / Tabel

Pada `models.py`, kita mendefinisikan class dari tabel Items dengan column id, nama, harga, created_at dan modified_at yang bisa kita
lihat pada kode

```
class Items(models.Model):
    id = fields.IntField(pk=True)
    nama = fields.CharField(max_length=50, null=True)
    harga = fields.FloatField(null=False)
    created_at = fields.DatetimeField(auto_now_add=True)
    modified_at = fields.DatetimeField(auto_now=True)
```

dan terdapat juga kode untuk membuat pydantic class dari model Item pada kode berikut

```
Item_Pydantic = pydantic_model_creator(Items, name="Item")
ItemIn_Pydantic = pydantic_model_creator(Items, name="ItemIn", exclude_readonly=True)
```

Kode ini digunakan untuk mendefinisikan skema pydantic untuk FastAPI. Skema ini digunakan oleh FastAPI untuk membuat validasi otomatis pada API ketika menerima request

Dari kode diatas bisa kita lihat bahwa terdapat 2 hal yang perlu dilakukan untuk membuat sebuah model / tabel baru yaitu, mendefinisikan model & membuat skema pydantic dari model tersebut

#### Integrasi

Kode integrasi FastAPI dengan Tortoise ORM terdapat pada file `main.py` tepatnya pada kode berikut

```{.python}
register_tortoise(
    app,
    db_url="sqlite://:memory:",
    modules={"models": ["models"]},
    generate_schemas=True,
    add_exception_handlers=True,
)
```

Function `register_tortoise` menerima instance dari FastAPI `app`, url database pada `db_url`, import model pada `modules` dan beberapa parameter optional lainnya.

`db_url` adalah connection string yang umumnya digunakan untuk mendifinisikan connection ke database. Url ini bisa berupa URL Postgres, MySQL ataupun database lainnya

`modules` adalah objek yang mendefinisikan modules atau file models mana saja yang digunakan oleh aplikasi. Pada kode diatas modules memiliki value `[models]` dikarenakan model yang kita buat berada di file models.py

misal kita ingin menggunakan postgres dengan models berada pada file database.py, maka konfigurasi tortoise akan berubah menjadi

```{.python}
register_tortoise(
    app,
    db_url="postgres://user:pass@host:5432/somedb",
    modules={"models": ["database"]},
    generate_schemas=True,
    add_exception_handlers=True,
)
```

#### Membaca dan Menambah Data

Penulisan code untuk membaca data dilakukan dengan cara memanggil function pada Class Model dan meng-convert data ke bentuk skema pydantic seperti pada contoh di file `main.py`

```{.python}
@app.get("/items")
async def get_items():
    return await Item_Pydantic.from_queryset(Items.all())
```

Data list items didapatkan dari kode `Items.all` lalu di-convert menjadi data pydantic menggunakan function `Item_Pydantic.from_queryset`

Sedangkan untuk menambah data kita perlu menggunakan kedua class pydantic yaitu `Item_Pydantic` dan `ItemIn_Pydantic`

Item_Pydantic digunakan untuk me-return response API sedangkan ItemIn_Pydantic digunakan untuk menerima request body/data seperti pada kode dibawah

```{.python}
@app.post("/items")
async def create_item(item: ItemIn_Pydantic):
    item_obj = await Items.create(**item.dict(exclude_unset=True))
    return await Item_Pydantic.from_tortoise_orm(item_obj)
```

Perhatikan penempatan kedua class pydantic, `ItemIn_Pydantic` ditempatkan sebagai parameter function dan `item_Pydantic` ditempatkan pada return function.

Sedangkan untuk membuat data baru digunakan kode `Items.create` dengan data dari kode `**item.dict(exclude_unset=True)`

penggunaan \*\* pada kode diatas adalah untuk memetakan data pydantic sebagai parameter untuk kode `Items.create`. Contoh penggunaannya adalah seperti berikut

```{.python}
params = {"name": "naufal", "sex": "male"}

function_add(**params)
# adalah sama dengan penulisan dibawah
function_add(name=params["name], sex=params["sex])
```
