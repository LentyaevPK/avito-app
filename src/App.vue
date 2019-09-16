<template>
  <div id="app">
    <Header />
    <div class="price-selection">
      <span>Введите цену:</span><br>
      <span>От</span>
      <input class="minprice" />
      <span>руб.</span>
      <span>До</span>
      <input class="maxprice" />
      <span>руб.</span>
      <button @click="getPrice">Применить</button>
    </div>
    <div class="categories">
      <span>Выберите категорию:</span><br>
      <select v-model="categoryFilter">
        <option value="all">Все</option>
        <option value="auto">Авто</option>
        <option value="immovable">Недвижимость</option>
        <option value="laptops">Ноутбуки</option>
        <option value="cameras">Фотоаппараты</option>
      </select>
    </div>
    <div class="sort">
      <span>Сортировать по:</span><br>
      <select v-model="sortFilter">
        <option value="all">Все</option>
        <option value="rating">По рейтингу</option>
        <option value="price">По цене</option>
      </select>
    </div>
    <div class="favourites">
      <span>Показывать избранные</span>
      <input class="showFav" type="checkbox" @change="showFav" />
    </div>
    <Loader v-if="loading" />
    <div class="products" v-else-if="storageProducts.length">
      <div class="product-container" v-for="product in storageProducts" :key="product.id">
        <img :src="product.pictures[0]" class="img-product" />
        <span class="fav" :class="{ active: product.favourite }" @click="addFav(product, $event)">&#10084;</span>
        <span class="img-amount">{{ product.pictures.length }}</span>
        <h3>{{ product.title }}</h3>
        <span class="price" v-if="product.price">{{ product.price | spaceDigits }} ₽</span>
        <span class="price" v-else>Цена не указана</span>
        <span class="seller">Продваец: {{ product.sellerName }}</span>
        <span class="seller-rating">Рейтинг: {{ product.sellerRating }}</span>
      </div>
    </div>
    <h2 class="no-products" v-else>Ничего не найдено</h2>
  </div>
</template>

<script>
import Header from "./components/Header.vue"
import axios from "axios"
import Loader from "./components/Loader.vue"

