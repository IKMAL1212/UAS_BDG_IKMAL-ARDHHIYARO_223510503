//html

<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <title>Popoto.js Example</title>

  <!-- Popoto.js CSS -->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/popoto/dist/popoto.min.css" />

  <!-- FontAwesome untuk ikon -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" />

  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      height: 100vh;
      display: flex;
      flex-direction: column;
    }
    .ppt-section-header {
      background: #34495e;
      color: white;
      padding: 10px 20px;
      font-size: 1.2rem;
      flex: none;
    }
    .ppt-container-graph {
      flex: 1;
      display: flex;
      height: 100%;
    }
    #popoto-taxonomy {
      width: 200px;
      background: #ecf0f1;
      overflow-y: auto;
      border-right: 1px solid #bdc3c7;
    }
    #popoto-graph {
      flex: 1;
      background: #fff;
    }
    #popoto-cypher {
      padding: 10px;
      background: #f5f5f5;
      font-family: monospace;
      font-size: 0.9rem;
      border-top: 1px solid #ddd;
      flex: none;
    }
    #popoto-results {
      padding: 10px;
      background: #fafafa;
      border-top: 1px solid #ddd;
      flex: none;
      max-height: 150px;
      overflow-y: auto;
    }
  </style>
</head>
<body>

  <div class="ppt-section-header">Neo4j (Popoto.js) - Visualisasi Perusahaan dan Produk</div>

  <div class="ppt-container-graph">
    <nav id="popoto-taxonomy" class="ppt-taxo-nav"></nav>
    <div id="popoto-graph" class="ppt-div-graph"></div>
  </div>

  <div id="popoto-cypher"></div>
  <div id="popoto-results"></div>

  <!-- Dependencies -->
  <script src="https://cdn.jsdelivr.net/npm/d3@7"></script>
  <script src="https://cdn.jsdelivr.net/npm/neo4j-driver-lite/lib/browser/neo4j-lite-web.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/popoto/dist/popoto.min.js"></script>

  <script>
    // Buat koneksi ke Neo4j
    const driver = neo4j.driver(
      "bolt://localhost:7687",
      neo4j.auth.basic("neo4j", "12345678")  // Ganti username & password sesuai Neo4j kamu
    );

    popoto.runner.DRIVER = driver;
    popoto.query.COLLECT_RELATIONS_WITH_VALUES = true;

    // Konfigurasi node dengan atribut yang akan ditampilkan
    popoto.provider.node.Provider = {
      "Perusahaan": {
        returnAttributes: ["nama", "negara", "tahun_berdiri"],
        constraintAttribute: "id",
        autoExpandRelations: true,
        isLabelGroup: true
      },
      "Produk": {
        returnAttributes: ["nama", "jenis", "tahun_rilis", "harga"],
        constraintAttribute: "nama",
        autoExpandRelations: true,
        isLabelGroup: true
      },
      "Industri": {
        returnAttributes: ["kategori"],
        constraintAttribute: "kategori",
        autoExpandRelations: true,
        isLabelGroup: true
      },
      "Wilayah": {
        returnAttributes: ["nama"],
        constraintAttribute: "nama",
        autoExpandRelations: true,
        isLabelGroup: true
      }
    };

    // Konfigurasi relasi supaya label tampil rapi
    popoto.provider.link.Provider = {
      getTextValue: function (link) {
        return link.label
          .replace(/_/g, " ")
          .replace(/\b\w/g, l => l.toUpperCase());
      }
    };

    // Konfigurasi taxonomy sidebar dengan label-label Neo4j kamu
    popoto.provider.taxonomy.Provider = {
      getTaxonomy: function () {
        return [
          { label: "Perusahaan" },
          { label: "Produk" },
          { label: "Industri" },
          { label: "Wilayah" }
        ];
      },
      getCSSClass: function (label, element) {
        return "ppt-taxo__" + element + " " + label;
      }
    };

    // Update jumlah hasil query di bagian bawah
    popoto.result.onTotalResultCount(function (count) {
      // Buat elemen span untuk hasil count di header (bisa ditambahkan)
      let countSpan = document.getElementById("result-total-count");
      if (!countSpan) {
        countSpan = document.createElement("span");
        countSpan.id = "result-total-count";
        countSpan.style.marginLeft = "8px";
        document.querySelector(".ppt-section-header").appendChild(countSpan);
      }
      countSpan.textContent = "(" + count + ")";
    });

    // Mulai Popoto setelah cek koneksi Neo4j
    driver.verifyConnectivity()
      .then(() => popoto.start("Perusahaan"))
      .catch(error => {
        alert("Koneksi ke Neo4j gagal: " + error);
        console.error(error);
      });
  </script>
</body>
</html>





//NEO4J

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
