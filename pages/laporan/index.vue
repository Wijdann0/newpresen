<template>
  <div class="container-fluid atas mt-3">
    <div class="row pt-3">
      <div class="col-12 col-sm-6 col-md-3 mb-2">
        <nuxt-link to="/halamanUtama">
          <button class="btn btn-dark bck m-2 border-white w-100">Kembali</button>
        </nuxt-link>
      </div>
      <div class="col-12 col-sm-6 col-md-3 mb-2">
        <button class="btn btn-success m-2 w-100" @click="printPage">Print</button>
      </div>
      <div class="col-12 col-md-3 mb-2">
        <button class="btn btn-info m-2 w-100" @click="downloadPDF">Download PDF</button>
      </div>
    </div>
    <div class="row text-center bebas pb-5">
      <div class="col-12 text-white">
        <h3>Presensi</h3>
      </div>
    </div>
    <form @submit.prevent="getPresensi">
      <div class="row justify-content-end pb-5">
        <div class="col-12 col-md-3 mb-2">
          <p style="color: white; text-align: center;">Tanggal</p>
          <input v-model="tgl_awal" type="date" class="form-control form-control-lg">
        </div>
        <div class="col-12 col-md-3 mb-2">
          <p style="color: white; text-align: center;">Kelas</p>
          <input v-model="tingkat" type="text" class="form-control form-control-lg" placeholder="Tingkat">
        </div>
        <div class="col-12 col-md-3 mb-2">
          <p style="color: white; text-align: center;">Jurusan</p>
          <select v-model="jurusan" class="form-control form-control-lg">
            <option value="">Pilih Jurusan</option>
            <option v-for="option in jurusanOptions" :key="option.id" :value="option.nama">
              {{ option.nama }}
            </option>
          </select>
        </div>
        <div class="col-12 col-md-3 mb-2">
          <p style="color: white; text-align: center;">No Kelas</p>
          <input v-model="kelas" type="text" class="form-control form-control-lg" placeholder="Kelas">
        </div>
        <div class="col-12 col-md-3 justify-content-center align-items-end mb-2">
          <button type="submit" class="btn btn-primary mt-5 w-100">Cari</button>
        </div>
      </div>
    </form>
    <div class="container pp" id="content">
      <table id="presensiTable" class="table table-bordered text-white">
        <thead>
          <tr class="b">
            <th>No</th>
            <th>Tanggal</th>
            <th>Nama</th>
            <th>Keterangan</th>
            <th>Kelas</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(visitor, i) in filteredVisitors" :key="i">
            <td>{{ i + 1 }}</td>
            <td>{{ visitor.tanggal }}</td>
            <td>{{ visitor.siswa?.nama || 'Tidak ada data' }}</td>
            <td>{{ visitor.keterangan?.nama || 'Tidak ada data' }}</td>
            <td>{{ visitor.siswa?.tingkat || 'N/A' }} {{ visitor.jurusan?.nama || 'N/A' }} {{ visitor.siswa?.kelas ||
              'N/A' }}</td>
          </tr>
        </tbody>
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
    // Ambil tanggal hari ini
    const today = getTodayDate();
    console.log("Tanggal hari ini:", today); // Debugging: Tampilkan tanggal yang dikirim

    // Set input tanggal ke hari ini secara otomatis
    tgl_awal.value = today; 

    // Query untuk mengambil data dari tabel presensi dengan join ke tabel siswa, jurusan, dan keterangan
    let query = supabase
      .from("presensi")
      .select("tanggal, siswa(id, nama, tingkat, kelas), keterangan(nama), jurusan(nama)");

    // Filter berdasarkan tanggal hari ini
    query = query.eq("tanggal", today);

    console.log("Query yang dikirim:", query); // Debugging: Tampilkan query

    // Eksekusi query dan tangani hasil atau error
    const { data, error } = await query;

    if (error) {
      console.error("Error fetching data:", error);
    } else {
      visitors.value = data || [];
      console.log("Data fetched:", visitors.value); 
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
  console.log("Filtered Visitors:", visitors.value); // Debugging: Tampilkan visitor yang difilter
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
    const imgWidth = 190; // Adjust based on PDF page width
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
  padding-bottom: 270px;
}

/* Style for printing */
@media print {
  body * {
    visibility: hidden;
  }
  #content, #content * {
    visibility: visible;
  }
  #content {
    position: absolute;
    top: 0;
  }
}
</style>
