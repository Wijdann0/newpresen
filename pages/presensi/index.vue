<template>

  <div class="container-fluid">
    <div class="row d-flex justify-content-center pt-5">
      <div class="col-lg-8">
        <form @submit.prevent="kirimData">
          <div class="mb-4">
            <select v-model="form.tingkat" class="jurusan form-control form-control-lg rounded-4 fs-3 text-center">
              <option value="">Tingkat</option>
              <option value="10">10</option>
              <option value="11">11</option>
              <option value="12">12</option>
            </select>
          </div>
          <div class="mb-4">
            <div class="row mb-5">
              <div class="col-md-6">
                <select v-model="form.jurusan" :disabled="form.tingkat == ''"
                  class="jurusan form-control form-control-lg rounded-4 fs-3 text-center ">
                  <option value="">JURUSAN</option>
                  <option v-for="(jurus, i) in op" :key="i" :value="jurus.id">{{ jurus.nama }}</option>
                </select>
              </div>
              <div class="col-md-6">
                <select @change="cek" class="jurusan form-control form-control-lg rounded-4 fs-3 text-center">
                  <option value="">KELAS</option>
                  <option value="1">1</option>
                  <option value="2">2</option>
                  <option value="3">3</option>
                  <option value="4">4</option>
                </select>
              </div>
            </div>
          </div>
          <div class="row">
            <div class=" col-lg-12 d-flex justify-content-center">
              <select v-model="form.siswa" class="form-control form-control-lg nama fs-3 text-center mb-5">
                <option value="">Nama</option>
                <option v-for="(member, i) in members" :key="i" :value="member.id">{{ member.nama }}</option>
              </select>
            </div>
            <div class="col-lg-12 d-flex justify-content-center">
              <select v-model="form.keterangan" :disabled="form.jurusan == ''"
                class="form-control form-control-lg nama fs-3 text-center mb-5">
                <option value="">Keterangan</option>
                <option v-for="(item, i) in objectives" :key="i" :value="item.id">{{ item.nama }}</option>
              </select>
            </div>
            <div class="col d-flex justify-content-center">
              <button type="submit" class="btn btn-success krm">Kirim</button>
            </div>
          </div>
        </form>
      </div>
    </div>
    <nuxt-link to="/halamanUtama">
      <button type="button" class="btn btn-dark bck mt-5 mb-5 border-white">Kembali</button>
    </nuxt-link>
  </div>


</template>

<script setup>
const supabase = useSupabaseClient()

const kelas = ref([])
const members = ref([])
const objectives = ref([])
const op = ref([])

const form = ref({
  siswa: "",
  tingkat: "",
  jurusan: "",
  kelas: "",
  keterangan: ""
})

async function cek(event) {
  form.value.kelas = event.target.value
  if (form.value != '') {
    let { data, error } = await supabase
      .from('siswa')
      .select('*')
      .eq('tingkat', form.value.tingkat)
      .eq('jurusan', form.value.jurusan)
      .eq('kelas', form.value.kelas)
    if (data) {
      members.value = data
    }
  }
}

const getSiswa = async () => {
  const { data, error } = await supabase
    .from('siswa')
    .select('*')
  if (data) members.value = data
}

const getKelas = async () => {
  const { data, error } = await supabase
    .from('siswa')
    .select('*, kelas')
  if (data) kelas.value = data
}

const getKeterangan = async () => {
  const { data, error } = await supabase
    .from('keterangan')
    .select('*')
  if (data) objectives.value = data
}

const kirimData = async () => {
  const { error } = await supabase
    .from('presensi')
    .insert([
      {
        siswa: form.value.siswa,
        keterangan: form.value.keterangan,
        jurusan: form.value.jurusan
      }
    ])

  if (!error) {
    alert('Presensi berhasil!')

    // Hapus siswa yang dipilih dari list
    members.value = members.value.filter(member => member.id !== form.value.siswa)

    // Reset form
    form.value.siswa = ''
    form.value.keterangan = ''
  }
}

const getJurusan = async () => {
  const { data, error } = await supabase
    .from('jurusan')
    .select("*")
  if (data) op.value = data
}

onMounted(() => {
  getSiswa()
  getKelas()
  getKeterangan()
  getJurusan()
})
</script>


<style scoped>
.container-fluid {
  background: rgb(26, 26, 26);
  min-height: 90vh;
}

.hei {
  min-height: 90vh;
  background: rgb(26, 26, 26);
}

.jurusan {
  height: 100px;
  color: white;
  background: rgb(26, 26, 26) !important;
  box-shadow: 10px 10px 0px 2px lightblue;
  border: 2px solid lightblue;
}

.link {
  width: 500px;
  height: 50px;
  color: white;
  background-color: black;
  box-shadow: 10px 10px 0px 5px lightblue;
  border: 2px solid lightblue;
  transition: width 1.5s, height 1.5s;
  text-decoration-style: none;
}

.atas {
  padding-top: 100px;
}

.nama {
  color: white;
  background: rgb(26, 26, 26) !important;
  width: 500px;
  height: 70px;
  box-shadow: 10px 10px 0px 3px lightblue;
  border: 2px solid lightblue;
}

.krm {
  width: 100px;
  height: 50px;
  box-shadow: 5px 5px 0px 1.5px lightblue;
  border: 2px solid lightblue;
  background: rgb(26, 26, 26);
}
</style>