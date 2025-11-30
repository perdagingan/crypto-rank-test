<template>
  <div class="crypto-container">
    <h1>Daftar Cryptocurrency</h1>

    <button @click="tampilkanDataCrypto" class="btn-load" :disabled="loading">
      {{ loading ? "Memuat Data..." : "Tampilkan Data Cryptocurrency" }}
    </button>

    <div v-if="tampil" class="sort-selector">
      <label for="sortBy">Urutkan berdasarkan:</label>
      <select id="sortBy" v-model="sortBy" @change="sortData">
        <option value="market_cap">Market Cap</option>
        <option value="price">Harga (Price)</option>
      </select>
    </div>

    <div v-if="error" class="error-message">
      {{ error }}
    </div>

    <div v-if="tampil" class="table-container">
      <div class="info-box">
        <p>Kurs: 1 USD = Rp {{ kursUSDtoIDR.toLocaleString("id-ID") }}</p>
      </div>

      <table class="crypto-table">
        <thead>
          <tr>
            <th>Rank</th>
            <th>Logo</th>
            <th>Name</th>
            <th>Symbol</th>
            <th>Market Cap (USD)</th>
            <th>Price (USD)</th>
            <th>Price (IDR)</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(crypto, index) in sortedCryptoList" :key="crypto.id">
            <td class="rank-cell" :class="getRankClass(crypto)">
              <span class="rank-value">{{ index + 1 }}</span>
              <span v-if="crypto.rankChange === 'up'" class="rank-indicator up">
                ↑
              </span>
              <span
                v-if="crypto.rankChange === 'down'"
                class="rank-indicator down"
              >
                ↓
              </span>
            </td>
            <td class="logo-cell">
              <img
                :src="getCryptoLogo(crypto.symbol)"
                :alt="crypto.name"
                class="crypto-logo"
                loading="lazy"
              />
            </td>
            <td class="name-cell">{{ crypto.name }}</td>
            <td class="symbol-cell">{{ crypto.symbol }}</td>
            <td class="market-cap-cell">
              ${{ formatMarketCap(crypto.market_cap_usd) }}
            </td>
            <td class="price-usd" :class="getPriceClass(crypto)">
              ${{ formatNumber(crypto.price_usd) }}
              <span v-if="crypto.priceChange === 'up'" class="price-indicator">
                ▲
              </span>
              <span
                v-if="crypto.priceChange === 'down'"
                class="price-indicator"
              >
                ▼
              </span>
            </td>
            <td class="price-idr" :class="getPriceClass(crypto)">
              Rp {{ formatIDR(crypto.price_usd) }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Footer -->
    <footer class="footer">
      <div class="footer-content">
        <div class="footer-section">
          <h3>📊 Crypto Tracker</h3>
          <p>Real-time cryptocurrency price tracking application</p>
        </div>
        
        <div class="footer-section">
          <h4>Data Source</h4>
          <p>
            <a href="https://www.coinlore.com/" target="_blank" rel="noopener noreferrer">
              CoinLore API
            </a>
          </p>
          <p class="api-note">Free & Public Cryptocurrency API</p>
        </div>
        
        <div class="footer-section">
          <h4>Informasi</h4>
          <p>Kurs: 1 USD = Rp {{ kursUSDtoIDR.toLocaleString("id-ID") }}</p>
          <p class="update-note">Data diperbarui secara real-time</p>
        </div>
      </div>
      
      <div class="footer-bottom">
        <p>&copy; {{ currentYear }} Crypto Tracker. Made with ❤️ for MSIM4401</p>
        <p class="disclaimer">
          ⚠️ Disclaimer: Data hanya untuk referensi. Lakukan riset sendiri sebelum berinvestasi.
        </p>
      </div>
    </footer>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, onMounted, onUnmounted } from "vue";
import axios from "axios";

interface Cryptocurrency {
  id: string;
  rank: string;
  name: string;
  symbol: string;
  price_usd: string;
  market_cap_usd: string;
  priceChange?: "up" | "down" | "neutral";
  rankChange?: "up" | "down" | "neutral";
}

