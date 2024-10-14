<script>

import axios from 'axios';

export default {
  data() {
    name: 'App'

    return {
      data: [],
      per_page: 10,
      page: 1,
      countpages: 1,
      page_reduct: false,
      inputValue: null,
      filterValue: null,
      Loading: false,
    }
  },
  methods: {

    /**
     * questa funzione prende il risultato dell'input con il filtro e prende i risultati
     */
    getGithub() {

      // salvo il valore del input
      this.inputValue = document.getElementById('search').value.trim().replace(' ', '-');

      // salvo il valore del filtraggio
      this.filterValue = document.getElementById('filter').value;

      // se e sopra di tre il valore di input
      if (this.inputValue.length > 3) {

        // se il filtro e diverso da 
        if (this.filterValue !== 'Seleziona una') {

          // il loading diventa vero
          this.Loading = true;

          // invoco la funzione
          this.callAPI();
        } else {

          // comparira l'errore in console
          throw new ('devi selezionare un filtro')
        }

      } else {
        // comparirea l'errore in console
        throw new ("deve essere di almeno tre caratteri la parola")
      }
    },

    /**
     * funzione dove inseririsco la logica della request api
     */
    callAPI() {

      // faccio una chiamata api 
      axios({
        method: 'get',
        url: `https://api.github.com/search/${this.filterValue}?q=${this.inputValue}&page=${this.page}&per_page=${this.per_page}`,
        per_page: 50,
        headers: {
          "x-github-api-version": "2022-11-28",
        },
      })
        .then(resp => {

          // salvo i risultati nell'istanza data
          this.data = resp.data.items;

          // se countapages e uno
          if (this.countpages === 1) {

            // salvo il numero di pagine che ci sono
            this.countpages = Math.ceil(resp.data.total_count / this.per_page);

            // seci sono piu di 10 pagine
            if (this.countpages > 10) {

              // creo un array che usero per il paginate
              this.page_reduct = []

              // itero per i prime dieci pagine
              for (let index = 1; index <= 10; index++) {
                // pusho il numero in page_reduct
                this.page_reduct.push(index);

              }
            }
          }

          // loading diventa falso
          this.Loading = false
        }).catch((error) => {
          // faccio vedere il tipo di errore della chiamata
          console.error(error)
        })
    },

    /**
     * funzione che ti porta alla pagina che clicci
     * @param number numero della pagina
     */
    goToPage(number) {

      // inserisco il numero della pagina nell'istanza
      this.page = number;

      // do come valore true al caricamento
      this.Loading = true;

      // costante che prende la differenza 
      const diff = number - this.page_reduct[0];

      // creo un array con il nuovo paginate
      const pages = this.page_reduct.map((num) => {
        return num + diff
      });

      // prima vuoto il risultato
      this.page_reduct = null;

      // poi salvo il nuovo paginate 
      this.page_reduct = pages;

      // invoco la funzione
      this.callAPI();

    },

    /**
     * funzione che ti porta alla -pagina precedente
     */
    prevPage() {

      // diminuisco di 1 della pagina
      this.page--;

      // do come valore true al caricamento
      this.Loading = true;

      // creo un array con il nuovo paginate
      const pages = this.page_reduct.map((num) => {
        return num - 1;
      });

      // prima vuoto il risultato
      this.page_reduct = null;

      // poi salvo il nuovo paginate 
      this.page_reduct = pages;

      // invoco la funzione
      this.callAPI();
    },

    /**
     * funzione che ti passa alla pagina dopo
     */
    nextPage() {

      // aumenta di 1
      this.page++;

      // do come valore true al caricamento
      this.Loading = true;

      // creo un array con il nuovo paginate
      const pages = this.page_reduct.map((num) => {
        return num + 1;
      });

      // prima vuoto il risultato
      this.page_reduct = null;

      // poi salvo il nuovo paginate 
      this.page_reduct = pages;

      // invoco la funzione
      this.callAPI();
    },

    /**
     * funzione che parte se non inserisco niente dopo 7 secondi nell'input
     */
    getDebounce() {

      // setto che dopo 7s invoco la funzione
      setTimeout(() => {
        this.getGithub();
      }, 7000);
    }

  },
  mounted() {
    console.log('ciao')


  }
}

</script>

