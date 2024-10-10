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
    }
  },
  methods: {
    getGithub() {
      this.inputValue = document.getElementById('search').value.trim().replace(' ', '-');
      this.filterValue = document.getElementById('filter').value;

      if (this.inputValue.length > 3) {
        if (this.filterValue !== 'Seleziona una') {
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
          this.countpages = Math.ceil(resp.data.total_count / this.per_page);
          if (this.countpages > 10) {
            this.page_reduct = []
            for (let index = 1; index <= 10; index++) {
              this.page_reduct.push(index);

            }
          }

        }).catch((error) => {
          console.error(error)
        })
    }

  },
  mounted() {
    console.log('ciao')


  }
}

</script>

<template>

  <header>
    <nav class=" navbar navbar-expand-sm navbar-dark bg-dark px-2 justify-content-between">
      <a class="navbar-brand" href="#">Github API</a>


      <form class="form-inline my-2 my-lg-0 d-flex align-items-center bg-white rounded-2" @submit.prevent>

        <div>
          <input type="text" class="form-control border-0" name="search" id="search" aria-describedby="helpId"
            placeholder="Cerca.." />
        </div>

        <div style="border-left: 1px solid gray;">

          <select class="form-select border-0" name="filter" id="filter">
            <option selected disabled>Seleziona una</option>
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
      <div class="row row-cols-1 row-cols-sm-2 row-cols-md-3 row-cols-lg-5 gap-2">

        <div class="card p-0" v-for="data in data" style="background-color: #efefef;">
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

      <div class="paginate" v-if="this.countpages > 1">
        <div class="paginate_arrow_left" v-if="this.page > 1">
          <i class="fa fa-arrow-left" aria-hidden="true"></i>
        </div>
        <span v-for="n in this.countpages" v-if="this.page_reduct === false">{{ n }}</span>
        <span v-else v-for="number in this.page_reduct">{{ number }}</span>

        <div class="paginate_arrow_right" v-if="this.page < this.countpages">
          <i class="fa fa-arrow-right" aria-hidden="true"></i>

        </div>

      </div>
    </div>
  </main>




</template>

<style></style>
