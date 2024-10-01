<template>
  <!DOCTYPE html>
  <html lang="en">

  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
  </head>

  <body>
    <div class="container-fluid">
      <div class="row pp text-center atas pb-5">
        <div class="col-lg-12">
          <h1>Hallo Selamat Datang</h1>
        </div>
      </div>
      <div class="row text-center">
        <div class="col">
          <h4>Login terlebih dahulu untuk melakukan presensi</h4>
        </div>
      </div>
      <div class="row log text-center">
        <div class="col-lg-12">
          <nuxt-link to="/log/login">
            <button class="btn btn-success">Login</button>
          </nuxt-link>
        </div>
      </div>
    </div>
  </body>

  </html>
</template>

<style scoped>
.log {
  padding-top: 100px;
}

.atas {
  padding-top: 150px;
}

button {
  width: 150px;
}
</style>