<template>
  <LoaderWrapper :loading="$fetchState.pending">
    <div>
      <b-modal id="modal-prevent-closing" ref="modal" :title="(`${name}`)" @show="resetModal" @hidden="resetModal"
        @ok="showdata">
        <div class="card">
          <img class="card-body" :src="(`${image}`)" />
          <div class="card-footer">
            <table class="table">
              <thead>
                <tr>
                  <th class="text-center">
                    Abilities
                  </th>
                  <th class="text-center">
                    Type
                  </th>
                  <th class="text-center">
                    Move
                  </th>
                  <th class="text-center">
                    Weight
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td class="align-middle">
                    <li v-for="ability in abilities">{{ability}}</li>
                  </td>
                  <td class="align-middle">{{type}}</td>
                  <td class="align-middle">{{move}}</td>
                  <td class="align-middle text-center">{{weight}}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </b-modal>
      <table class="table table-bordered">
        <thead>
          <tr>
            <th class="align-middle text-center" scope="col">No</th>
            <th class="text-center" scope="col">Image</th>
            <th class="align-middle text-center" scope="col">Character Name</th>
            <th class="align-middle text-center" scope="col">Actions</th>
          </tr>
        </thead>
        <template>
          <tbody>
            <tr v-for="(char_results, index) in results" :key="char_results">
              <th class="align-middle col-md-1 text-center" scope="row">{{index+1}}</th>
              <td><img class="img-size"
                  :src="(`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${currentData+index+1}.png`)" />
              </td>
              <td class="align-middle text-center">{{char_results.name}}</td>
              <td class="align-middle text-center col-md-3">
                <h5 class="btn btn-success" @click="createData(`${char_results.name}`)">SAVE</h5>
                <h5 class="btn btn-info" v-b-modal.modal-prevent-closing @click="showDetails((currentData+index+1))">
                  DETAILS
                </h5>
              </td>
            </tr>
          </tbody>
        </template>
        <tfoot>
          <tr>
            <td colspan="4">
              <div class="d-flex justify-content-center">
                <div class="p-2">
                  <button @click="decrement" class="btn btn-warning">Previous</button>
                </div>
                <div class="p-2">
                  <span class="form-control"> {{ from }} of {{ total }} </span>
                </div>
                <div class="p-2">
                  <button @click="increment" class="btn btn-warning">Next</button>
                </div>
              </div>
            </td>
          </tr>
        </tfoot>
      </table>
    </div>
  </LoaderWrapper>
</template>

<style scoped>
  .img-size {
    display: block;
    margin-left: auto;
    margin-right: auto;
    width: 20%;
    height: 20%;
  }

</style>

<script>
  var axios = require("axios");
  export default {
    fetchOnServer: false,
    data() {
      return {
        results: [],
        from: 1,
        currentData: 0,
        total: 0,
        limit: 10,
        char_id: 1,
        front_images: [],
        name: '',
        // ---------------------- DETAILS ----------------------
        weight: '',
        image: '',
        type: '',
        move: '',
        abilities: [],
      }
    },
    fetch() {
      this.listsData();
      this.detailsData();
      this.frontImage();
    },
    mounted() {
      this.increment();
    },
    methods: {
      showdata() {
        this.$fetch();
      },
      decrement() {
        if (this.from > 1) {
          this.from--;
          this.currentData = this.from * 10 - 10;
          this.$fetch();
        }
      },
      increment() {
        if (this.from < this.total) {
          this.from++;
          this.currentData = this.from * 10 - 10;
          this.$fetch();
        }
      },
      countPages(all_data) {
        if (all_data / 10 % 10 > 0) {
          return parseInt(all_data / 10) + 1;
        } else {
          return parseInt(all_data / 10);
        }
      },
      showDetails(id) {
        this.char_id = id;
        this.$fetch();
      },
      listsData() {
        const baseURL = `${this.$axios.defaults.baseURL}/pokemon?limit=${this.limit}&offset=${this.currentData}`;
        axios(baseURL).then(response => {
          let datas = response.data;
          this.results = datas.results;
          this.total = this.countPages(datas.count);
        });
      },
      frontImage() {
        for (let i = 1; i <= 10; i++) {
          axios(`${this.$axios.defaults.baseURL}/pokemon/${i}`).then(response => {
            this.front_images.push(response.data.sprites.other.home.front_default);
          })
        }
      },
      createData(names) {
        var datajson = JSON.stringify({
          name: names,
        });
        var config = {
          method: "post",
          url: 'http://127.0.0.1:8000/programming-languages/insert-user',
          data: datajson
        };
        axios(config).then(response => {
          console.log("whoaaps!")
          console.log(response)
        })
      },
      detailsData() {
        const baseURL = `${this.$axios.defaults.baseURL}/pokemon/${this.char_id}`;
        axios(baseURL).then(response => {
          let datas = response.data;
          this.name = datas.name;
          this.weight = datas.weight;
          this.image = datas.sprites.other.home.front_default;
          this.type = datas.types[0].type.name;
          this.move = datas.moves[0].move.name
          this.abilities = []
          for (let i = 0; i < datas.abilities.length; i++) {
            this.abilities.push(datas.abilities[i].ability.name)
          }
        });
      }
    }
  }

</script>
