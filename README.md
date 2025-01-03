# Flask-CRUD
## langkah di bawah benar hanya saja saya memodifikasi beberapa codenya jadi untuk source code yang benar download file di atas

# Flask-CRUD
Implementasi Create, Read, Update, Delete menggunakan Flask dan MySQL

## Prasyarat
- SQL Engine [Services harus aktif]
- Database yang digunakan [db_kuliah.sql]
- SQL Connection
- Flask Installation

## Mengaktifkan SQL
1. Buka XAMPP Control Panel
2. Aktifkan Service Apache dan MySQL

## Mengakses MySQL
1. Buka browser
2. Pada address bar ketikkan alamat url: `http://localhost/phpmyadmin`
3. Default account:
   - Username: root
   - Password: (kosong/tidak ada password)

## Import SQL
1. Siapkan database yang akan digunakan
2. Buat database dengan nama `db_kuliah`
3. Import database `db_kuliah.sql`
4. Pastikan database db_kuliah memiliki struktur tabel yang sesuai

## Instalasi Flask
1. Pastikan Flask telah terinstall dengan baik
2. Uji instalasi dengan menjalankan service Flask untuk menampilkan "Hello World" pada Browser

## Koneksi SQL - Flask
1. Pastikan Package mysql-connector telah terinstall
   - **PENTING**: Instalasi mysql-connector harus dilakukan pada virtualenvironment

2. Open Connection SQL - Flask:
```python
# Import library mysql-connector
import mysql.connector

# Melakukan open connection
mydb = mysql.connector.connect(
    host="localhost",
    user="root",
    password="",
    database="db_kuliah"
)

# Untuk mengetahui keberhasilan proses open connection
print(mydb)
```

## Struktur Flask CRUD
### Tujuan: Dashboard [Administration Page]
- Page Title
- Main Content
- Content Title

### Struktur File HTML (dalam folder `templates`)
1. `base.html`
2. `index.html`
3. `tambah.html`
4. `ubah.html`

**Catatan**: Semua file HTML harus diletakkan dalam directory `templates`

### Import yang Diperlukan
```python
from flask import render_template
```

## Implementasi File HTML

### 1. base.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <h1>Administration Page</h1>
    <div class="content">
        <h2>{% block content_title %}{% endblock %}</h2>
        {% block main_content %}{% endblock %}
    </div>
</body>
</html>
```

### 2. index.html
```html
{% extends 'base.html' %}

{% block title %}Data Mahasiswa{% endblock %}

{% block content_title %}Data Mahasiswa{% endblock %}

{% block main_content %}
    <table border="1">
        <tr>
            <th>NIM</th>
            <th>Nama</th>
            <th>Aksi</th>
        </tr>
        {% for row in container %}
        <tr>
            <td>{{row[0]}}</td>
            <td>{{row[1]}}</td>
            <td>
                <a href="/ubah/{{row[0]}}">Ubah</a>
                <a href="/hapus/{{row[0]}}">Hapus</a>
            </td>
        </tr>
        {% endfor %}
    </table>
    <a href="/tambah">Tambah Data</a>
{% endblock %}
```

## Implementasi CRUD di app.py

### 1. Menampilkan Data (Read)
```python
@app.route('/')
def index():
    cursor = mydb.cursor()
    cursor.execute('SELECT * FROM mahasiswa')
    result = cursor.fetchall()
    cursor.close()
    return render_template('index.html', container=result)
```

### 2. Tambah Data (Create)
```python
from flask import request, redirect, url_for

@app.route('/tambah', methods=['GET', 'POST'])
def tambah():
    if request.method == 'POST':
        nim = request.form['nim']
        nama = request.form['nama']
        cursor = mydb.cursor()
        cursor.execute('INSERT INTO mahasiswa (nim, nama) VALUES (%s, %s)', (nim, nama))
        mydb.commit()
        cursor.close()
        return redirect(url_for('index'))
    return render_template('tambah.html')
```

### 3. Ubah Data (Update)
```python
@app.route('/ubah/<nim>', methods=['GET', 'POST'])
def ubah(nim):
    cursor = mydb.cursor()
    if request.method == 'POST':
        nim_baru = request.form['nim']
        nama = request.form['nama']
        cursor.execute('UPDATE mahasiswa SET nim=%s, nama=%s WHERE nim=%s', (nim_baru, nama, nim))
        mydb.commit()
        cursor.close()
        return redirect(url_for('index'))
    cursor.execute('SELECT * FROM mahasiswa WHERE nim=%s', (nim,))
    result = cursor.fetchone()
    cursor.close()
    return render_template('ubah.html', data=result)
```

### 4. Hapus Data (Delete)
```python
@app.route('/hapus/<nim>')
def hapus(nim):
    cursor = mydb.cursor()
    cursor.execute('DELETE FROM mahasiswa WHERE nim=%s', (nim,))
    mydb.commit()
    cursor.close()
    return redirect(url_for('index'))
```

## Mempercantik Tampilan Web

### Struktur Folder Static
```
static/
├── css/
├── js/
├── img/
└── fonts/
```

### Langkah-langkah:
1. Buat directory dengan nama `static`
2. Copy file css, js, img dan font yang ada pada directory assets ke folder static
3. Update referensi file di base.html:
```html
<!-- CSS -->
<link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">

<!-- JavaScript -->
<script src="{{ url_for('static', filename='js/script.js') }}"></script>

<!-- Images -->
<img src="{{ url_for('static', filename='img/logo.png') }}">

<!-- Fonts -->
<link rel="stylesheet" href="{{ url_for('static', filename='fonts/font.css') }}">
```

## Menjalankan Aplikasi
1. Pastikan service MySQL sudah berjalan
2. Jalankan aplikasi Flask:
```bash
python app.py
```
3. Buka browser dan akses `http://localhost:5000`

## Catatan Penting
- Pastikan semua service (Apache dan MySQL) sudah berjalan sebelum menjalankan aplikasi
- Periksa koneksi database sebelum menjalankan operasi CRUD
- Semua file HTML harus berada dalam folder `templates`
- Semua asset (CSS, JS, images, fonts) harus berada dalam folder `static`
- Gunakan virtual environment untuk menghindari konflik dependensi

## Troubleshooting
1. Jika terjadi error koneksi database:
   - Periksa service MySQL sudah berjalan
   - Periksa kredensial database (username dan password)
   - Pastikan database `db_kuliah` sudah dibuat

2. Jika template tidak ditemukan:
   - Pastikan struktur folder sudah benar
   - Periksa nama file template sesuai dengan yang dipanggil di `render_template()`

3. Jika static files tidak termuat:
   - Periksa struktur folder static
   - Pastikan path di `url_for()` sudah benar
