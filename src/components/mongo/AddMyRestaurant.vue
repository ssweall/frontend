<template>
  <div v-if="this.haveARestaurant == false" class="list row">
    <h2>Ajouter un restaurant</h2>
    <form v-on:submit.prevent="submitForm">
      <div class="form-group">
        <label for="name">Nom *</label>
        <input
          type="text"
          class="form-control"
          id="name"
          placeholder="Nom du restaurant"
          v-model="form.name"
          required="required"
        />
      </div>
      <!-- <div class="form-group">
        <label for="idRestaurateur">Restaurateur *</label>
        <input
          type="string"
          class="form-control"
          id="idRestaurateur"
          placeholder="Selectionner le restaurateur"
          v-model="form.idRestaurateur"
          required="required"
        />
      </div> -->
      <div class="form-group">
        <label for="address">Adresse *</label>
        <input
          type="text"
          class="form-control"
          id="address"
          placeholder="Adresse"
          v-model="form.address"
          required="required"
        />
      </div>
      <div class="form-group">
        <input
          @change="handleImage"
          class="custom-input"
          type="file"
          accept="image/*"
        />
      </div>
      <div @click="scrollToTop">
        <div class="form-group">
          <button class="green_button styled_button">Valider</button>
        </div>
      </div>
    </form>
  </div>
</template>

<script>
import DataService from "../../services/DataService";
import axios from "axios";
import jwt_decode from "jwt-decode";
const user = JSON.parse(localStorage.getItem("user"));
let haveARestaurant;
export default {
  name: "PostFormAxios",
  data() {
    return {
      form: {
        name: "",
        idRestaurateur: "",
        address: "",
        picture: ""
      },
      haveARestaurant: false
    };
  },
  methods: {
    decodeToken(token) {
      return jwt_decode(token);
    },
    retrieveRestaurant() {
      this.payloadUser = this.decodeToken(user.accessToken);
      this.userId = this.payloadUser.userId;
      DataService.getAllRestaurantsByRestaurateur(this.payloadUser.userId)
        .then(response => {
          this.restaurantId = response.data.restaurants[0]._id;
          console.log("Utilisateur: " + this.userId);
          console.log("le restau: " + this.restaurantId);
          DataService.getOneRestaurant(this.restaurantId)
            .then(response => {
              this.restaurant = response.data.restaurant;
              console.log(this.restaurant);
              if (this.restaurant) {
                this.haveARestaurant = true;
                console.log(this.haveARestaurant);
              }
            })
            .catch(e => {
              console.log(e);
            });
        })
        .catch(e => {
          console.log(e);
        });
    },
    submitForm() {
      this.form.picture = this.image;
      this.payloadUser = this.decodeToken(user.accessToken);
      this.form.idRestaurateur = this.payloadUser.userId;
      axios
        .post("http://10.117.129.194:8080/api/restaurants/create", this.form, {
          headers: {
            "X-Server-Select": "mongo"
          }
        })
        .then(res => {
          //Perform Success Action
          console.log("donnéee" + res.data);
          location.reload();
        })
        .catch(error => {
          console.log(error);
          // error.response.status Check status code
          console.log("err: " + error);
        })
        .finally(() => {
          //Perform action in always
          console.log("finally");
        });
      // location.reload();
    },
    scrollToTop() {
      window.scrollTo(0, 0);
    },
    handleImage(e) {
      const selectedImage = e.target.files[0];
      this.createBase64Image(selectedImage);
    },
    createBase64Image(fileObject) {
      const reader = new FileReader();

      reader.onload = e => {
        this.image = e.target.result;
      };
      reader.readAsDataURL(fileObject);
    },
    isItFalse() {
      console.log(haveARestaurant);
    }
  },
  mounted() {
    this.retrieveRestaurant();
    this.isItFalse();
  }
};
</script>

<style scoped>
.form-group {
  margin-bottom: 10px;
}
</style>
