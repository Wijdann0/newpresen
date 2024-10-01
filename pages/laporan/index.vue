<!-- <template>

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
        <div class="col-lg-2">
          <p style="color: white;">End Date</p>
          <input v-model="tgl_akhir" type="date" class="form-control form-control-lg">
        </div>
        <div class="col-lg-1 justify-content-center align-items-end">
          <button type="submit" class="btn btn-primary mt-5">Cari</button>
        </div>
      </div>
    </form>
    <div class="container pp">
      <table border="1" style="width:100%; text-align: center; color: white;" class="border-white">
        <tr class="b">
          <th>No</th>
          <th>Tanggal</th>
          <th>Nama</th>
          <th>Keterangan</th>
          <th>Kelas</th>
        </tr>
        <tr v-for="(visitor, i) in visitors" :key="i">
          <td>{{ i + 1 }}</td>
          <td>{{ visitor.tanggal }}</td>
          <td>{{ visitor.siswa.nama }}</td>
          <td>{{ visitor.keterangan.nama }}</td>
          <td>{{ visitor.siswa.tingkat }} {{ visitor.jurusan.nama }} {{ visitor.siswa.kelas }}</td>
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

// Function to get today's date in YYYY-MM-DD format
const getTodayDate = () => {
  const today = new Date()
  const yyyy = today.getFullYear()
  const mm = String(today.getMonth() + 1).padStart(2, '0') // Months start from 0!
  const dd = String(today.getDate()).padStart(2, '0')
  return `${yyyy}-${mm}-${dd}`
}

// Fetch presensi data
const getPresensi = async () => {
  // If date range is provided, search within that range
  if (tgl_awal.value && tgl_akhir.value) {
    let { data, error } = await supabase
      .from('presensi')
      .select('*,keterangan(*),siswa(*),jurusan(*)')
      .gte('tanggal', tgl_awal.value)
      .lte('tanggal', tgl_akhir.value)
    if (data) visitors.value = data
    if (error) console.error(error)
  } else {
    // Default to fetching today's data
    const today = getTodayDate()
    let { data, error } = await supabase
      .from('presensi')
      .select('*,keterangan(*),siswa(*),jurusan(*)')
      .eq('tanggal', today)
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
  padding: 10px;
}

tr.b {
  border-bottom: 2px solid #fff;
  background: rgb(0, 0, 0);
}

.pp {
  padding-bottom: 250px;
}
</style> -->


<template>
  <div class="container-fluid atas mt-3">
    <div class="row pt-3">
      <div class="col-lg-1">
        <nuxt-link to="/halamanUtama">
          <button class="btn btn-dark bck m-2 border-white">Kembali</button>
        </nuxt-link>
      </div>
      <div class="col-lg-2">
        <button class="btn btn-success m-2" @click="printPage">Print</button>
        <button class="btn btn-info m-2" @click="downloadPDF">Download</button>
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
        <div class="col-lg-2">
          <p style="color: white;">End Date</p>
          <input v-model="tgl_akhir" type="date" class="form-control form-control-lg">
        </div>
        <div class="col-lg-1 justify-content-center align-items-end">
          <button type="submit" class="btn btn-primary mt-5">Cari</button>
        </div>
      </div>
    </form>
    <div class="container pp">
      <table id="presensiTable" border="1" style="width:100%; text-align: center; color: white;" class="border-white">
        <tr class="b">
          <th>No</th>
          <th>Tanggal</th>
          <th>Nama</th>
          <th>Keterangan</th>
          <th>Kelas</th>
        </tr>
        <tr v-for="(visitor, i) in visitors" :key="i">
          <td>{{ i + 1 }}</td>
          <td>{{ visitor.tanggal }}</td>
          <td>{{ visitor.siswa.nama }}</td>
          <td>{{ visitor.keterangan.nama }}</td>
          <td>{{ visitor.siswa.tingkat }} {{ visitor.jurusan.nama }} {{ visitor.siswa.kelas }}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script setup>
import jsPDF from 'jspdf'
import html2canvas from 'html2canvas'

const supabase = useSupabaseClient()
const visitors = ref([])
const tgl_awal = ref('')
const tgl_akhir = ref('')

const getTodayDate = () => {
  const today = new Date()
  const yyyy = today.getFullYear()
  const mm = String(today.getMonth() + 1).padStart(2, '0')
  const dd = String(today.getDate()).padStart(2, '0')
  return `${yyyy}-${mm}-${dd}`
}

const getPresensi = async () => {
  if (tgl_awal.value && tgl_akhir.value) {
    let { data, error } = await supabase
      .from('presensi')
      .select('*,keterangan(*),siswa(*),jurusan(*)')
      .gte('tanggal', tgl_awal.value)
      .lte('tanggal', tgl_akhir.value)
    if (data) visitors.value = data
    if (error) console.error(error)
  } else {
    const today = getTodayDate()
    let { data, error } = await supabase
      .from('presensi')
      .select('*,keterangan(*),siswa(*),jurusan(*)')
      .eq('tanggal', today)
    if (data) visitors.value = data
    if (error) console.error(error)
  }
}

const printPage = () => {
  window.print()
}

// Download as PDF
const downloadPDF = async () => {
  const table = document.getElementById('presensiTable')
  
  // Capture the HTML table using html2canvas
  const canvas = await html2canvas(table, { scale: 2 })
  
  // Get the image data from canvas
  const imgData = canvas.toDataURL('image/png')

  // Initialize jsPDF
  const pdf = new jsPDF('p', 'mm', 'a4')

  // PDF page size
  const pageWidth = pdf.internal.pageSize.getWidth()
  const pageHeight = pdf.internal.pageSize.getHeight()

  // Adjust image size to fit PDF
  const imgWidth = pageWidth - 20 // Maintain margin
  const imgHeight = (canvas.height * imgWidth) / canvas.width

  let heightLeft = imgHeight
  let position = 0

  // Add first page
  pdf.addImage(imgData, 'PNG', 10, position, imgWidth, imgHeight)
  heightLeft -= pageHeight

  // Add new pages if content overflows
  while (heightLeft >= 0) {
    position = heightLeft - imgHeight
    pdf.addPage()
    pdf.addImage(imgData, 'PNG', 10, position, imgWidth, imgHeight)
    heightLeft -= pageHeight
  }

  // Save the generated PDF
  pdf.save('presensi.pdf')
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
  padding: 10px;
}

tr.b {
  border-bottom: 2px solid #fff;
  background: rgb(0, 0, 0);
}

.pp {
  padding-bottom: 250px;
}

/* Style for printing */
@media print {
  body * {
    visibility: hidden;
  }

  table {
    visibility: visible;
    width: 100%;
    border-collapse: collapse;
    color: black;
  }

  table,
  th,
  td {
    border: 1px solid black;
    padding: 10px;
  }

  .pp {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    padding-bottom: 0;
  }

  .pp,
  .pp * {
    visibility: visible;
  }

  .bck,
  .btn-success,
  .btn-primary,
  form,
  .bebas,
  .row.pt-3 {
    display: none;
  }
}
</style>
