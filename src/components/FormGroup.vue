<template>
  <div>
    <div>
      <label for="npcity"></label>
      <DropDown
        inputId="npcity"
        :options="npcities"
        v-on:selected="npCitySelection"
        :disabled="false"
        placeholder="Почніть вводити місто"
      />
    </div>
    <div v-if="selectedCity">
      <label for="npwarehouses"></label>
      <DropDown
        inputId="npwarehouses"
        :options="npwarehouses"
        v-on:selected="npWarehouseSelection"
        :disabled="false"
        placeholder="Почніть вводити відділення"
      />
      <input type="hidden" name="delivery_details" v-model="deliveryDetail" required />
    </div>
  </div>
</template>
<script>
import axios from "axios";
import DropDown from "./DropDown.vue";
export default {
  props: {
    apikey: String
  },
  data() {
    return {
      loadingWarehouses: false,
      loadingCities: false,
      cityName: "",
      warehousename: "",
      citieslist: [],
      warehouseslist: [],
      selectedCity: null
    };
  },
  methods: {
    getNpCities() {
      axios({
        method: "post",
        url: "http://api.novaposhta.ua/v2.0/json/",
        data: {
          apiKey: this.apikey,
          modelName: "Address",
          calledMethod: "getCities"
        }
      }).then(response => (this.citieslist = response.data.data));
    },
    getNpWarehouses() {
      axios({
        method: "post",
        url: "http://api.novaposhta.ua/v2.0/json/",
        data: {
          apiKey: this.apikey,
          modelName: "Address",
          calledMethod: "getWarehouses",
          methodProperties: {
            CityRef: this.selectedCity
          }
        }
      }).then(response => (this.warehouseslist = response.data.data));
    },
    npCitySelection(city) {
      console.log(city);
      this.selectedCity = city.id;
      this.cityName = city.name;
      if (this.selectedCity) {
        this.getNpWarehouses();
      }
    },
    npWarehouseSelection(warehouse) {
      console.log(warehouse);
      this.warehousename = warehouse.name;
    }
  },
  components: { DropDown },
  mounted() {
    this.getNpCities();
    console.log("mounted");
  },
  computed: {
    npcities() {
      return this.citieslist.map(obj => {
        return { id: obj.Ref, name: obj.Description, ru: obj.DescriptionRu };
      });
    },
    npwarehouses() {
      return this.warehouseslist.map(obj => {
        return { id: obj.Ref, name: obj.Description, ru: obj.DescriptionRu };
      });
    },
    deliveryDetail() {
      if (this.cityName && this.warehousename) {
        return `${this.cityName}, ${this.warehousename}`;
      } else {
        return "";
      }
    }
  }
};
</script>
