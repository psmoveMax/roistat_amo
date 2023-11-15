<template>
  <div>
    <form @submit.prevent="handleSubmit">
      <div>
        <label for="name">Имя:</label>
        <input type="text" id="name" v-model="form.name">
      </div>
      <div>
        <label for="email">Email:</label>
        <input type="email" id="email" v-model="form.email">
      </div>
      <div>
        <label for="phone">Телефон:</label>
        <input type="tel" id="phone" v-model="form.phone">
      </div>
      <div>
        <label for="price">Цена:</label>
        <input type="number" id="price" v-model="form.price">
      </div>
      <button type="submit">Отправить</button>
    </form>
  </div>

  <div v-if="showModal" class="modal">
    <div class="modal-content">
      <span class="close" @click="showModal=false">&times;</span>
      <p>{{modalMessage}}</p>
    </div>

  </div>
</template>

<script>
import axios from 'axios';
import { config, updateAccessToken } from '../../config';

export default {
  data() {
    return {
      form: {
        name: '',
        email: '',
        phone: '',
        price: '',
      },
      showModal: false,
      modalMessage:'',
    };
  },
  methods: {

    async handleSubmit() {

      const infoData = {
        name: this.form.name,
        email: this.form.email,
        phone: this.form.phone,
        price: this.form.price,
      };


      await this.createComplexDeal(infoData,config.accessToken);
    },

    async getAuthToken() {
      const newToken = await axios.post('https://macsimawvakumov.amocrm.ru/oauth2/access_token', {
        client_id: config.clientId,
        client_secret: config.clientSecret,
        grant_type: "refresh_token",
        refresh_token: config.refreshToken,
        redirect_uri: "http://localhost:8080/"
      });

      config.refreshToken = newToken.data['refresh_token'];
      config.accessToken = newToken.data['access_token'];

      return newToken.data['access_token'];
    },



    async createComplexDeal(infoData, accessToken) {
      try {
        const response = await axios.post('https://macsimawvakumov.amocrm.ru/api/v4/leads/complex',
            [
              {
                name: `Сделка от ${infoData.name}`,
                price: infoData.price,
                "_embedded": {
                  "contacts": [
                    {
                      "first_name": infoData.name,
                      "custom_fields_values": [
                        {
                          "field_code": "PHONE",
                          "values": [
                            {
                              "value": infoData.phone,
                            }
                          ]
                        },
                        {
                          "field_code": "EMAIL",
                          "values": [
                            {
                              "value": infoData.email,
                            }
                          ]
                        },
                      ]
                    }
                  ]
                }
              }
            ], {
          headers: {
            'Authorization': `Bearer ${accessToken}`,
            'Content-Type': 'application/json'
          }
        });
        this.modalMessage = 'Форма отправилась успешно';
        this.showModal = true;
        return response.data;
      } catch (error) {
        if (error.response && error.response.status === 401) {
           const newAuthToken = await this.getAuthToken();
          updateAccessToken(newAuthToken);
          this.modalMessage = 'Форма отправилась успешно';
          this.showModal = true;
          return this.createComplexDeal(infoData, newAuthToken);
        } else {
          this.modalMessage = 'Ошибка при отправке данных: '+ error.message;
          this.showModal = true;
          throw error;
        }
      }
    },
  }
}
</script>


<style>
.modal {
  display: block;
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgb(0,0,0);
  background-color: rgba(0,0,0,0.4);
  padding-top: 60px;
}

.modal-content {
  background-color: #fefefe;
  margin: 5% auto;
  padding: 20px;
  border: 1px solid #888;
  width: 80%;
}

.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: black;
  text-decoration: none;
  cursor: pointer;
}
form{
  max-width: max-content;
  display: inline-block;
}
</style>
