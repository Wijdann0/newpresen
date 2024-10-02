<template>

  <div class="container-fluid">
    <div class="row pp text-center atas pb-2">
      <div class="col-lg-12">
        <h1>Selamat Datang</h1>
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