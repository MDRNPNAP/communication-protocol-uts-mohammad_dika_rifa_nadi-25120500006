//menambah produk dengan body tidak lengkap

curl -i -X POST "http://127.0.0.1:8088/api/products"
-H "Content-Type: application/json"
-d '{"name":"Produk Tanpa Stok","category":"accessory","price":99000}'

//melakukan penyesuaian pada body 
-d '{ "name": "Produk Tanpa Stok", "category": "accessory", "price": 99000, "stock": 20 }
