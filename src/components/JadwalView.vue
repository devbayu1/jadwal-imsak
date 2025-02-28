<template>
  <div class="container">
    <div class="header">
      <h1>Jadwal Imsakiyah</h1>
      <DarkModeToggle />
    </div>

    <!-- Pesan Error -->
    <div v-if="error" class="error-message">
      {{ error }}
    </div>

    <!-- Form Seleksi -->
    <div class="form-grid">
      <!-- Pilih Provinsi -->
      <div class="form-group">
        <label for="provinsi">Pilih Provinsi</label>
        <select id="provinsi" v-model="selectedProvinsi" :disabled="loading || provinsi.length === 0" @change="handleProvinsiChange">
          <option value="">-- Pilih Provinsi --</option>
          <option v-for="p in provinsi" :key="p" :value="p">{{ p }}</option>
        </select>
      </div>

      <!-- Pilih Kota -->
      <div class="form-grou">
        <label for="kota">Pilih Kota/Kabupaten</label>
        <select id="kota" v-model="selectedKota" :disabled="loading || kota.length === 0 || !selectedProvinsi" @change="handleKotaChange">
          <option value="">-- Pilih Kota --</option>
          <option v-for="k in kota" :key="k" :value="k">{{ k }}</option>
        </select>
      </div>
    </div>

    <!-- Loading indicator -->
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
    </div>

    <!-- Jadwal Table -->
    <transition name="fade" mode="out-in">
      <div v-if="jadwal.length > 0" class="table-container">
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

      <!-- Pesan Informasi -->
      <div v-else-if="selectedKota && !loading" class="info-message">Tidak ada jadwal tersedia untuk kota yang dipilih</div>
    </transition>

    <div v-if="!selectedProvinsi && !loading" class="info-message">Silakan pilih provinsi untuk melihat daftar kota</div>

    <div v-else-if="selectedProvinsi && !selectedKota && !loading && kota.length > 0" class="info-message">
      Silakan pilih kota untuk melihat jadwal
    </div>
    <div class="info-message">
        API menggunakan <a href="https://equran.id/apidev/imsakiyah" target="_blank">equran.id</a>,
        Sumber dari data pada API ini adalah Bimas Islam Kemenag RI
    </div>
  </div>
</template>

<script>
import axios from "axios";
import DarkModeToggle from "./DarkModeToggle.vue";

export default {
  name: "JadwalView",
  components: {
    DarkModeToggle,
  },
  data() {
    return {
      provinsi: [],
      kota: [],
      jadwal: [],
      selectedProvinsi: "",
      selectedKota: "",
      loading: false,
      error: null,
    };
  },
  mounted() {
    this.fetchProvinsi();
  },
  methods: {
    async fetchProvinsi() {
      this.loading = true;
      try {
        const response = await axios.get("https://equran.id/api/v2/imsakiyah/provinsi");
        this.provinsi = response.data.data;
        this.error = null;
      } catch (err) {
        this.error = "Gagal mengambil data provinsi";
        console.error("Error fetching provinsi:", err);
      } finally {
        this.loading = false;
      }
    },
    async fetchKota() {
      if (!this.selectedProvinsi) {
        this.kota = [];
        return;
      }

      this.loading = true;
      try {
        const response = await axios.post(`https://equran.id/api/v2/imsakiyah/kabkota`, {
          provinsi: this.selectedProvinsi,
        });
        this.kota = response.data.data;
        this.selectedKota = "";
        this.jadwal = [];
        this.error = null;
      } catch (err) {
        this.error = "Gagal mengambil data kota";
        console.error("Error fetching kota:", err);
      } finally {
        this.loading = false;
      }
    },
    async fetchJadwal() {
      if (!this.selectedKota) {
        this.jadwal = [];
        return;
      }

      this.loading = true;
      try {
        const response = await axios.post(`https://equran.id/api/v2/imsakiyah`, {
          kabkota: this.selectedKota,
          provinsi: this.selectedProvinsi,
        });

        this.jadwal = response.data.data[0].imsakiyah;
        this.error = null;
      } catch (err) {
        this.error = "Gagal mengambil data jadwal";
        console.error("Error fetching jadwal:", err);
      } finally {
        this.loading = false;
      }
    },
    handleProvinsiChange() {
      this.fetchKota();
    },
    handleKotaChange() {
      this.fetchJadwal();
    },
  },
};
</script>
