<template>
  <!DOCTYPE html>
  <html lang="en">

  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
  </head>

  <body>
    <div class="container-fluid hei">
      <div class="row my-5 d-flex justify-content-around mt-3 atas">
        <div class="col-lg-4 mb-5">
          <nuxt-link to="/presensi" class="text-decoration-none">
            <div class="card bg-isiPresensi rounded-5 text-light">
              <div class="card-body d-flex justify-content-center align-items-center">
                <h2 class="">Isi Presensi</h2>
              </div>
            </div>
          </nuxt-link>
        </div>
        <div class="col-lg-4 mb-5">
          <nuxt-link to="/laporan" class="text-decoration-none">
            <div class="card bg-laporanMingguan rounded-5 text-light">
              <div class="card-body d-flex justify-content-center align-items-center">
                <h2 class="">Laporan</h2>
              </div>
            </div>
          </nuxt-link>
        </div>
      </div>
      <div class="row pb-3 ">
        <div class="col p-3">
          <nuxt-link to="/log/logout">
            <button class="btn btn-dark bck mt-5 mb-5 border-white">LogOut</button>
          </nuxt-link>
        </div>
      </div>
    </div>
  </body>

  </html>

</template>
<style scoped>
* {
  padding: 0;
  margin: 0;
}

.hei {
  min-height: 41vmax;
  background: rgb(26, 26, 26) !important;
}

.card {
  color: white;
  height: 200px;
  background-color: rgb(26, 26, 26);
  box-shadow: 10px 15px 0px 3px lightblue;
  transition: width 3s, height 2s;
  border: 1px solid lightblue;
}

.card:hover {
  background-color: rgb(9, 82, 141);
  color: black;
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
</style>