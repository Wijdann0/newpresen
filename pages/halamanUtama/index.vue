<template>
  <div class="container-fluid hei">
    <div class="row my-5 d-flex justify-content-around mt-3 atas">
      <div class="col-12 col-sm-6 col-lg-4 mb-5">
        <nuxt-link to="/presensi" class="text-decoration-none">
          <div class="card bg-isiPresensi rounded-5 text-light mx-2"> <!-- Tambahkan mx-2 -->
            <div class="card-body d-flex justify-content-center align-items-center">
              <h2 class="">Isi Presensi</h2>
            </div>
          </div>
        </nuxt-link>
      </div>
      <div class="col-12 col-sm-6 col-lg-4 mb-5">
        <nuxt-link to="/laporan" class="text-decoration-none">
          <div class="card bg-laporanMingguan rounded-5 text-light mx-2"> <!-- Tambahkan mx-2 -->
            <div class="card-body d-flex justify-content-center align-items-center">
              <h2 class="">Laporan</h2>
            </div>
          </div>
        </nuxt-link>
      </div>
    </div>
    <div class="row pb-3">
      <div class="col text-center p-3">
        <nuxt-link to="/log/logout">
          <button class="btn btn-dark bck mt-5 mb-5 border-white">LogOut</button>
        </nuxt-link>
      </div>
    </div>
  </div>
</template>


<style scoped>
* {
  padding: 0;
  margin: 0;
}

html,
body {
  height: 100%; /* Memastikan body mengisi tinggi layar */
}

.hei {
  min-height: 100vh; /* Memastikan minimal tinggi container memenuhi tinggi layar */
  background: rgb(26, 26, 26) !important; /* Menggunakan warna latar belakang */
}

.card {
  color: rgb(0, 0, 0);
  height: 150px;
  background-color: rgb(26, 26, 26);
  box-shadow: 10px 15px 0px 3px lightblue;
  transition: width 0.3s, height 0.2s, background-color 0.3s; /* Tambahkan transisi untuk perubahan background */
  border: 1px solid lightblue;
}

.card:hover {
  background-color: lightblue;
  box-shadow: 0 5px #666;
  transform: translateY(4px);
}

h2 {
  font-size: 40px;
}

.bawah {
  padding-bottom: 110px;
}

.atas {
  padding-top: 70px;
  padding-bottom: 0px;
}

.btn {
  width: 100px;
  height: 50px;
}

/* Responsive Styles */
@media (max-width: 576px) {
  h2 {
    font-size: 30px; /* Mengurangi ukuran font pada perangkat kecil */
  }

  /* Menonaktifkan efek hover pada perangkat kecil */
  .card {
    transition: none; /* Menonaktifkan transisi untuk perangkat kecil */
  }

  .card:hover {
    background-color: rgb(9, 82, 141); /* Menetapkan warna latar belakang saat ditekan */
    color: black;
    box-shadow: 0 5px #666;
    transform: translateY(4px);
  }
}

@media (min-width: 576px) and (max-width: 768px) {
  .card {
    height: 150px; /* Mengurangi tinggi kartu pada perangkat medium */
  }
}

@media (min-width: 768px) {
  .card {
    height: 200px; /* Kembali ke tinggi asli pada perangkat besar */
  }
}
</style>