export default defineComponent({
  name: "CryptoList",
  setup() {
    const tampil = ref(false);
    const loading = ref(false);
    const error = ref("");
    const dataCryptoList = ref<Cryptocurrency[]>([]);
    const kursUSDtoIDR = ref(15700);
    const currentYear = new Date().getFullYear();
    const sortBy = ref<"market_cap" | "price">("market_cap");
    let refreshInterval: number | null = null;
    const previousData = ref<{ [id: string]: { price: number; rank: number } }>(
      {}
    );

    // Computed property to sort the crypto list based on selected criteria
    const sortedCryptoList = computed(() => {
      const data = [...dataCryptoList.value];
      if (sortBy.value === "market_cap") {
        return data.sort((a, b) => {
          const marketCapA = parseFloat(a.market_cap_usd) || 0;
          const marketCapB = parseFloat(b.market_cap_usd) || 0;
          return marketCapB - marketCapA; // Descending order (highest first)
        });
      } else if (sortBy.value === "price") {
        return data.sort((a, b) => {
          const priceA = parseFloat(a.price_usd) || 0;
          const priceB = parseFloat(b.price_usd) || 0;
          return priceB - priceA; // Descending order (highest first)
        });
      }
      return data;
    });

    // Function to handle sort change
    const sortData = () => {
      // The computed property automatically handles sorting
      // This function is called on change event for any additional logic if needed
    };

    const tampilkanDataCrypto = async () => {
      loading.value = true;
      error.value = "";

      try {
        const response = await axios.get(
          "https://api.coinlore.net/api/tickers/"
        );

        if (response.data && response.data.data) {
          const newData = response.data.data;

          // Detect changes
          newData.forEach((crypto: Cryptocurrency) => {
            const prev = previousData.value[crypto.id];
            if (prev) {
              const newPrice = parseFloat(crypto.price_usd);
              const oldPrice = prev.price;
              const newRank = parseInt(crypto.rank);
              const oldRank = prev.rank;

              // Price change
              if (newPrice > oldPrice) {
                crypto.priceChange = "up";
              } else if (newPrice < oldPrice) {
                crypto.priceChange = "down";
              } else {
                crypto.priceChange = "neutral";
              }

              // Rank change
              if (newRank < oldRank) {
                crypto.rankChange = "up";
              } else if (newRank > oldRank) {
                crypto.rankChange = "down";
              } else {
                crypto.rankChange = "neutral";
              }

              // Remove animation class after 2 seconds
              setTimeout(() => {
                crypto.priceChange = "neutral";
                crypto.rankChange = "neutral";
              }, 2000);
            }

            // Store current data for next comparison
            previousData.value[crypto.id] = {
              price: parseFloat(crypto.price_usd),
              rank: parseInt(crypto.rank),
            };
          });

          dataCryptoList.value = newData;
          tampil.value = true;
        } else {
          error.value = "Format data tidak sesuai";
        }
      } catch (err) {
        error.value = "Gagal mengambil data. Periksa koneksi internet Anda.";
        console.error("Error fetching data:", err);
      } finally {
        loading.value = false;
      }
    };

    // Function untuk mendapatkan URL logo cryptocurrency (dengan caching)
    const logoCache: { [key: string]: string } = {};
    const getCryptoLogo = (symbol: string): string => {
      if (!logoCache[symbol]) {
        const lowerSymbol = symbol.toLowerCase();
        logoCache[symbol] = `https://assets.coincap.io/assets/icons/${lowerSymbol}@2x.png`;
      }
      return logoCache[symbol];
    };

    // Format angka dengan 2 desimal
    const formatNumber = (value: string): string => {
      const num = parseFloat(value);
      if (num >= 1) {
        return num.toLocaleString("en-US", {
          minimumFractionDigits: 2,
          maximumFractionDigits: 2,
        });
      } else {
        return num.toLocaleString("en-US", {
          minimumFractionDigits: 2,
          maximumFractionDigits: 8,
        });
      }
    };

    // Format ke Rupiah
    const formatIDR = (priceUSD: string): string => {
      const usd = parseFloat(priceUSD);
      const idr = usd * kursUSDtoIDR.value;
      return idr.toLocaleString("id-ID", {
        minimumFractionDigits: 0,
        maximumFractionDigits: 0,
      });
    };

    // Format market cap with abbreviated format (B for billions, M for millions)
    const formatMarketCap = (value: string): string => {
      const num = parseFloat(value);
      if (num >= 1e12) {
        return (num / 1e12).toFixed(2) + "T";
      } else if (num >= 1e9) {
        return (num / 1e9).toFixed(2) + "B";
      } else if (num >= 1e6) {
        return (num / 1e6).toFixed(2) + "M";
      } else if (num >= 1e3) {
        return (num / 1e3).toFixed(2) + "K";
      }
      return num.toLocaleString("en-US", {
        minimumFractionDigits: 0,
        maximumFractionDigits: 0,
      });
    };

    // Get CSS class for price animation
    const getPriceClass = (crypto: Cryptocurrency): string => {
      if (crypto.priceChange === "up") return "price-up";
      if (crypto.priceChange === "down") return "price-down";
      return "";
    };

    // Get CSS class for rank animation
    const getRankClass = (crypto: Cryptocurrency): string => {
      if (crypto.rankChange === "up") return "rank-up";
      if (crypto.rankChange === "down") return "rank-down";
      return "";
    };

    // Auto-refresh setiap 30 detik
    onMounted(() => {
      refreshInterval = window.setInterval(() => {
        if (tampil.value && !loading.value) {
          tampilkanDataCrypto();
        }
      }, 30000); // 30 detik
    });

    // Cleanup interval saat component unmount
    onUnmounted(() => {
      if (refreshInterval) {
        clearInterval(refreshInterval);
      }
    });

    return {
      tampil,
      loading,
      error,
      dataCryptoList,
      sortedCryptoList,
      sortBy,
      kursUSDtoIDR,
      currentYear,
      tampilkanDataCrypto,
      sortData,
      getCryptoLogo,
      formatNumber,
      formatIDR,
      formatMarketCap,
      getPriceClass,
      getRankClass,
    };
  },
});
</script>

