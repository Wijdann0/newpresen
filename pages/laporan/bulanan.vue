<template>
    <div class="container-fluid atas mt-3">
      <div class="row pt-3" id="navbar">
        <div class="col-6 col-sm-3 mb-2">
          <nuxt-link to="/halamanUtama">
            <button class="btn btn-dark bck mb-5 border-white w-100 h-40">Kembali</button>
          </nuxt-link>
        </div>
      </div>
      <div class="row" id="navbar">
        <div class="col-6 col-sm-3 mb-2">
          <button class="btn btn-success w-100" @click="printPage">Print</button>
        </div>
        <div class="col-6 col-sm-3 mb-2">
          <button class="btn btn-info w-100" @click="downloadPDF">Download PDF</button>
        </div>
      </div>
  
      <div class="row d-flex justify-content-center pb-5" id="filter">
        <div class="col-6 col-sm-3 mb-2">
          <p class="text-white text-center">Bulan</p>
          <input v-model="tgl_awal" type="month" class="form-control form-control-lg">
        </div>
        <div class="col-6 col-sm-3 mb-2">
          <p class="text-white text-center">Kelas</p>
          <input v-model="tingkat" type="text" class="form-control form-control-lg" placeholder="Tingkat">
        </div>
        <div class="col-6 col-sm-3 mb-2">
          <p class="text-white text-center">Jurusan</p>
          <select v-model="jurusan" class="form-control form-control-lg">
            <option value="" disabled selected>Pilih Jurusan</option>
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
          <h1 style="font-weight: bold;">Presensi Bulanan</h1>
          <p>Kelas: {{ tingkat }} {{ jurusan }} {{ kelas }}</p>
          <p>Bulan: {{ tgl_awal || today }}</p>
        </div>
        <div class="table-container">
          <table id="presensiTable" class="table table-bordered text-white">
            <thead>
              <tr class="b">
                <th>No</th>
                <th>Nama</th>
                <th>S</th>
                <th>I</th>
                <th>A</th>
                <th>Jumlah</th>
                <th>Kelas</th>
                <th>Total Hadir Pada Bulan Ini</th> <!-- Kolom Total Hari -->
              </tr>
            </thead>
            <tbody>
              <tr v-for="(visitor, i) in groupedVisitors" :key="i">
                <td>{{ i + 1 }}</td>
                <td>{{ visitor.nama }}</td>
                <td>{{ visitor.sakit }}</td>
                <td>{{ visitor.izin }}</td>
                <td>{{ visitor.alfa }}</td>
                <td>{{ visitor.total }}</td>
                <td>{{ visitor.keterangan }}</td>
                <td>{{ visitor.totalHari }}</td> 
              </tr>
            </tbody>
          </table>
        </div>
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
  
  const getDaysInMonth = (year, month) => {
    return new Date(year, month, 0).getDate(); // Mendapatkan jumlah hari dalam bulan
  };
  
  const supabaseUrl = 'https://lagwvzegjurpjhjvbwca.supabase.co';
  const supabaseAnonKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImxhZ3d2emVnanVycGpoanZid2NhIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MjQwMzMyNTIsImV4cCI6MjAzOTYwOTI1Mn0.XrdQtFfHp4mxuJjtxE810KZnsWIS47wyc5uYY4r_KmQ'; // Ganti dengan key Anda
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
        const [year, month] = tanggal.split("-");
        const startDate = `${year}-${month}-01`;
        const endDate = `${year}-${month}-${getDaysInMonth(year, month)}`;
        query = query.gte("tanggal", startDate).lte("tanggal", endDate);
      }
  
      const { data, error } = await query;
  
      console.log("Data fetched:", data); // Tambahkan log ini
  
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
  
  // Filter berdasarkan input bulan, kelas, jurusan, no kelas
  const filteredVisitors = computed(() => {
    return visitors.value.filter((visitor) => {
      const matchTanggal = !tgl_awal.value || visitor.tanggal.startsWith(tgl_awal.value);
      const matchTingkat = !tingkat.value || visitor.siswa?.tingkat === tingkat.value;
      const matchJurusan = !jurusan.value || visitor.jurusan?.nama === jurusan.value;
      const matchKelas = !kelas.value || visitor.siswa?.kelas === kelas.value;
  
      return matchTanggal && matchTingkat && matchJurusan && matchKelas;
    });
  });
  
  // Kelompokkan pengunjung setelah filter diterapkan
  const groupedVisitors = computed(() => {
    const result = {};
    filteredVisitors.value.forEach((visitor) => {
      const nama = visitor.siswa?.nama || "Tidak ada data";
      const year = tgl_awal.value.split("-")[0];
      const month = parseInt(tgl_awal.value.split("-")[1]);
  
      if (!result[nama]) {
        result[nama] = {
          nama,
          sakit: 0,
          izin: 0,
          alfa: 0,
          total: 0,
          totalHari: getDaysInMonth(year, month), // Menghitung total hari dalam bulan
          keterangan: visitor.siswa?.tingkat + ' ' + visitor.jurusan?.nama + ' ' + visitor.siswa?.kelas
        };
      }
  
      if (visitor.keterangan?.nama === 'Sakit') {
        result[nama].sakit += 1;
      } else if (visitor.keterangan?.nama === 'Izin') {
        result[nama].izin += 1;
      } else if (visitor.keterangan?.nama === 'Alpa') {
        result[nama].alfa += 1;
      }
  
      result[nama].total = result[nama].sakit + result[nama].izin + result[nama].alfa;
    });
  
    // Menghitung total hari yang dikurangi dengan jumlah sakit, izin, dan alfa
    Object.values(result).forEach((visitor) => {
      visitor.totalHari -= visitor.total;
    });
  
    return Object.values(result).sort((a, b) => a.nama.localeCompare(b.nama));
  });
  
  const printPage = () => {
    window.print();
  };
  
  const downloadPDF = () => {
    const content = document.getElementById('presensiTable'); // Ambil tabel presensi
  
    if (!content) {
      console.error("Element with id 'presensiTable' not found.");
      return;
    }
  
    html2canvas(content).then((canvas) => {
      const pdf = new jsPDF();
      const imgData = canvas.toDataURL('image/png');
      const imgWidth = 190;
      const imgHeight = (canvas.height * imgWidth) / canvas.width;
      let position = 50; // Mulai setelah judul dan navbar
  
      // Tambah Navbar Judul
      pdf.setFontSize(20);
      pdf.text("Presensi Harian", 105, 20, null, null, "center");
  
      // Tambah Tanggal dan Kelas di bawahnya
      const chosenDate = tgl_awal.value || today;
      const chosenKelas = `${tingkat.value} ${jurusan.value} ${kelas.value}` || 'Semua Kelas';
  
      pdf.setFontSize(12);
      pdf.text(`Tanggal: ${chosenDate}`, 105, 30, null, null, "center");
      pdf.text(`Kelas: ${chosenKelas}`, 105, 40, null, null, "center");
  
      // Tambahkan tabel presensi
      pdf.addImage(imgData, 'PNG', 10, position, imgWidth, imgHeight);
  
      let heightLeft = imgHeight;
      heightLeft -= pdf.internal.pageSize.height;
  
      while (heightLeft >= 0) {
        position = heightLeft - imgHeight;
        pdf.addPage();
        pdf.addImage(imgData, 'PNG', 10, position, imgWidth, imgHeight);
        heightLeft -= pdf.internal.pageSize.height;
      }
  
      // Simpan file PDF dengan nama sesuai tanggal
      pdf.save(`presensi_bulanan_${chosenDate}_${chosenKelas}.pdf`);
    });
  };
  
  const getJurusanOptions = async () => {
    try {
      const { data, error } = await supabase.from('jurusan').select('id, nama');
  
      if (error) {
        console.error('Error fetching jurusan options:', error);
      } else {
        jurusanOptions.value = data || [];
      }
    } catch (error) {
      console.error('Unexpected error:', error);
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
  
  /* Untuk memastikan tabel bisa discroll jika tidak cukup ruang */
  .table-container {
    overflow-x: auto;
    margin-bottom: 1.5rem;
  }
  
  table {
    width: 100%;
    border-collapse: collapse;
  }
  
  th, td {
    text-align: center;
    padding: 8px;
    white-space: nowrap; /* Hindari wrap teks */
  }
  
  thead {
    background-color: #333; /* Warna background header tabel */
    color: #fff; /* Warna teks header */
  }
  
  tbody tr:nth-child(even) {
    background-color: #2c2c2c; /* Warna latar untuk baris genap */
  }
  
  tbody tr:nth-child(odd) {
    background-color: #1a1a1a; /* Warna latar untuk baris ganjil */
  }
  
  /* Responsif untuk perangkat mobile */
  @media (max-width: 768px) {
    /* Mengecilkan font di layar kecil */
    th, td {
      font-size: 12px; /* Ukuran font lebih kecil */
      padding: 6px; /* Padding lebih kecil */
    }
  
    /* Opsional: mengecilkan tabel lebih jauh untuk layar yang sangat kecil */
    table {
      font-size: 0.9rem;
    }
    
    /* Jika tabel terlalu lebar, tambahkan overflow scroll */
    .table-container {
      overflow-x: scroll;
    }
  }
  
  /* Responsif untuk layar yang sangat kecil (max-width: 480px) */
  @media (max-width: 480px) {
    th, td {
      font-size: 10px; /* Ukuran font lebih kecil */
      padding: 4px; /* Padding lebih kecil */
    }
  
    table {
      font-size: 0.8rem; /* Ukuran tabel lebih kecil */
    }
  }
  
  
  @media (max-width :768px) {
      input::placeholder, option::placeholder{
        font-size: 20px;
      }
  }
  
  @media (max-width : 480px){
    input::placeholder, option::placeholder{
      font-size: 16px;
      justify-content: center
    }
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
      padding: 10px; /* Tambahkan padding untuk konten */
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
      padding: 6px;
      text-align: center;
    }
  
    thead th {
      border-bottom: 2px solid black;
      background-color: #f2f2f2;
    }
  
    h1, p {
      color: black;
      text-align: center;
    }
  }
  
  
  </style>
  