<template>

  <!-- template quando non e in fase di caricamento -->
  <template v-if="this.Loading === false">


    <!-- header -->
    <header>
      <nav class=" navbar navbar-expand-sm navbar-dark bg-dark px-2 justify-content-between">

        <!-- logo -->
        <a class="navbar-brand" href="#">Github API</a>


        <form class="form-inline my-2 my-lg-0 d-flex align-items-center bg-white rounded-2" @submit.prevent>

          <!-- input -->
          <div>
            <input type="text" class="form-control border-0" name="search" id="search" aria-describedby="helpId"
              placeholder="Cerca.." @keyup="getDebounce()" />
          </div>

          <!-- select -->
          <div style="border-left: 1px solid gray;">
            <select class="form-select border-0" name="filter" id="filter">
              <option value="repositories">Cartelle</option>
              <option value="users">Utenti</option>
            </select>
          </div>


          <!-- button -->
          <button class="bg-warning my-2 my-sm-0" type="submit"
            style="padding: 0.375rem 0.75rem; border: 1px solid gray; border-top-right-radius: 5px; border-bottom-right-radius: 5px;"
            @click="getGithub()">Search</button>
        </form>

      </nav>
    </header>

    <!-- main -->
    <main id="site_main">

      <div class="container px-4 py-5">
        <div class="row gap-2 justify-content-center" v-if="data.length >= 1">

          <!-- se ci sono risultati -->
          <div class="card p-0" v-for="data in data" style="background-color: #efefef; max-width: 250px; width: 100%;">
            <div class="card_header">
              <div class="card_background">

              </div>
              <img class="img_card" :src="data.owner.avatar_url" alt="" v-if="data.owner">
              <img class="img_card" :src="data.avatar_url" alt="" v-else>


            </div>

            <div class="card-body">
              <h4 class="card-title" v-if="data.full_name">{{ data.full_name }}</h4>
              <h4 class="card-title" v-else>{{ data.login }}</h4>

              <p class="card-text" v-if="data.description">{{ data.description }}</p>
              <p class="card-text" v-if="data.type">
                <strong>Profilo: </strong>
                <span>{{ data.type }}</span>
              </p>
              <div class="star_card d-flex align-items-center gap-2" v-if="data.stargazers_count">
                <i class="fa fa-star" aria-hidden="true"></i>
                <span>{{ data.stargazers_count }}</span>

              </div>
              <div class="issue_card d-flex align-items-center gap-2" v-if="data.open_issues_count">
                <i class="fa-solid fa-circle-exclamation"></i>
                <span>{{ data.open_issues_count }}</span>
              </div>


            </div>
            <div class="card-footer text-center">
              <a :href="data.html_url" class="text-decoration-none text-dark" target="_blank">

                <span v-if="data.owner">Vai alla repo</span>
                <span v-else>Vai al profilo</span>
                <i class="fa-solid fa-arrow-up-right-from-square"></i>
              </a>
            </div>

          </div>
        </div>

        <!-- in caso non ci sono risultrati -->
        <div v-else>
          <span>Non esistono risultati</span>
        </div>

        <!-- paginate -->
        <div class="paginate d-flex gap-2 mt-5 align-items-center" v-if="this.countpages > 1">
          <div class="paginate_arrow_left" v-if="this.page > 1" @click="prevPage">
            <i class="fa fa-arrow-left" aria-hidden="true"></i>
          </div>
          <div class="number_page d-flex gap-1">
            <span v-for="n in this.countpages" v-if="this.page_reduct === false" @click="goToPage(n)">{{ n }}</span>
            <span v-else v-for="number in this.page_reduct" @click="goToPage(number)"
              :class="[number === this.page ? 'active_page' : '']">{{ number }}</span>

          </div>

          <div class=" paginate_arrow_right" v-if="this.page < this.countpages" @click="nextPage">
            <i class="fa fa-arrow-right" aria-hidden="true"></i>

          </div>

        </div>
      </div>
    </main>
  </template>
  <template v-if="this.Loading">
    <div class="container_loading">
      <svg viewBox="0 0 100 100">
        <g fill="none" stroke="#fff" stroke-linecap="round" stroke-linejoin="round" stroke-width="6">
          <!-- left line -->
          <path d="M 21 40 V 59">
            <animateTransform attributeName="transform" attributeType="XML" type="rotate" values="0 21 59; 180 21 59"
              dur="2s" repeatCount="indefinite" />
          </path>
          <!-- right line -->
          <path d="M 79 40 V 59">
            <animateTransform attributeName="transform" attributeType="XML" type="rotate" values="0 79 59; -180 79 59"
              dur="2s" repeatCount="indefinite" />
          </path>
          <!-- top line -->
          <path d="M 50 21 V 40">
            <animate attributeName="d" values="M 50 21 V 40; M 50 59 V 40" dur="2s" repeatCount="indefinite" />
          </path>
          <!-- btm line -->
          <path d="M 50 60 V 79">
            <animate attributeName="d" values="M 50 60 V 79; M 50 98 V 79" dur="2s" repeatCount="indefinite" />
          </path>
          <!-- top box -->
          <path d="M 50 21 L 79 40 L 50 60 L 21 40 Z">
            <animate attributeName="stroke" values="rgba(255,255,255,1); rgba(100,100,100,0)" dur="2s"
              repeatCount="indefinite" />
          </path>
          <!-- mid box -->
          <path d="M 50 40 L 79 59 L 50 79 L 21 59 Z" />
          <!-- btm box -->
          <path d="M 50 59 L 79 78 L 50 98 L 21 78 Z">
            <animate attributeName="stroke" values="rgba(100,100,100,0); rgba(255,255,255,1)" dur="2s"
              repeatCount="indefinite" />
          </path>
          <animateTransform attributeName="transform" attributeType="XML" type="translate" values="0 0; 0 -19" dur="2s"
            repeatCount="indefinite" />
        </g>
      </svg>
    </div>

  </template>







</template>

<style></style>