export default {
  name: "app",
  data() {
    return {
      products: [],
      allProducts: [],
      currentProducts: [],
      sellers: [],
      categoryFilter: "all",
      sortFilter: "all",
      loading: true
    };
  },
  components: {
    Header, Loader
  },
  mounted() {
    axios
      .get("http://avito.dump.academy/products")
      .then(response => {
        this.products = response.data.data;
        this.allProducts = this.products;
        this.currentProducts = this.products;
        this.products.forEach(a => (a.favourite = false));
    });
    axios
      .get("http://avito.dump.academy/sellers")
      .then(response => {
        this.sellers = response.data.data; 
        this.loading = false
      });
  },
  methods: {
    // Метод, позволяющий задать диапазон цен
    getPrice() {
      let favInput = document.querySelector(".showFav");
      let minPrice = document.querySelector(".minprice").value;
      let maxPrice = document.querySelector(".maxprice").value;
      let filtProd = this.allProducts.filter(
          t => t.price >= minPrice && t.price <= maxPrice
        );
      if (isFinite(minPrice) && isFinite(maxPrice) && minPrice != '' && maxPrice != '') {
        if (favInput.checked === true) {
          this.products = this.allProducts.filter(t => t.favourite === true);
          this.products = this.products.filter(
            t => t.price >= minPrice && t.price <= maxPrice
          );
          this.currentProducts = filtProd
        } else {
          this.products = filtProd
          this.currentProducts = this.products;
        }
      } else {
        alert('Необходимо ввести число!')
      }
    },

    // Метод для добавления товара в избранное, и сохранение информации в LocalStorage
    addFav(obj, e) {
      let favIcon = document.querySelectorAll(".fav");
      if (obj.favourite) {
        e.target.className = "fav";
        obj.favourite = false;
        let oldItems = JSON.parse(localStorage.getItem("testObject")) || [];
        oldItems.pop(obj);
        localStorage.setItem("testObject", JSON.stringify(oldItems));
      } else {
        e.target.className = "fav active";
        obj.favourite = true;
        let oldItems = JSON.parse(localStorage.getItem("testObject")) || [];
        oldItems.push(obj);
        localStorage.setItem("testObject", JSON.stringify(oldItems));
      }
    },

    // Метод для отображения избранных товаров
    showFav() {
      let favInput = document.querySelector(".showFav");
      if (favInput.checked === true) {
        this.products = this.products.filter(t => t.favourite === true);
      } else {
        this.products = this.currentProducts;
      }
    },
  },
  computed: {
    // Объединение товаров и продавцов в один массив
    findSellers() {
      for (let i = 0; i < this.products.length; i++) {
        for (let j = 0; j < this.sellers.length; j++) {
          if (this.sellers[j].id === this.products[i].relationships.seller) {
            this.products[i].sellerName = this.sellers[j].name;
            this.products[i].sellerRating = this.sellers[j].rating;
          }
        }
      }
      return this.products;
    },

    // Фильтрация по категории
    filteredProducts() {
      let filtered = this.findSellers;
      switch (this.categoryFilter) {
        case "all":
          return filtered;
        case "auto":
          return filtered.filter(t => t.category === "auto");
        case "immovable":
          return filtered.filter(t => t.category === "immovable");
        case "laptops":
          return filtered.filter(t => t.category === "laptops");
        case "cameras":
          return filtered.filter(t => t.category === "cameras");
      }
      return filtered;
    },

    // Сортировка по рейтингу продавца или по цене
    sortedProducts() {
      let filtered = this.filteredProducts;
      switch (this.sortFilter) {
        case "all":
          return filtered.sort((a,b) => a.id - b.id);
        case "rating":
          return filtered.sort((a,b) => b.sellerRating - a.sellerRating);
        case "price":
          return filtered.sort((a,b) => a.price - b.price);
      }
      return filtered
    },

    // Получение информации из LocalStorage
    storageProducts() {
      let filtered = this.sortedProducts;
      let oldItems = JSON.parse(localStorage.getItem('testObject')) || [];
      for (let i = 0; i < filtered.length; i++) {
        for (let j = 0; j < oldItems.length; j++) {
          if (filtered[i].id === oldItems[j].id) {
            filtered[i].favourite = true;
          }
        }
      }
      return filtered
    }
  },
  filters: {

    // Фильтр для разделения каждых 3 разрядов в цене пробелом
    spaceDigits(value) {
      return value.toString().replace(/(\d)(?=(\d\d\d)+([^\d]|$))/g, "$1 ");
    }
  }
};
</script>

<style scoped>
body {
  margin: 0;
}

.price-selection {
  margin: 10px;
  font-family: Ubuntu, Arial;
}

.price-selection * {
  padding: 0 5px;
}

.categories, .sort {
  margin: 0 15px;
  display: inline-block;
  font-family: Ubuntu, Arial;
}

.favourites {
  margin: 10px 15px;
}

.product-container {
  position: relative;
  min-width: 205px;
  margin: 30px;
  margin-left: 10px;
  height: 300px;
  width: 250px;
}

.img-product {
  display: block;
  height: 200px;
  width: 100%;
}

.product-container h3 {
  margin: 5px 0;
  font-family: Ubuntu, Arial;
  text-transform: capitalize;
  font-size: 16px;
}

.product-container span {
  display: block;
  margin: 5px 0;
}

.price {
  font-family: Arial, Helvetica Neue, Helvetica, sans-serif;
  font-size: 14px;
}

.seller {
  font-family: Ubuntu, Roboto, Arial;
  font-size: 14px;
}

.seller-rating {
  font-family: Roboto, Arial, Ubuntu;
  font-size: 14px;
}

.img-amount {
  padding: 3px 5px;
  position: absolute;
  top: 56%;
  left: 4%;
  color: white;
  background-color: grey;
  border-radius: 50%;
}

.fav {
  position: absolute;
  top: 73%;
  left: 89%;
  height: 20px;
  cursor: pointer;
}

.active {
  color: red;
}

.products {
  display: flex;
  justify-content: center;
  flex-flow: wrap;
  margin: 40px;
}

.no-products {
  font-family: Ubuntu, Arial;
  text-align: center;
}

@media screen and (max-width: 425px) {
  .price-selection * {
    width: 20%;
    font-size: 12px;
  }

  .categories, .sort {
    font-size: 12px;
  }
}

@media screen and (max-width: 375px) {
  .price-selection * {
    width: 15%;
  }

  .price-selection button {
    width: 27%
  }
}
</style>
