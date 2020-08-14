<template>
  <main>
    <br/>
    <v-container fluid grid-list-md>
      <v-layout column align-left>
        <blockquote>
          Welcome {{validUserName}}!
          <footer>
            <small>
              <em>&mdash; Every great film should seem new every time you see it. - Roger Ebert</em>
            </small>
          </footer>
        </blockquote>
      </v-layout>

      <v-layout column align-center>
        <v-flex xs6 sm8 md7>
          <v-alert v-if="showMsg === 'new'"
                   dismissible
                   :value="true"
                   type="success"
          >
            New movie has been added.
          </v-alert>
          <v-alert v-if="showMsg === 'update'" dismissible
                   :value="true"
                   type="success"
          >
            Movie information has been updated.
          </v-alert>
          <v-alert v-if="showMsg === 'deleted'" dismissible
                   :value="true"
                   type="success"
          >
            Selected Movie has been deleted.
          </v-alert>

        </v-flex>
      </v-layout>
      <br/>
      <v-container fluid grid-list-md fill-height>
        <v-layout column>
          <v-flex md6>
            <v-data-table
              :headers="headers"
              :items="movies"
              hide-actions
              class="elevation-1"
              fixed
              style="max-height: 300px; overflow-y: auto"
            >

              <template slot="items" slot-scope="props" >
                
                <td>{{ props.item.name }}</td>
                <td nowrap="true">{{ props.item.description }}</td>
                <td nowrap="true">{{ props.item.year }}</td>
                <td nowrap="true">{{ props.item.rating }}</td>
                <td nowrap="true">
                  <v-icon @click="updateMovie(props.item)">edit</v-icon>
                </td>
                <td nowrap="true">
                  <v-icon @click="deleteMovie(props.item)">delete</v-icon>
                </td>

              </template>

            </v-data-table>
          </v-flex>
        </v-layout>
      </v-container>

      <v-btn class="blue white--text" @click="addNewMovie">Add Movie</v-btn>
    </v-container>

  </main>
</template>


<script>

  import router from '../router';
  import {APIService} from '../http/APIService';
  const apiService = new APIService();

  export default {
    name: "MovieList",
    data: () => ({
      movies: [],
      validUserName: "Guest",
      movieSize: 0,
      showMsg: '',
      headers: [
        {text: 'Name', sortable: false, align: 'left',},
        {text: 'Description', sortable: false, align: 'left',},
        {text: 'Year', sortable: false, align: 'left',},
        {text: 'Rating', sortable: false, align: 'left',},
        {text: 'Update', sortable: false, align: 'left',},
        {text: 'Delete', sortable: false, align: 'left',}
      ],

    }),
    mounted() {
      this.getMovies();
      this.showMessages();
    },
    methods: {
      showMessages(){
        console.log(this.$route.params.msg)
        if (this.$route.params.msg) {
          this.showMsg = this.$route.params.msg;
        }
      },
      getMovies() {
        apiService.getMovieList().then(response => {
          this.movies = response.data.data;
          this.movieSize = this.movies.length;
          if (localStorage.getItem("isAuthenticates")
            && JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
            this.validUserName = JSON.parse(localStorage.getItem("log_user"));
          }
        }).catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem('isAuthenticates');
            localStorage.removeItem('log_user');
            localStorage.removeItem('token');
            router.push("/auth");
          }
        });
      },
      addNewMovie() {
        if (localStorage.getItem("isAuthenticates")
          && JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
          router.push('movie-create');
        } else {
          router.push("/auth");
        }
      },
      updateMovie(movie) {
        router.push('/movie-create/' + movie.pk);
      },
      deleteMovie(movie) {
        apiService.deleteMovie(movie.pk).then(response => {
          if (response.status === 204) {
            alert("Movie deleted");
            this.showMsg = 'deleted';
            this.$router.go();
          }
        }).catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem('isAuthenticates');
            localStorage.removeItem('log_user');
            localStorage.removeItem('token');
            router.push("/auth");
          }
        });
      }
    }
  };
</script>
