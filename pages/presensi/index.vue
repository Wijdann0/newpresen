<template>
  <div class="container-fluid">
    <div class="row d-flex justify-content-center pt-5">
      <div class="col-lg-8 col-md-10 col-sm-12">
        <form @submit.prevent="kirimData">
          <div class="mb-4">
            <select v-model="form.tingkat" class="jurusan form-control form-control-lg rounded-4 fs-5 text-center"
              data-bs-toggle="tooltip" data-bs-placement="top" title="Masukan Kelas">
              <option value="">Kelas</option>
              <option value="10">10</option>
              <option value="11">11</option>
              <option value="12">12</option>
            </select>
          </div>
          <div class="mb-4">
            <div class="row mb-5">
              <div class="col-md-6 mb-3 mb-md-0">
                <select v-model="form.jurusan" :disabled="form.tingkat === ''"
                  class="jurusan form-control form-control-lg rounded-4 fs-5 text-center" data-bs-toggle="tooltip"
                  data-bs-placement="top" title="Masukan Jurusan">
                  <option value="">Jurusan</option>
                  <option v-for="(jurus, i) in op" :key="i" :value="jurus.id">{{ jurus.nama }}</option>
                </select>
              </div>
              <div class="col-md-6">
                <select @change="cek" v-model="form.kelas"
                  class="jurusan form-control form-control-lg rounded-4 fs-5 text-center" data-bs-toggle="tooltip"
                  data-bs-placement="top" title="Masukan Nomor Kelas">
                  <option value="">No Kelas</option>
                  <option value="1">1</option>
                  <option value="2">2</option>
                  <option value="3">3</option>
                  <option value="4">4</option>
                </select>
              </div>
            </div>
          </div>
          <div class="row text-white" v-if="form.kelas"> <!-- Only show students if kelas is selected -->
            <div class="col-12 d-flex justify-content-center">
              <div class="w-100 text-center">
                <h5>Nama</h5>
                <div v-for="(member, i) in members" :key="i" class="form-check">
                  <input type="checkbox" :id="'siswa-' + member.id" :value="member.id" v-model="form.siswa"
                    class="form-check-input" @change="toggleKeterangan(member.id)">
                  <label :for="'siswa-' + member.id" class="form-check-label">{{ member.nama }}</label>
                  <div v-if="form.siswa.includes(member.id)">
                    <h6>Keterangan untuk {{ member.nama }}</h6>
                    <div v-for="(item, j) in objectives" :key="j" class="form-check">
                      <input type="checkbox" :id="'keterangan-' + item.id + '-' + member.id" :value="item.id"
                        v-model="form.siswaKeterangan[member.id]" class="form-check-input">
                      <label :for="'keterangan-' + item.id + '-' + member.id" class="form-check-label">{{ item.nama
                        }}</label>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <div class="col-12 d-flex justify-content-center pt-5">
              <button type="submit" class="btn btn-success krm" data-bs-toggle="tooltip" data-bs-placement="top"
                title="Kirim Presensi">Kirim</button>
            </div>
          </div>
        </form>
      </div>
    </div>
    <nuxt-link to="/halamanUtama">
      <button type="button" class="btn btn-dark bck mt-5 mb-5 border-white" data-bs-toggle="tooltip"
        data-bs-placement="top" title="Kembali Ke Halaman Utama">Kembali</button>
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
  siswa: [],
  siswaKeterangan: {},  // Object to hold keterangan for each siswa
  tingkat: "",
  jurusan: "",
  kelas: ""
})

async function cek(event) {
  form.value.kelas = event.target.value
  if (form.value.tingkat && form.value.jurusan && form.value.kelas) {
    let { data, error } = await supabase
      .from('siswa')
      .select('*')
      .eq('tingkat', form.value.tingkat)
      .eq('jurusan', form.value.jurusan)
      .eq('kelas', form.value.kelas)
    if (data) {
      members.value = data
    }
  } else {
    members.value = [] // Clear members if kelas is not selected
  }
}

const toggleKeterangan = (memberId) => {
  if (!form.value.siswaKeterangan[memberId]) {
    form.value.siswaKeterangan[memberId] = []
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
  const entries = []

  form.value.siswa.forEach(siswaId => {
    const keteranganIds = form.value.siswaKeterangan[siswaId] || []
    keteranganIds.forEach(keteranganId => {
      entries.push({
        siswa: siswaId,
        keterangan: keteranganId,
        jurusan: form.value.jurusan
      })
    })
  })

  const { error } = await supabase
    .from('presensi')
    .insert(entries)

  if (!error) {
    alert('Presensi berhasil!')

    members.value = members.value.filter(member => !form.value.siswa.includes(member.id))

    form.value.siswa = []
    form.value.siswaKeterangan = {}
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

.jurusan {
  height: auto;
  color: white;
  background: rgb(26, 26, 26) !important;
  box-shadow: 10px 10px 0px 2px lightblue;
  border: 2px solid lightblue;
}

.jurusan:hover {
  background-color: rgb(9, 82, 141);
  color: white;
  box-shadow: 0 5px #666;
  transform: translateY(4px);
}

.nama {
  color: white;
  background: rgb(26, 26, 26) !important;
  box-shadow: 10px 10px 0px 3px lightblue;
  border: 2px solid lightblue;
}

.krm {
  width: 150px;
  height: 50px;
  box-shadow: 5px 5px 0px 1.5px lightblue;
  border: 2px solid lightblue;
  background: rgb(26, 26, 26);
}

@media (max-width: 768px) {
  .krm {
    width: 100px;
    height: 40px;
  }

  .jurusan,
  .nama {
    font-size: 16px;
  }

  .container-fluid {
    padding: 10px;
  }
}
</style>
