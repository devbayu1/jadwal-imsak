<template>
  <div class="container">
    <div class="header">
      <div class="header-left">
        <h1>Jadwal Imsakiyah</h1>

        <!-- ✅ Tombol kembali ke domain utama -->
        <a href="/" class="btn-back cursor-pointer">← Kembali ke Beranda</a>
      </div>

      <DarkModeToggle />
    </div>

    <!-- Error -->
    <div v-if="error" class="error-message">
      {{ error }}
    </div>

    <!-- FORM -->
    <div class="form-grid">
      <!-- KOTA SEARCHABLE -->
      <div class="form-group">
        <label>Cari Kota / Kabupaten</label>

        <Select
          v-model="selectedKota"
          :options="kotaList"
          :get-option-label="(option) => option.lokasi"
          :get-option-value="(option) => option.id"
          placeholder="Ketik nama kota..."
          searchable
          clearable
          @update:modelValue="fetchJadwal"
        />
      </div>

      <!-- BULAN -->
      <div class="form-group">
        <label>Bulan</label>
        <VueDatePicker v-model="selectedMonth" month-picker @update:modelValue="fetchJadwal"></VueDatePicker>
      </div>
    </div>

    <!-- LOADING -->
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
    </div>

    <!-- TABLE -->
    <div v-if="jadwal.length" class="table-container">
      <table>
        <thead>
          <tr>
            <th>Tanggal</th>
            <th>Imsak</th>
            <th>Subuh</th>
            <th>Terbit</th>
            <th>Dhuha</th>
            <th>Dzuhur</th>
            <th>Ashar</th>
            <th>Maghrib</th>
            <th>Isya</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in jadwal" :key="index">
            <td>{{ item.tanggal }}</td>
            <td>{{ item.imsak }}</td>
            <td>{{ item.subuh }}</td>
            <td>{{ item.terbit }}</td>
            <td>{{ item.dhuha }}</td>
            <td>{{ item.dzuhur }}</td>
            <td>{{ item.ashar }}</td>
            <td>{{ item.maghrib }}</td>
            <td>{{ item.isya }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- INFO -->
    <div v-if="!jadwal.length && !loading" class="info-message">Pilih kota dan bulan untuk melihat jadwal</div>

    <div class="info-message">
      API menggunakan
      <a href="https://api.myquran.com" target="_blank">myquran.com</a>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import DarkModeToggle from "./DarkModeToggle.vue";
import Select from "vue3-select-component";
import "vue3-select-component/styles";
import { VueDatePicker } from "@vuepic/vue-datepicker";
import "@vuepic/vue-datepicker/dist/main.css";

export default {
  name: "JadwalView",

  components: {
    DarkModeToggle,
    Select,
    VueDatePicker,
  },

  data() {
    return {
      kotaList: [],
      selectedKota: "",
      selectedMonth: null,
      jadwal: [],
      loading: false,
      error: null,
    };
  },

  mounted() {
    this.setDefaultMonth();
    this.fetchKota();
  },

  methods: {
    // ✅ Default bulan sekarang
    setDefaultMonth() {
      const now = new Date();
      // const tahun = now.getFullYear();
      // const bulan = String(now.getMonth() + 1).padStart(2, "0");
      this.selectedMonth = {
        month: new Date().getMonth(),
        year: new Date().getFullYear(),
      };
    },

    // ✅ Ambil semua kota
    async fetchKota() {
      try {
        const res = await axios.get("https://api.myquran.com/v3/sholat/kabkota/semua");

        this.kotaList = res.data.data;

        // ✅ Default Jakarta
        const selectedC = this.kotaList.find((k) => k.lokasi.toLowerCase().includes("pacitan"));

        if (selectedC) {
          this.selectedKota = selectedC.id;
          this.fetchJadwal();
        }
      } catch (err) {
        this.error = "Gagal mengambil daftar kota";
        console.error(err);
      }
    },

    // ✅ Ambil jadwal sholat
    async fetchJadwal() {
      if (!this.selectedKota || !this.selectedMonth) return;
      console.log(this.selectedMonth);
      console.log(new Date());
      this.loading = true;
      this.jadwal = [];
      const year = this.selectedMonth.year;
      const month = String(this.selectedMonth.month + 1).padStart(2, "0"); // ← WAJIB +1
      const formattedMonth = `${year}-${month}`;

      try {
        const res = await axios.get(`https://api.myquran.com/v3/sholat/jadwal/${this.selectedKota}/${formattedMonth}`);

        const dataObj = res.data.data.jadwal;

        // ✅ Object → Array
        this.jadwal = Object.values(dataObj);
        this.error = null;
      } catch (err) {
        this.error = "Gagal mengambil jadwal sholat";
        console.error(err);
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style></style>
