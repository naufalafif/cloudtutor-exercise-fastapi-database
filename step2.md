Sekarang mari kita jalankan dan test aplikasi

Pertama install dependensi dan jalankan uvicorn

```{.bash .copy}
pip install -r requirements.txt
```

```{.bash .copy}
uvicorn main:app --host 0.0.0.0 --port 80 --reload
```

Selanjutnya buka dokumentasi aplikasi dan coba API pada url berikut

```{.bash .copy}
https://#ID#.host.cloudtutor.io/docs
```

> Gunakan browser disebelah kanan atau tab baru pada browser anda

#### Menambah API Delete

Untuk menambah API delete, kita akan menggunakan function delete pada class Items.

Pertama import `HTTPException` pada bagian atas file `main.py`

```{.python .copy}
from fastapi import HTTPException
```

Lalu tambahkan kode berikut pada deretan kode route

```{.python .copy}
@app.delete("/item/{item_id}")
async def delete_item(item_id: int):
    deleted_count = await Items.filter(id=item_id).delete()
    if not deleted_count:
        raise HTTPException(status_code=404, detail=f"item {item_id} not found")
    return {
      "message": f"item {item_id} berhasil dihapus",
      "success": True
    }
```

Berhasil, selanjutnya kita bisa mencoba API ini url pada dokumentasi sebelumnya.

> Klik tombol dengan icon refresh (pojok kanan atas) pada browser disebelah kanan untuk merefresh halaman

#### Menambah API Put

Penulisan kode put atau update mirip dengan penulisan post pada API menambah data, perbedaannya hanya pada penggunaan function `update` dan penambahan parameter item_id untuk menentukan item yang ingin di ubah

Tambahkan kode berikut pada deretan kode route

```{.python .copy}
@app.put("/item/{item_id}")
async def update_item(item_id: int, item: ItemIn_Pydantic):
    await Items.filter(id=item_id).update(**item.dict(exclude_unset=True))
    return await Item_Pydantic.from_queryset_single(Items.get(id=item_id))
```

Berhasil, kita bisa mencoba API ini pada url dokumentasi sebelumnya.