<style scoped>
.crypto-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
  color: #ffffff;
  margin-bottom: 30px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  font-size: 2.5em;
}

.btn-load {
  display: block;
  margin: 0 auto 20px;
  padding: 15px 30px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
}

.btn-load:hover {
  background-color: #45a049;
  transform: translateY(-2px);
  box-shadow: 0 6px 8px rgba(0, 0, 0, 0.3);
}

.btn-load:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
  transform: none;
}

.sort-selector {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
}

.sort-selector label {
  font-weight: bold;
  color: #2c3e50;
}

.sort-selector select {
  padding: 10px 15px;
  font-size: 14px;
  border: 2px solid #4caf50;
  border-radius: 6px;
  background-color: white;
  color: #2c3e50;
  cursor: pointer;
  transition: all 0.3s;
}

.sort-selector select:hover {
  border-color: #45a049;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.sort-selector select:focus {
  outline: none;
  border-color: #2196f3;
  box-shadow: 0 0 0 3px rgba(33, 150, 243, 0.2);
}

.error-message {
  background-color: #ffebee;
  color: #c62828;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
  text-align: center;
  font-weight: bold;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.info-box {
  background-color: #fff3cd;
  border: 1px solid #ffc107;
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 15px;
  text-align: center;
  font-weight: bold;
  color: #856404;
}

.table-container {
  overflow-x: auto;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.crypto-table {
  width: 100%;
  border-collapse: collapse;
  background-color: white;
}

.crypto-table thead {
  background: linear-gradient(135deg, #2c3e50 0%, #34495e 100%);
  color: white;
}

.crypto-table th {
  padding: 15px 12px;
  text-align: left;
  font-weight: bold;
  text-transform: uppercase;
  font-size: 13px;
  letter-spacing: 0.5px;
}

.crypto-table th:first-child {
  text-align: center;
}

.crypto-table th:nth-child(2) {
  text-align: center;
}

.crypto-table th:nth-child(5),
.crypto-table th:nth-child(6),
.crypto-table th:nth-child(7) {
  text-align: right;
}

.crypto-table td {
  padding: 15px 12px;
  border-bottom: 1px solid #e0e0e0;
  font-size: 14px;
}

.crypto-table tbody tr {
  transition: background-color 0.2s;
}

.crypto-table tbody tr:hover {
  background-color: #f5f5f5;
}

.crypto-table tbody tr:last-child td {
  border-bottom: none;
}

.rank-cell {
  font-weight: bold;
  color: #4caf50;
  text-align: center;
  width: 60px;
  position: relative;
  transition: all 0.5s ease;
}

.rank-value {
  display: inline-block;
  transition: transform 0.5s ease;
}

.rank-indicator {
  position: absolute;
  right: 5px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 14px;
  animation: slideIn 0.5s ease;
}

.rank-indicator.up {
  color: #4caf50;
}

.rank-indicator.down {
  color: #f44336;
}

.rank-up {
  background-color: rgba(76, 175, 80, 0.1) !important;
  animation: pulseGreen 0.5s ease;
}

.rank-down {
  background-color: rgba(244, 67, 54, 0.1) !important;
  animation: pulseRed 0.5s ease;
}

.logo-cell {
  text-align: center;
  width: 70px;
}

.crypto-logo {
  width: 40px;
  height: 40px;
  object-fit: contain;
  border-radius: 50%;
  background-color: #f9f9f9;
  padding: 3px;
  /* Prevent reloading */
  will-change: auto;
}

.name-cell {
  font-weight: 500;
  color: #2c3e50;
  min-width: 150px;
  text-align: left;
}

.symbol-cell {
  font-weight: 600;
  color: #7f8c8d;
  text-transform: uppercase;
  min-width: 100px;
  text-align: left;
}

.market-cap-cell {
  font-weight: bold;
  color: #9c27b0;
  text-align: right;
  min-width: 140px;
}

.price-usd {
  font-weight: bold;
  color: #2196f3;
  text-align: right;
  min-width: 140px;
  padding-right: 20px !important;
  position: relative;
  transition: all 0.5s ease;
}

.price-idr {
  font-weight: bold;
  color: #ff9800;
  text-align: right;
  min-width: 180px;
  padding-right: 20px !important;
  transition: all 0.5s ease;
}

.price-indicator {
  position: absolute;
  right: 2px;
  font-size: 12px;
  animation: bounce 0.5s ease;
}

.price-up {
  background-color: rgba(76, 175, 80, 0.15) !important;
  animation: pulseGreen 0.5s ease;
}

.price-up .price-indicator {
  color: #4caf50;
}

.price-down {
  background-color: rgba(244, 67, 54, 0.15) !important;
  animation: pulseRed 0.5s ease;
}

.price-down .price-indicator {
  color: #f44336;
}

/* Animations */
@keyframes pulseGreen {
  0% {
    background-color: rgba(76, 175, 80, 0.3);
  }
  50% {
    background-color: rgba(76, 175, 80, 0.5);
  }
  100% {
    background-color: rgba(76, 175, 80, 0.15);
  }
}

@keyframes pulseRed {
  0% {
    background-color: rgba(244, 67, 54, 0.3);
  }
  50% {
    background-color: rgba(244, 67, 54, 0.5);
  }
  100% {
    background-color: rgba(244, 67, 54, 0.15);
  }
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-50%) translateX(10px);
  }
  to {
    opacity: 1;
    transform: translateY(-50%) translateX(0);
  }
}

@keyframes bounce {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-5px);
  }
}

