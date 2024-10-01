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
          <p style="color: white;">Tanggal</p>
          <input v-model="tgl_awal" type="date" class="form-control form-control-lg">
        </div>
        <div class="col-lg-2">
          <p style="color: white;">Tingkat</p>
          <input v-model="tingkat" type="text" class="form-control form-control-lg" placeholder="Tingkat">
        </div>
        <div class="col-lg-2">
          <p style="color: white;">Jurusan</p>
          <select v-model="jurusan" class="form-control form-control-lg">
            <option value="">Pilih Jurusan</option>
            <option v-for="option in jurusanOptions" :key="option.id" :value="option.nama">
              {{ option.nama }}
            </option>
          </select>
        </div>
        <div class="col-lg-2">
          <p style="color: white;">Kelas</p>
          <input v-model="kelas" type="text" class="form-control form-control-lg" placeholder="Kelas">
        </div>
        <div class="col-lg-1 justify-content-center align-items-end">
          <button type="submit" class="btn btn-primary mt-5">Cari</button>
        </div>
      </div>
    </form>
    <div class="container pp" id="content">
      <table id="presensiTable" border="1" style="width:100%; text-align: center; color: white;" class="border-white">
        <tr class="b">
          <th>No</th>
          <th>Tanggal</th>
          <th>Nama</th>
          <th>Keterangan</th>
          <th>Kelas</th>
        </tr>
        <tr v-for="(visitor, i) in filteredVisitors" :key="i">
          <td>{{ i + 1 }}</td>
          <td>{{ visitor.tanggal }}</td>
          <td>{{ visitor.siswa?.nama || 'Tidak ada data' }}</td>
          <td>{{ visitor.keterangan?.nama || 'Tidak ada data' }}</td>
          <td>{{ visitor.siswa?.tingkat || 'N/A' }} {{ visitor.jurusan?.nama || 'N/A' }} {{ visitor.siswa?.kelas || 'N/A' }}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { createClient } from '@supabase/supabase-js';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

const supabaseUrl = 'https://lagwvzegjurpjhjvbwca.supabase.co';
const supabaseAnonKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImxhZ3d2emVnanVycGpoanZid2NhIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MjQwMzMyNTIsImV4cCI6MjAzOTYwOTI1Mn0.XrdQtFfHp4mxuJjtxE810KZnsWIS47wyc5uYY4r_KmQ';
const supabase = createClient(supabaseUrl, supabaseAnonKey);

const visitors = ref([]);
const tgl_awal = ref("");
const tingkat = ref("");
const jurusan = ref("");
const kelas = ref("");
const jurusanOptions = ref([]);


const getPresensi = async () => {
  try {
    let query = supabase
      .from("presensi")
      .select("*, keterangan(*), siswa(*), jurusan(*)");

    if (tgl_awal.value) {
      query = query.gte("tanggal", tgl_awal.value);
    } else {
      const today = getTodayDate();
      query = query.eq("tanggal", today);
    }

    const { data, error } = await query;

    if (error) {
      console.error("Error fetching data:", error);
    } else {
      visitors.value = data || [];
    }
  } catch (error) {
    console.error("Unexpected error:", error);
  }
};


const getTodayDate = () => {
  const today = new Date();
  const yyyy = today.getFullYear();
  const mm = String(today.getMonth() + 1).padStart(2, "0");
  const dd = String(today.getDate()).padStart(2, "0");
  return `${yyyy}-${mm}-${dd}`;
};


const filteredVisitors = computed(() => {
  return visitors.value.filter((visitor) => {
    const matchTanggal = !tgl_awal.value || visitor.tanggal === tgl_awal.value; 
    const matchTingkat = !tingkat.value || visitor.siswa?.tingkat === tingkat.value;
    const matchJurusan = !jurusan.value || visitor.jurusan?.nama === jurusan.value;
    const matchKelas = !kelas.value || visitor.siswa?.kelas === kelas.value;

    return matchTanggal && matchTingkat && matchJurusan && matchKelas;
  });
});



const printPage = () => {
  window.print();
};


const downloadPDF = () => {
  const content = document.getElementById('content');

  if (!content) {
    console.error("Element with id 'content' not found.");
    return;
  }

  html2canvas(content).then((canvas) => {
    const pdf = new jsPDF();
    const imgData = canvas.toDataURL('image/png');
    const imgWidth = 190; // Sesuaikan dengan lebar halaman PDF
    const imgHeight = (canvas.height * imgWidth) / canvas.width;
    let heightLeft = imgHeight;

    let position = 0;

    pdf.addImage(imgData, 'PNG', 10, position, imgWidth, imgHeight);
    heightLeft -= pdf.internal.pageSize.height;

    while (heightLeft >= 0) {
      position = heightLeft - imgHeight;
      pdf.addPage();
      pdf.addImage(imgData, 'PNG', 10, position, imgWidth, imgHeight);
      heightLeft -= pdf.internal.pageSize.height;
    }

    pdf.save('presensi.pdf');
  });
};

const getJurusanOptions = async () => {
  try {
    const { data, error } = await supabase
      .from("jurusan")
      .select("*");

    if (error) {
      console.error("Error fetching jurusan data:", error);
    } else {
      jurusanOptions.value = data;
    }
  } catch (error) {
    console.error("Unexpected error:", error);
  }
};


onMounted(() => {
  getPresensi();
  getJurusanOptions();
});
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
    border: 2px solid black;
    padding: 10px;
  }

  .pp {
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
