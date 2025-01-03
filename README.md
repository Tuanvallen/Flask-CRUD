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
    <title> Halaman {% block judul%} {% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/bootstrap.min.css') }}">
</head>
<body>
    <div class="content-wrapper">
        <section class="content-header">
            <h1>
                <a href="/">
                    <img src="{{ url_for('static', filename='img/amikom_logo.png') }}" style="width: 40px;">
                </a>
                Data Mahasiswa    
                <small>Teknik Komputer</small>
            </h1>
        </section>  
        <section class="content">
            <div class="row">
                <div class="col-xs-12">
                    <div class="box">
                        <div class="box-header">
                            {% block JudulKonten %} {% endblock %}
                        </div>
                        <div class="box-body">
                            {% block konten %} {% endblock %}
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </div>
    
    <script src="{{ url_for('static', filename='js/bootstrap.min.js') }}"></script>
</body>
</html>
```

### 2. index.html
```html
{% extends 'base.html' %}
{% block judul %} Data Mahasiswa {% endblock %}

{% block JudulKonten %}
    <a href="/tambah/" class="btn btn-info"> Tambah Data </a>
{% endblock %}

{% block konten %}
    <table class="table table-bordered table-hover">
        <thead>
            <tr>
                <th>NIM</th>
                <th>Nama</th>
                <th>Asal</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            {% for row in hasil %}
                <tr>
                    <td>{{ row.0 }}</td>
                    <td>{{ row.1 }}</td>
                    <td>{{ row.2 }}</td>
                    <td>
                        <a href="/ubah/{{ row.0 }}" class="btn btn-warning"> Ubah </a>
                        <a href="/hapus/{{ row.0 }}" class="btn btn-danger" onclick="return confirm('Yakin akan menghapus {{ row.1 }}?')"> Hapus </a>
                    </td>
                </tr>
            {% endfor %}                
        </tbody>
    </table>
{% endblock %}
```

## Implementasi CRUD di app.py

### 1. Menampilkan Data (Read)
```python
@app.route('/')
def halaman_awal():
    cur = db. cursor ()
    cur.execute("select * from tbl_mahasiswa ")
    res = cur. fetchall()
    cur.close()
    return render_template('index.html', hasil=res)

```

### 2. Tambah Data (Create)
```python
@app.route('/tambah/')
def tambah_data():
    return render_template('tambah.html')

@app.route('/proses_tambah/', methods=['post'])
def proses_tambah():
    nim = request. form['nim']
    nama = request. form['nama']
    asal = request. form['asal']
    cur = db.cursor ()
    cur.execute('INSERT INTO tbl_mahasiswa (nim, nama, asal) VALUES (%s, %s, %s) ' , (nim, nama, asal) )
    db.commit()
    return redirect(url_for ('halaman_awal'))
```

### 3. Ubah Data (Update)
```python
@app.route('/ubah/<nim>', methods=['GET'])
def ubah_data(nim):
    cur = db.cursor()
    cur.execute('SELECT * FROM tbl_mahasiswa WHERE nim=%s', (nim,))
    res = cur.fetchall()
    cur.close()
    return render_template('ubah.html', hasil=res)

@app.route('/proses_ubah', methods=['POST'])
def proses_ubah():
    nim_ori = request.form['nim_ori']
    nim = request.form['nim']
    nama = request.form['nama']  
    asal = request.form['asal']
    
    try:
        cur = db.cursor()
        sql = "UPDATE tbl_mahasiswa SET nim=%s, nama=%s, asal=%s WHERE nim=%s"
        value = (nim, nama, asal, nim_ori)
        cur.execute(sql, value)
        db.commit()
        cur.close()
        return redirect(url_for('halaman_awal'))
    except Exception as e:
        db.rollback()
        return f"Error: {str(e)}"
```

### 4. Hapus Data (Delete)
```python
@app.route('/hapus/<nim>', methods=['GET'])
def hapus_data(nim):
    try:
        cur = db.cursor()
        cur.execute('DELETE FROM tbl_mahasiswa WHERE nim=%s', (nim,))
        db.commit()
        cur.close()
        return redirect(url_for('halaman_awal'))
    except Exception as e:
        db.rollback()
        return f"Error: {str(e)}"
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

<!-- Fonts -->(saya tidak menggunakanya karena magar)
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
