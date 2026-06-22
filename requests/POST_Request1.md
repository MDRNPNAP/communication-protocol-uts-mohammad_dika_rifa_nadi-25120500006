//menambahkan produk
curl -i -X POST "http://127.0.0.1:8088/api/products" \
 -H "Content-Type: application/json" \
 -d '{"name":"Mechanical Keyboard K1","category":"accessory","price":650000,"stock":15}'
