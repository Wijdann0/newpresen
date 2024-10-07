<template>
  <div class="container-fluid psh">
    <div class="row text-center pp">
      <div class="col">
        <h1 class="text-white">Presensi Kelas</h1>
      </div>
    </div>
    <div class="row justify-content-center ps">
      <div class="col-lg-6 d-flex justify-content-center">
        <div class="card rounded-5 text-center pt-4">
          <h3 class="text-white">Login</h3>
          <div class="card-body d-flex justify-content-center pt-5">
            <form @submit.prevent="log">
              <label for="exampleInputEmail"></label>
              <input v-model="email" type="email" class="form-control form-control-lg rounded-3 akn text-center"
                id="exampleInputEmail" placeholder="Email">
              <label for="exampleInputPassword"></label>
              <input v-model="password" type="password" class="form-control form-control-lg rounded-3 akn text-center"
                id="exampleInputPassword" placeholder="Password">
              <button class="btn btn-success" type="submit">Login</button>
            </form>
          </div>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col text-white text-center pb-5">
        <p>&copy; CopyRightBy@Di-Tech</p>
      </div>
    </div>
    <div class="row kbl">
      <div class="col-lg-3">
        <nuxt-link to="/">
          <button class="btn btn-light">Kembali</button>
        </nuxt-link>
      </div>
    </div>
  </div>
</template>

<script setup>
definePageMeta({
  layout: 'login'
})

const supa = useSupabaseClient()
const email = ref('')
const password = ref('')

async function log() {
  const { data, error } = await supa.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  })

  if (!error) {
    navigateTo('/halamanUtama')
  } else {
    alert('Email or Password Invalid')
  }
}
</script>

<style scoped>
.psh {
  min-height: 100vh;
  background: rgb(26, 26, 26) !important;
}

.kbl {
  padding-top: 20px;
  padding-left: 40px;
  padding-bottom: 70px;
}

.btn {
  width: 150px;
  height: 40px;
}

.akn {
  width: 300px;
  margin-bottom: 30px;
}

* {
  margin: 0px;
  padding: 0px;
}

.pp {
  padding-top: 70px;
  padding-bottom: 50px
}

.card {
  width: 400px;
  height: 400px;
  box-shadow: 15px 15px 0px 5px gray;
  border: 2px solid gray;
  background: rgb(26, 26, 26);
}

.ps {
  padding-bottom: 50px;
}
</style>
