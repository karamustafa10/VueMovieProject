<template>
  <!-- Site adını ve arama kutusunu içeren başlık kısmı -->
  <div class="header">
    <!-- Dinamik olarak site adını göster -->
    <div class="siteName"> {{ sitename }} </div>
    <div>
      <!-- Film aramak için giriş kutusu -->
      <input class="inputBox" type="text" placeholder="Type Here...">
      <!-- Arama fonksiyonunu tetikleyen buton -->
      <button class="searchButton" @click="searchMovies">Search!</button>
    </div>
  </div>

  <!-- Alınan filmleri listeleyen film kısmı -->
  <div class="movies">
    <!-- movies dizisi üzerinden geçerek her film için bir kart oluştur -->
    <div class="movieCard" v-for="movie in movies" :key="movie.id">
      <!-- Film posteri varsa göster -->
      <img
        v-if="movie.poster_path"
        :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
        :alt="movie.title" 
        class="moviePoster"
      />
      <!-- Film başlığını göster -->
      <p class="movieName">{{ movie.title }}</p>
      <!-- Formatlanmış çıkış tarihini göster -->
      <p class="movieInfo"><strong>Release Date:</strong> {{ formatDate(movie.release_date) }}</p>
      <!-- Film puanını göster -->
      <p class="movieInfo"><strong>Rating:</strong> {{ movie.vote_average }} / 10</p>
      <!-- Film özeti göster -->
      <p class="movieInfo"><strong>Overview:</strong> {{ movie.overview }}</p>
      <!-- Filmi izlemek için ve IMDb sayfasına gitmek için butonlar -->
      <button class="movieButtonWatch">Watch Now</button>
      <button class="movieButtonIMDb">IMDb</button>
    </div>
  </div>

  <!-- Filmler sayfasında gezinmek için sayfalama kontrolleri -->
  <div style="text-align: center;">
    <!-- Bir önceki sayfaya gitme butonu, birinci sayfadaysa devre dışı -->
    <button class="pageButtonBack" @click="prevPage" :disabled="currentPage === 1">Back</button>
    <!-- Mevcut sayfa ve toplam sayfa sayısını göster -->
    <span class="pageInfo">Page {{ currentPage }} / {{ totalPages }}</span>
    <!-- Sonraki sayfaya gitme butonu, son sayfadaysa devre dışı -->
    <button class="pageButtonNext" @click="nextPage" :disabled="currentPage >= totalPages">Next</button>
  </div>
</template>

<script>
import { ref } from 'vue'; // Vue'dan reaktivite için ref'i içe aktar
import axios from 'axios'; // API istekleri için axios'u içe aktar

export default {
  setup() {
    // Reaktif değişkenler
    const sitename = ref('VueMovieProject'); // Site adı
    const movies = ref([]); // Film verilerini tutan dizi
    const currentPage = ref(1); // Geçerli sayfa numarası
    const totalPages = ref(500); // Toplam sayfa sayısı, sabit olarak 500

    // Çıkış tarihini formatlamak için fonksiyon
    const formatDate = (dateString) => {
      const options = { day: 'numeric', month: 'long', year: 'numeric' }; // Format seçenekleri
      return new Date(dateString).toLocaleDateString('en-US', options); // Yerelleştirilmiş tarih dizesine çevir
    };

    // API'den filmleri almak için fonksiyon
    const fetchMovies = async (page_info) => {
      try {
        const response = await axios.get('https://api.themoviedb.org/3/movie/popular', {
          params: {
            api_key: '9d35ac0ee6cbf1e6e554c6f54a4d6df5', // Kimlik doğrulama için API anahtarı
            language: 'en-US', // Dil parametresi
            page: page_info, // Alınacak sayfa numarası
            sort_by: 'vote_average.desc', // Filmleri en yüksek oylama puanına göre sıralar
          },
        });
        movies.value = response.data.results; // Alınan verilerle movies dizisini güncelle
      } catch (error) {
        console.error('Error fetching movies:', error); // Alım başarısız olursa hatayı logla
      }
    };

    // Bir sonraki film sayfasına gitme fonksiyonu
    const nextPage = () => {
      if (currentPage.value < totalPages.value) { // Son sayfada değilse kontrol et
        currentPage.value++; // Geçerli sayfayı artır
        fetchMovies(currentPage.value).then(() => {
          window.scrollTo({ top: 0, behavior: 'smooth' }); // Yavaşça üst kısma kaydır
        });
      }
    };

    // Bir önceki film sayfasına gitme fonksiyonu
    const prevPage = () => {
      if (currentPage.value > 1) { // Birinci sayfada değilse kontrol et
        currentPage.value--; // Geçerli sayfayı azalt
        fetchMovies(currentPage.value).then(() => {
          window.scrollTo({ top: 0, behavior: 'smooth' }); // Yavaşça üst kısma kaydır
        });
      }
    };

    // Kullanıcı girdisine göre filmleri aramak için fonksiyon
    const searchMovies = async () => {
      const query = document.querySelector('.inputBox').value; // Giriş değerini al

      if (!query) {
        fetchMovies(currentPage.value); // Eğer sorgu yoksa popüler filmleri al
        return;
      }

      try {
        const currentPage = ref(1); // Geçerli sayfa numarası

        const response = await axios.get('https://api.themoviedb.org/3/search/movie', {
          params: {
            api_key: '9d35ac0ee6cbf1e6e554c6f54a4d6df5', // Kimlik doğrulama için API anahtarı
            language: 'en-US', // Dil parametresi
            query: query, // Kullanıcının arama sorgusu
            page: currentPage.value, // Mevcut sayfa numarası
            sort_by: 'vote_average.desc', // Filmleri oylama puanına göre sıralar
          },
        });
        
        movies.value = response.data.results; // Arama sonuçları ile movies dizisini güncelle
        window.scrollTo({ top: 0, behavior: 'smooth' }); // Yavaşça üst kısma kaydır
      } catch (error) {
        console.error('Error searching movies:', error); // Arama başarısız olursa hatayı logla
      }
    };

    // Bileşen oluşturulurken başlangıçta filmleri al
    fetchMovies(currentPage.value);

    // Şablona reaktif değişkenleri ve fonksiyonları döndür
    return {
      sitename,
      movies,
      currentPage,
      totalPages,
      nextPage,
      prevPage,
      formatDate,
      searchMovies,
    };
  },
};
</script>

<style src="./style.css"></style>