/* Responsive */
@media (max-width: 768px) {
  .crypto-table {
    font-size: 12px;
  }

  .crypto-table th,
  .crypto-table td {
    padding: 10px 8px;
  }

  .crypto-logo {
    width: 32px;
    height: 32px;
  }

  h1 {
    font-size: 1.8em;
  }

  .footer-content {
    flex-direction: column;
    gap: 20px;
  }

  .footer-section {
    text-align: center;
  }
}

/* Footer Styles */
.footer {
  margin-top: 60px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 15px;
  padding: 40px 30px 20px;
  box-shadow: 0 -4px 20px rgba(0, 0, 0, 0.2);
}

.footer-content {
  display: flex;
  justify-content: space-between;
  gap: 30px;
  margin-bottom: 30px;
  flex-wrap: wrap;
}

.footer-section {
  flex: 1;
  min-width: 200px;
}

.footer-section h3 {
  color: #2c3e50;
  margin-bottom: 10px;
  font-size: 1.3em;
}

.footer-section h4 {
  color: #34495e;
  margin-bottom: 10px;
  font-size: 1.1em;
  font-weight: 600;
}

.footer-section p {
  color: #555;
  line-height: 1.6;
  margin: 5px 0;
}

.footer-section a {
  color: #2196f3;
  text-decoration: none;
  font-weight: 600;
  transition: color 0.3s;
}

.footer-section a:hover {
  color: #1976d2;
  text-decoration: underline;
}

.api-note {
  font-size: 0.85em;
  color: #888;
  font-style: italic;
}

.update-note {
  font-size: 0.85em;
  color: #4caf50;
  font-weight: 500;
}

.footer-bottom {
  border-top: 2px solid #e0e0e0;
  padding-top: 20px;
  text-align: center;
}

.footer-bottom p {
  color: #666;
  margin: 8px 0;
  font-size: 0.95em;
}

.disclaimer {
  font-size: 0.85em !important;
  color: #ff9800 !important;
  font-weight: 500;
  margin-top: 10px !important;
}
</style>
