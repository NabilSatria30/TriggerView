# 📦 Database Toko Minuman

## 🗄️ 1. Membuat Database
CREATE DATABASE toko_minuman;
USE toko_minuman;

---

## 📋 2. Struktur Tabel

### Tabel produk
CREATE TABLE produk (
    id_produk INT AUTO_INCREMENT PRIMARY KEY,
    nama_produk VARCHAR(100),
    harga INT,
    stok INT
);

### Tabel pelanggan
CREATE TABLE pelanggan (
    id_pelanggan INT AUTO_INCREMENT PRIMARY KEY,
    nama_pelanggan VARCHAR(100)
);

### Tabel transaksi
CREATE TABLE transaksi (
    id_transaksi INT AUTO_INCREMENT PRIMARY KEY,
    id_produk INT,
    id_pelanggan INT,
    jumlah INT,
    total_harga INT,
    FOREIGN KEY (id_produk) REFERENCES produk(id_produk),
    FOREIGN KEY (id_pelanggan) REFERENCES pelanggan(id_pelanggan)
);

### Tabel pembayaran
CREATE TABLE pembayaran (
    id_bayar INT AUTO_INCREMENT PRIMARY KEY,
    id_transaksi INT,
    jumlah_bayar INT,
    FOREIGN KEY (id_transaksi) REFERENCES transaksi(id_transaksi)
);

### Tabel log_aktivitas
CREATE TABLE log_aktivitas (
    id_log INT AUTO_INCREMENT PRIMARY KEY,
    keterangan VARCHAR(255)
);

---

## 🧾 3. Insert Data Awal

### Produk
INSERT INTO produk (nama_produk, harga, stok) VALUES
('Es Kopi Susu', 18000, 20),
('Matcha Latte', 22000, 15),
('Thai Tea', 15000, 25),
('Cokelat Creamy', 20000, 10);

### Pelanggan
INSERT INTO pelanggan (nama_pelanggan) VALUES
('Kemala'),
('Bama'),
('Nabil');

---

## ⚙️ 4. Trigger

### Hitung Total Harga
DELIMITER $$
CREATE TRIGGER trg_total_harga
BEFORE INSERT ON transaksi
FOR EACH ROW
SET NEW.total_harga = (
    SELECT harga * NEW.jumlah
    FROM produk
    WHERE id_produk = NEW.id_produk
);
$$
DELIMITER ;

### Kurangi Stok
DELIMITER $$
CREATE TRIGGER trg_kurangi_stok
AFTER INSERT ON transaksi
FOR EACH ROW
BEGIN
    UPDATE produk
    SET stok = stok - NEW.jumlah
    WHERE id_produk = NEW.id_produk;
END$$
DELIMITER ;

### Log Transaksi
DELIMITER $$
CREATE TRIGGER trg_log_transaksi
AFTER INSERT ON transaksi
FOR EACH ROW
BEGIN
    INSERT INTO log_aktivitas(keterangan)
    VALUES (
        CONCAT('Transaksi baru: Produk ID ', NEW.id_produk,
        ', Pelanggan ID ', NEW.id_pelanggan,
        ', Jumlah ', NEW.jumlah,
        ', Total ', NEW.total_harga)
    );
END$$
DELIMITER ;

### Log Stok Habis
DELIMITER $$
CREATE TRIGGER trg_stok_habis
AFTER UPDATE ON produk
FOR EACH ROW
BEGIN
    IF NEW.stok = 0 THEN
        INSERT INTO log_aktivitas(keterangan)
        VALUES (CONCAT('Stok habis untuk Produk ID ', NEW.id_produk));
    END IF;
END$$
DELIMITER ;

### Kembalikan Stok
DELIMITER $$
CREATE TRIGGER trg_kembalikan_stok
AFTER DELETE ON transaksi
FOR EACH ROW
BEGIN
    UPDATE produk
    SET stok = stok + OLD.jumlah
    WHERE id_produk = OLD.id_produk;
END$$
DELIMITER ;

### Log Pembayaran
DELIMITER $$
CREATE TRIGGER trg_log_pembayaran
AFTER INSERT ON pembayaran
FOR EACH ROW
BEGIN
    INSERT INTO log_aktivitas(keterangan)
    VALUES (
        CONCAT('Pembayaran berhasil: ID Transaksi ',
        NEW.id_transaksi, ', Jumlah ', NEW.jumlah_bayar)
    );
END$$
DELIMITER ;

---

## 💳 5. Transaksi & Pembayaran

INSERT INTO transaksi (id_produk, id_pelanggan, jumlah) VALUES
(1,1,2),
(2,2,1),
(3,3,5);

INSERT INTO pembayaran (id_transaksi, jumlah_bayar) VALUES
(1, 36000),
(2, 22000),
(3, 75000);

---

## 👁️ 6. View

### View Transaksi Detail
CREATE VIEW view_transaksi_detail AS
SELECT 
    t.id_transaksi,
    p.nama_produk,
    c.nama_pelanggan,
    t.jumlah,
    t.total_harga
FROM transaksi t
JOIN produk p ON t.id_produk = p.id_produk
JOIN pelanggan c ON t.id_pelanggan = c.id_pelanggan;

### View Stok Produk
CREATE VIEW view_stok_produk AS
SELECT id_produk, nama_produk, stok
FROM produk;

---

## 📊 7. Query Data

SELECT * FROM transaksi;
SELECT * FROM produk;
SELECT * FROM log_aktivitas;
SELECT * FROM view_transaksi_detail;
SELECT * FROM view_stok_produk;

---

## ✅ 8. Hasil

- Total harga otomatis terhitung
- Stok berkurang otomatis
- Log aktivitas tercatat
- View mempermudah query data
