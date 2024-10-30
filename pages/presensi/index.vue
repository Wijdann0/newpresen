[ <template>
  <div class="container-fluid">
    <div class="row d-flex justify-content-center pt-5">
      <div class="col-lg-8 col-md-10 col-sm-12">
        <form @submit.prevent="kirimData">
          <div class="mb-4">
            <select v-model="form.tingkat" class="jurusan form-control form-control-lg rounded-4 fs-5 text-center">
              <option value="" disabled selected>Kelas</option>
              <option value="10">10</option>
              <option value="11">11</option>
              <option value="12">12</option>
            </select>
          </div>
          <div class="mb-4">
            <div class="row md-5">
              <div class="col-md-6 mb-3 mb-md-0">
                <select v-model="form.jurusan" :disabled="form.tingkat === ''"
                  class="jurusan form-control form-control-lg rounded-4 fs-5 text-center">
                  <option value="" disabled selected>Jurusan</option>
                  <option v-for="(jurus, i) in op" :key="i" :value="jurus.id">{{ jurus.nama }}</option>
                </select>
              </div>
              <div class="col-md-6">
                <select @change="cek" v-model="form.kelas"
                  class="jurusan form-control form-control-lg rounded-4 fs-5 text-center">
                  <option value="" disabled selected>No Kelas</option>
                  <option value="1">1</option>
                  <option value="2">2</option>
                  <option value="3">3</option>
                  <option value="4">4</option>
                </select>
              </div>
            </div>
          </div>

          <!-- Show students and their corresponding keterangan (radio buttons) -->
          <div class="row" v-if="form.kelas">
            <div class="col-12 d-flex justify-content-center">
              <div class="w-100 text-center text-light">
                <h5 class="pb-5">Nama dan Keterangan</h5>
                <div v-for="(member, i) in members" :key="i" class="d-flex align-items-center mb-3">
                  <!-- Student Name -->
                  <span class="me-3" style="flex: 1; text-align: left;">{{ member.nama }}</span>

                  <!-- Keterangan (Radio Buttons) -->
                  <div style="flex: 2;">
                    <div v-for="(item, j) in objectives" :key="j" class="form-check d-inline-block me-2">
                      <input type="radio" :id="'keterangan-' + item.id + '-' + member.id"
                        :name="'keterangan-' + member.id" :value="item.id" v-model="form.siswaKeterangan[member.id]"
                        class="form-check-input">
                      <label :for="'keterangan-' + item.id + '-' + member.id" class="form-check-label">{{ item.nama
                        }}</label>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Submit Button -->
            <div class="col-12 d-flex justify-content-center ">
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

const members = ref([])
const objectives = ref([])
const op = ref([])

const form = ref({
  siswaKeterangan: {},  // Object to hold keterangan for each siswa
  tingkat: "",
  jurusan: "",
  kelas: ""
})

// ID dari keterangan default "hadir"
let defaultKeteranganId = null

async function cek(event) {
  form.value.kelas = event.target.value
  if (form.value.tingkat && form.value.jurusan && form.value.kelas) {
    let { data, error } = await supabase
      .from('siswa')
      .select('*')
      .eq('tingkat', form.value.tingkat)
      .eq('jurusan', form.value.jurusan)
      .eq('kelas', form.value.kelas)
      .order('nama', { ascending: true })
    if (data) {
      members.value = data
      // Set nilai default "hadir" untuk setiap siswa
      form.value.siswaKeterangan = data.reduce((acc, member) => {
        acc[member.id] = defaultKeteranganId
        return acc
      }, {})
    }
  } else {
    members.value = []
  }
}

const getSiswa = async () => {
  const { data, error } = await supabase
    .from('siswa')
    .select('*')
  if (data) members.value = data
}

const getKeterangan = async () => {
  const { data, error } = await supabase
    .from('keterangan')
    .select('*')
  if (data) {
    objectives.value = data
    // Cari ID keterangan "hadir" dan simpan sebagai defaultKeteranganId
    const hadirKeterangan = data.find((item) => item.nama.toLowerCase() === 'hadir')
    defaultKeteranganId = hadirKeterangan ? hadirKeterangan.id : null
  }
}

const getJurusan = async () => {
  const { data, error } = await supabase
    .from('jurusan')
    .select("*")
  if (data) op.value = data
}

const kirimData = async () => {
  const entries = []

  // Iterate through each student and their keterangan
  for (const [siswaId, keteranganId] of Object.entries(form.value.siswaKeterangan)) {
    if (keteranganId) {
      entries.push({
        siswa: siswaId,
        keterangan: keteranganId,
        jurusan: form.value.jurusan
      })
    }
  }

  const { error } = await supabase
    .from('presensi')
    .insert(entries)

  if (!error) {
    alert('Presensi berhasil!')

    // Remove students that have been assigned keterangan
    const studentsToRemove = Object.keys(form.value.siswaKeterangan)
    members.value = members.value.filter(member => !studentsToRemove.includes(String(member.id)))

    // Reset form
    form.value.siswaKeterangan = {}
  }
}

onMounted(() => {
  getSiswa()
  getKeterangan()
  getJurusan()
})
</script>


<style scoped>
body {
  font-size: 18px;
}

.container-fluid {
  background: rgb(26, 26, 26);
  min-height: 90vh;
  padding: 0 20px;
  /* Tambahkan padding di sisi */
}

.jurusan {
  height: auto;
  color: white;
  background: rgb(26, 26, 26) !important;
  box-shadow: 5px 5px 0px 2px lightblue;
  border: 2px solid lightblue;
  transition: background-color 0.3s ease, border-color 0.3s ease;
  width: 100%;
  text-align: center;
  font-size: 18px;
}

.jurusan:hover {
  background-color: lightblue;
  /* Warna latar belakang berubah saat hover */
  border-color: blue;
  /* Warna border berubah saat hover */
}

.row.md-5 {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  justify-content: center;
}

.col-md-6 {
  flex: 1;
  min-width: 240px;
  /* Menjaga lebar minimal agar responsif */
}

.krm {
  width: 150px;
  height: 50px;
  box-shadow: 5px 5px 0px 1.5px lightblue;
  border: 2px solid lightblue;
  background: rgb(26, 26, 26);
  transition: background-color 0.3s ease, border-color 0.3s ease;
  font-size: 18px;
}

.krm:hover {
  background-color: lightblue;
  border-color: blue;
}

@media (max-width: 768px) {

  /* Flex container yang meresponsif untuk tampilan mobile */
  body {
    font-size: 14px;
  }

  .row.md-5 {
    flex-direction: column;
  }

  .col-md-6 {
    flex: 1;
    min-width: 100%;
    margin-bottom: 10px;
  }

  .krm {
    width: 100px;
    height: 40px;
    font-size: 14px;
  }

  .jurusan {
    font-size: 14px;
    width: 100%;
  }

  .container-fluid {
    padding: 10px;
  }
}
</style>
