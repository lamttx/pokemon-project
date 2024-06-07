<template>
  <div class="page-layout">
    <div>
      <label>Filter by type</label>
      <select v-model="selectedValue" class="form-select">
        <option v-for="type in pokemonTypes" :value="type.id" :key="type.id">
          {{ type.name }}
        </option>
      </select>
    </div>
    <table>
      <thead>
        <tr>
          <th class="pointer" :class="{ asc: isAsc, desc: !isAsc, active: column == 'number' }"
            @click="sortBy('number')">
            No.
          </th>
          <th>Name</th>
          <th>Type</th>
          <th class="pointer" :class="{ asc: isAsc, desc: !isAsc, active: column == 'total' }" @click="sortBy('total')">
            Total
          </th>
          <th class="pointer" :class="{ asc: isAsc, desc: !isAsc, active: column == 'hp' }" @click="sortBy('hp')">
            HP
          </th>
          <th class="pointer" :class="{ asc: isAsc, desc: !isAsc, active: column == 'attack' }"
            @click="sortBy('attack')">
            Attack
          </th>
          <th class="pointer" :class="{ asc: isAsc, desc: !isAsc, active: column == 'defense' }"
            @click="sortBy('defense')">
            Defense
          </th>
          <th class="pointer" :class="{ asc: isAsc, desc: !isAsc, active: column == 'sp_atk' }"
            @click="sortBy('sp_atk')">
            Sp. Attack
          </th>
          <th class="pointer" :class="{ asc: isAsc, desc: !isAsc, active: column == 'sp_def' }"
            @click="sortBy('sp_def')">
            Sp. Defense
          </th>
          <th class="pointer" :class="{ asc: isAsc, desc: !isAsc, active: column == 'speed' }" @click="sortBy('speed')">
            Speed
          </th>
          <th>View Detail</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="pokemon in pokemons" :key="pokemon.id">
          <td>{{ pokemon.number }}</td>
          <td>{{ pokemon.name }}</td>
          <td>{{ matchTypeName(pokemon.type_1, pokemon.type_2) }}</td>
          <td>{{ pokemon.total }}</td>
          <td>{{ pokemon.hp }}</td>
          <td>{{ pokemon.attack }}</td>
          <td>{{ pokemon.defense }}</td>
          <td>{{ pokemon.sp_atk }}</td>
          <td>{{ pokemon.sp_def }}</td>
          <td>{{ pokemon.speed }}</td>
          <td>
            <button class="buttonView" @click="openPokemonModal(pokemon)">View</button>
          </td>
        </tr>
      </tbody>
    </table>
    <div class="pagination-field">
      <ul id="pagination">
        <li><a :class="{ disabled: currentPage == 1, pointer: currentPage > 1 }"
            @click="showPage(currentPage - 1)">«</a></li>
        <li v-for="(page, $index) in totalPage" :key=$index>
          <a @click="showPage(page)" :class="{ active: page == currentPage }" class="pointer">
            {{ page }}
          </a>
        </li>
        <li><a :class="{ disabled: currentPage == totalPage, pointer: currentPage < totalPage }"
            @click="showPage(currentPage + 1)">»</a></li>
      </ul>
    </div>
  </div>

  <Modal :open-modal="viewPokemon" :pokemon-id="pokemonID" @closeModal="viewPokemon = !viewPokemon" />
</template>

<script>
import axios from 'axios';
import PokemonModal from './PokemonModal.vue';

export default {
  name: 'PokemonList',

  components: {
    Modal: PokemonModal
  },

  data() {
    return {
      pokemons: [],
      isAsc: true,
      selectedValue: '',
      column: 'number',
      sortKeyword: 'number',
      listAllTypesName: [],
      pokemonTypes: [],
      viewPokemon: false,
      pokemonID: null,
      totalPage: 0,
      currentPage: 0,
      perPage: 0,
    }
  },

  computed: {},

  mounted() {
    this.getListTypeName();
    this.sortPokemon('number', null, null);
  },

  methods: {
    async sortPokemon(keyword, filterType, filterValue) {
      await axios
        .get(
          `https://api.vandvietnam.com/api/pokemon-api/pokemons?page[number]=1&page[size]=10&sort=${keyword}&filter[${filterType}]=${filterValue}`
        )
        .then((response) => {
          if (response.data) {
            this.pokemons = response.data.data;
            this.totalPage = response.data.meta.last_page;
            this.currentPage = response.data.meta.current_page;
            this.perPage = response.data.meta.per_page;
          }
        })
        .catch((error) => {
          console.error(error)
        })
    },

    sortBy(keyword) {
      this.column = keyword; // change active column
      keyword = this.isAsc ? '-' + keyword : keyword;
      this.sortPokemon(keyword, null, null);
      this.isAsc = !this.isAsc;
      this.sortKeyword = keyword;
    },

    async getListTypeName() {
      await axios
        .get(`https://api.vandvietnam.com/api/pokemon-api/types`)
        .then((response) => {
          if (response.data) {
            this.listAllTypesName = response.data.data;
            this.pokemonTypes = [...this.listAllTypesName];
            this.pokemonTypes.unshift({ id: '', name: 'All' });
          }
        })
        .catch((error) => {
          console.error(error);
        })
    },

    matchTypeName(type1, type2) {
      let list = this.listAllTypesName.filter((i) => {
        return i.id === type1 || i.id === type2;
      })
      return list.map((item) => item.name).join(', ');
    },

    openPokemonModal(item) {
      this.viewPokemon = true;
      this.pokemonID = item.id;
    },

    async showPage(pageNum) {
      console.log(pageNum);
      await axios
        .get(
          `https://api.vandvietnam.com/api/pokemon-api/pokemons?page[number]=${pageNum}&page[size]=10&sort=${this.sortKeyword}`
        )
        .then((response) => {
          if (response.data) {
            this.pokemons = response.data.data;
            this.currentPage = pageNum;
          }
        })
        .catch((error) => {
          console.error(error)
        })
    },
  }
}
</script>
