<template>
  <div v-show="openModal" class="modal-view">
    <!-- <ScaleLoader :loading="loadingDone" color="rgba(235, 235, 235, 0.64)" height="30" width="40" /> -->

    <div class="modal-content">
      <div class="modal-block-image">
        <img class="modal-image" :src="imageUrl" alt="Image" />
      </div>
      <div class="modal-block-info">
        
      </div>
    </div>
    <button @click="close">Close</button>
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
      imageUrl: null
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
    }
  }
}
</script>
