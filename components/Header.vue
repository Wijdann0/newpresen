<template>
  <div class="container-fluid">
    <div class="row bg-white d-flex justify-content-center align-items-center height m-1 pt-1">
      <div class="col-2 col-md-1 d-flex justify-content-end">
        <img src="~/assets/logo.webp" alt="logo" class="img-fluid" style="max-width: 50px; padding-right: 10px;">
      </div>
      <div class="col-10 col-md-11 p-0 m-0 d-flex flex-column justify-content-center">
        <h1 class="pd fs-5 fs-md-5 text-black text-decoration-none"><u>Presensi Kelas</u></h1>
        <address class="p-0 m-0 clr fs-7 fs-md-6 text-black">
          SMKN 4 Tasikmalaya
        </address>
      </div>
    </div>
  </div>
</template>

<style scoped>
* {
  padding: 0;
  margin: 0;
}

.pd {
  text-decoration: none;
}

.height {
  height: 80px;
}

@media (max-width: 576px) {
  .height {
    height: 50px;
    /* Mengurangi tinggi pada layar kecil */
  }

  h1 {
    font-size: 16px;
    /* Ukuran font lebih kecil di ponsel */
  }

  address {
    font-size: 12px;
    /* Ukuran font lebih kecil di ponsel */
  }
}
</style>
