<template>
  <div class="container-fluid atas mt-3">
    <div class="row pt-3" id="navbar">
      <div class="col-6 col-md-3 mb-2">
        <nuxt-link to="/halamanUtama">
          <button class="btn btn-dark bck m-2 border-white w-100">Kembali</button>
        </nuxt-link>
      </div>
    </div>
    <div class="row pt-3" id="navbar">
      <div class="col-6 col-sm-3 mb-2">
        <button class="btn btn-success m-2 w-100" @click="printPage">Print</button>
      </div>
      <div class="col-6 col-sm-3 mb-2">
        <button class="btn btn-info m-2 w-100" @click="downloadPDF">Download PDF</button>
      </div>
    </div>

    <div class="row d-flex justify-content-center pb-5" id="filter">
      <div class="col-6 col-sm-3 mb-2">
        <p class="text-white text-center">Tanggal</p>
        <input v-model="tgl_awal" type="date" class="form-control form-control-lg">
      </div>
      <div class="col-6 col-sm-3 mb-2">
        <p class="text-white text-center">Kelas</p>
        <input v-model="tingkat" type="text" class="form-control form-control-lg" placeholder="Tingkat">
      </div>
      <div class="col-6 col-sm-3 mb-2">
        <p class="text-white text-center">Jurusan</p>
        <select v-model="jurusan" class="form-control form-control-lg">
          <option value="">Pilih Jurusan</option>
          <option v-for="option in jurusanOptions" :key="option.id" :value="option.nama">
            {{ option.nama }}
          </option>
        </select>
      </div>
      <div class="col-6 col-sm-3 mb-2">
        <p class="text-white text-center">No Kelas</p>
        <input v-model="kelas" type="text" class="form-control form-control-lg" placeholder="Kelas">
      </div>
    </div>

    <div class="container pp" id="content">
      <div class="text-center mb-5 text-light">
        <h1>Presensi</h1>
        <p>Kelas: {{ tingkat }} {{ jurusan }} {{ kelas }}</p>
        <p>Tanggal: {{ tgl_awal || today }}</p>
      </div>

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
            <td>{{ visitor.siswa?.tingkat || 'N/A' }} {{ visitor.jurusan?.nama || 'N/A' }} {{ visitor.siswa?.kelas || 'N/A' }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import { createClient } from '@supabase/supabase-js';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

const getTodayDate = () => {
  const today = new Date();
  const yyyy = today.getFullYear();
  const mm = String(today.getMonth() + 1).padStart(2, "0");
  const dd = String(today.getDate()).padStart(2, "0");
  return `${yyyy}-${mm}-${dd}`;
};

const supabaseUrl = 'https://lagwvzegjurpjhjvbwca.supabase.co';
const supabaseAnonKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImxhZ3d2emVnanVycGpoanZid2NhIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MjQwMzMyNTIsImV4cCI6MjAzOTYwOTI1Mn0.XrdQtFfHp4mxuJjtxE810KZnsWIS47wyc5uYY4r_KmQ';
const supabase = createClient(supabaseUrl, supabaseAnonKey);

const visitors = ref([]);
const tgl_awal = ref("");
const tingkat = ref("");
const jurusan = ref("");
const kelas = ref("");
const jurusanOptions = ref([]);
const today = getTodayDate();


const getPresensi = async (tanggal = today) => {
  try {
    let query = supabase
      .from("presensi")
      .select("tanggal, siswa(id, nama, tingkat, kelas), keterangan(nama), jurusan(nama)");

    if (tanggal) {
      query = query.eq("tanggal", tanggal);
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


onMounted(() => {
  getPresensi();  
  getJurusanOptions();
});


watch(tgl_awal, (newDate) => {
  if (newDate) {
    getPresensi(newDate);
  } else {
    getPresensi(); 
  }
});


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
    const imgWidth = 190;
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
</script>

<style scoped>

*, *::before, *::after {
  box-sizing: border-box;
}

html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow-x: hidden;
}

.container-fluid {
  background-color: rgb(26, 26, 26);
  min-height: 100vh; 
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  overflow-x: hidden; 
}

.row {
  margin-right: 0;
  margin-left: 0; /* Menghapus margin di sisi row */
}

.col-12, .col-sm-6, .col-md-3 {
  padding-right: 0;
  padding-left: 0; /* Menghapus padding di sisi kolom */
}

.table {
  padding-bottom: 100px; /* Atur padding bawah tabel */
  max-width: 100%; /* Pastikan tabel tidak meluap */
  overflow: hidden; /* Menghindari meluap */
}

.text-white {
  color: white; /* Mengatur warna teks menjadi putih */
}

.b {
  background-color: #343a40; /* Atur warna latar belakang header tabel */
}
/* Menghindari overflow pada elemen tertentu */
input[type="date"],
input[type="text"],
select {
  width: 100%; /* Pastikan input dan select tidak meluap */
  max-width: 100%; 
}

@media print {
  body * {
    visibility: hidden; /* Sembunyikan semua elemen */
  }

  #content, #content * {
    visibility: visible; /* Hanya tampilkan konten */
  }

  #content {
    top: 0;
    left: 0;
    right: 0;
    padding: 20px; /* Tambahkan padding untuk konten */
  }
  th:nth-child(2), td:nth-child(2) {
    display: none !important;
  }

  /* Sembunyikan elemen navbar dan filter */
  #navbar,
  #filter,
  .btn {
    display: none !important; /* Pastikan semua elemen ini tidak tampil */
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
    border: 2px solid black;
  }

  th, td {
    border: 1px solid black;
    padding: 8px;
    text-align: center;
  }

  thead th {
    border-bottom: 2px solid black;
    background-color: #f2f2f2;
  }

  h1, p {
    color: black;
    text-align: center; /* Atur warna dan perataan teks */
  }
}


</style>