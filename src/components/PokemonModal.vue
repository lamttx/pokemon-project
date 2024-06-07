<template>
  <div v-show="openModal" class="modal-view">
    <div class="modal">
      <h3>{{ showItem.name }}</h3>
      <div class="modal-content">
        <div class="modal-block-image">
          <img class="modal-image" :src="imageUrl" alt="Image" />
        </div>
        <div class="modal-block-info">
          <div class="block-info">
            <b>Total: </b>
            <p>{{ showItem.total }}</p>
          </div>
          <div class="block-info">
            <b>HP: </b>
            <p>{{ showItem.hp }}</p>
          </div>
        </div>
      </div>
      <div class="modal-footer">
        <button class="buttonView" @click="close">Close</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'PokemonModal',

  props: {
    openModal: Boolean,
    pokemonId: String
  },

  data() {
    return {
      showItem: {},
      imageUrl: null,
    }
  },

  watch: {
    openModal(bool) {
      if (bool) {
        this.getItemImage(this.pokemonId);
        this.getItem(this.pokemonId);
      }
    }
  },

  methods: {
    async getItemImage(id) {
      await axios({
        url: `https://api.vandvietnam.com/api/pokemon-api/pokemons/${id}/sprite`,
        method: 'GET',
        responseType: 'blob'
      }).then((response) => {
        this.imageUrl = window.URL.createObjectURL(new Blob([response.data]));
      })
    },

    async getItem(id) {
      await axios
        .get(`https://api.vandvietnam.com/api/pokemon-api/pokemons/${id}`)
        .then((response) => {
          if (response.data) {
            this.showItem = response.data.data;
          }
        })
        .catch((error) => {
          console.error(error)
        })
    },

    close() {
      this.imageUrl = null
      this.loadingDone = false
      this.$emit('closeModal')
    },
  }
}
</script>
