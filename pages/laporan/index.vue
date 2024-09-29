<template>
  <div class="container-fluid atas mt-3">
    <div class="row pt-3">
      <div class="col-lg">
        <nuxt-link to="/halamanUtama">
          <button class="btn btn-dark bck mb-5 border-white">Kembali</button>
        </nuxt-link>
      </div>
    </div>
    <div class="row text-center bebas">
      <div class="col-lg-12 text-white">
        <h3>Presensi Bulanan</h3>
      </div>
    </div>
    <form @submit.prevent="getPresensi">
      <div class="row justify-content-end pb-5">
        <div class="col-lg-2">
          <p style="color: white;">Start Date</p>
          <input v-model="tgl_awal" type="date" class="form-control form-control-lg">
        </div>
        <div class="col-lg-2 ">
          <p style="color: white;">End Date</p>
          <input v-model="tgl_akhir" type="date" class="form-control form-control-lg">
        </div>
        <div class="col-lg-1 justify-content-center align-items-end ">
          <button type="submit" class="btn btn-primary mt-5">Cari</button>
        </div>
      </div>
    </form>
    <div class="container pb-5">
      <table border="1" style="width:100%; text-align: center; color: white;" class="border-white">
        <tr class="b">
          <th>No</th>
          <th>Tanggal</th>
          <th>Nama</th>
          <th>Keterangan</th>
          <th>Tingkat</th>
          <th>Jurusan</th>
          <th>Kelas</th>
        </tr>
        <tr v-for="(visitor, i) in visitors" :key="i">
          <td>{{ i + 1 }}</td>
          <td>{{ visitor.tanggal }}</td>
          <td>{{ visitor.siswa.nama }}</td>
          <td>{{ visitor.keterangan.nama }}</td>
          <td>{{ visitor.siswa.tingkat }}</td>
          <td>{{ visitor.siswa.jurusan }}</td>
          <td>{{ visitor.siswa.kelas }}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script setup>
const supabase = useSupabaseClient()
const visitors = ref([])
const tgl_awal = ref('')
const tgl_akhir = ref('')

const getPresensi = async () => {
  if (tgl_awal.value && tgl_akhir.value) {
    let { data, error } = await supabase
      .from('presensi')
      .select('*,keterangan(*),siswa(*)')
      .gte('tanggal', tgl_awal.value)
      .lte('tanggal', tgl_akhir.value)
    if (data) visitors.value = data
    if (error) console.error(error)
  } else {
    let { data, error } = await supabase
      .from('presensi')
      .select('*,keterangan(*),siswa(*)')
    if (data) visitors.value = data
    if (error) console.error(error)
  }
}

onMounted(() => {
  getPresensi()
})
</script>

<style scoped>
.bck {
  text-decoration: none;
}

.bebas {
  padding-top: 50px;
}

.atas {
  min-height: 10rem;
  background: rgb(26, 26, 26);
}

td {
  border-bottom: 3px solid #fff;
  padding: 5px;
}

tr.b {
  border-bottom: 2px solid #fff;
  background: gray;
}
</style>
