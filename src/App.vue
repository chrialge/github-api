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
    getGithub() {
      this.inputValue = document.getElementById('search').value.trim().replace(' ', '-');
      this.filterValue = document.getElementById('filter').value;

      if (this.inputValue.length > 3) {
        if (this.filterValue !== 'Seleziona una') {
          this.Loading = true;
          console.log(this.inputValue.length, this.filterValue)
          this.callAPI();
        } else {
          throw new ('devi selezionare un filtro')
        }

      } else {
        throw new ("deve essere di almeno tre caratteri la parola")
      }
    },

    callAPI() {
      axios({
        method: 'get',
        url: `https://api.github.com/search/${this.filterValue}?q=${this.inputValue}&page=${this.page}&per_page=${this.per_page}`,
        per_page: 50,
        headers: {
          "x-github-api-version": "2022-11-28",
        },
      })
        .then(resp => {
          console.log(resp);
          this.data = resp.data.items;
          if (this.countpages === 1) {
            this.countpages = Math.ceil(resp.data.total_count / this.per_page);
            if (this.countpages > 10) {
              this.page_reduct = []
              for (let index = 1; index <= 10; index++) {
                this.page_reduct.push(index);

              }
            }
          }

          this.Loading = false
        }).catch((error) => {
          console.error(error)
        })
    },
    goToPage(number) {
      this.page = number;
      this.Loading = true;

      const diff = number - this.page_reduct[0];

      const pages = this.page_reduct.map((num) => {
        return num + diff
      });
      this.page_reduct = null;
      this.page_reduct = pages;

      this.callAPI();

    },
    prevPage() {
      this.page--;
      this.Loading = true;

      const pages = this.page_reduct.map((num) => {
        return num - 1;
      });
      this.page_reduct = null;
      this.page_reduct = pages;
      this.callAPI();


    },
    nextPage() {
      this.page++;
      this.Loading = true;

      const pages = this.page_reduct.map((num) => {
        return num + 1;
      });
      console.log(pages)
      this.page_reduct = null;
      this.page_reduct = pages;
      this.callAPI();
    },
    getDebounce() {
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


  <template v-if="this.Loading === false">
    <header>
      <nav class=" navbar navbar-expand-sm navbar-dark bg-dark px-2 justify-content-between">
        <a class="navbar-brand" href="#">Github API</a>


        <form class="form-inline my-2 my-lg-0 d-flex align-items-center bg-white rounded-2" @submit.prevent>

          <div>
            <input type="text" class="form-control border-0" name="search" id="search" aria-describedby="helpId"
              placeholder="Cerca.." @keyup="getDebounce()" />
          </div>

          <div style="border-left: 1px solid gray;">

            <select class="form-select border-0" name="filter" id="filter">

              <option value="repositories">Cartelle</option>
              <option value="users">Utenti</option>

            </select>
          </div>



          <button class="bg-warning my-2 my-sm-0" type="submit"
            style="padding: 0.375rem 0.75rem; border: 1px solid gray; border-top-right-radius: 5px; border-bottom-right-radius: 5px;"
            @click="getGithub()">Search</button>
        </form>

      </nav>
    </header>

    <main id="site_main">

      <div class="container px-4 py-5">
        <div class="row gap-2 justify-content-center" v-if="data.length >= 1">

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
              <p class="card-text" v-else>
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
        <div v-else>
          <span>Non esistono risultati</span>
        </div>

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
