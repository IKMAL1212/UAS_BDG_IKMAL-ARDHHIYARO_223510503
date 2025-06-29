LOAD CSV WITH HEADERS FROM 'file:///dataset_perusahaan_produk.csv' AS row
MERGE (p:Perusahaan {id: row.id_perusahaan})
SET p.nama = row.nama_perusahaan,
    p.negara = row.negara_asal,
    p.tahun_berdiri = toInteger(row.tahun_berdiri);

MERGE (pr:Produk {nama: row.nama_produk})
SET pr.jenis = row.jenis_produk,
    pr.tahun_rilis = toInteger(row.tahun_rilis),
    pr.harga = toFloat(row.harga_rata_rata);

MERGE (i:Industri {kategori: row.kategori_industri});
MERGE (w:Wilayah {nama: row.wilayah_pemasaran});

MERGE (p)-[:MEMPRODUKSI]->(pr);
MERGE (pr)-[:TERGOLONG_DALAM]->(i);
MERGE (pr)-[:DIPASARKAN_DI]->(w);
