<template>
  <div class="container">

  <div class="container mt-3 mb-5">
    <h3>Add Adoption</h3> 
  </div>

  <div class="container">
    <h5>Search for Eligible Application</h5> 
  </div>

    <form class="mb-5">
      <div class="input-group">
        <input
          v-on:keydown.13.prevent="search()"
          v-model="searchTerm"
          type="text"
          class="form-control"
          placeholder="Enter Applicant or Co-Applicant Last Name..."
        >
        <div class="input-group-append">
          <button 
            @click="search()" 
            type = "button" 
            class="btn btn-secondary"
          >Search
          </button>
        </div>
      </div>
      <div class="container mt-4 text-info" 
        v-if="searchResults.length">
        <h5>Results:</h5>
        <table class="table">
          <thead>
            <tr>
              <th>First Name</th>
              <th>Last Name</th>
              <th>Email Address</th>
              <th>Phone Number</th>
              <th>Street</th>
              <th>City</th>
              <th>Zip</th>
              <th>Co-Applicant Last Name</th>
            </tr>
          </thead>
          <tbody id="search-results">
            <tr v-for="(sr, index) in searchResults" :key="index" 
              v-on:click="selectedApplication(sr.email_address)">
                <td>{{sr.first_name}}</td>
                <td>{{sr.last_name}}</td>
                <td>{{sr.email_address}}</td>
                <td>{{sr.phone_number}}</td>
                <td>{{sr.street}}</td>
                <td>{{sr.city}}</td>
                <td>{{sr.zip}}</td>
                <td>{{sr.coapplicant_last_name}}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="mt-4 text-danger font-bold" 
        v-if="!searchResults.length">
        <h5>{{searchError}}</h5>  
      </div>
    </form>
  </div>
</template>

<script>

export default {
  name: 'Adoption',
  
  data() {
    return {
      searchTerm: '',
      searchResults: [],
      searchError: ''
    };
  },
 
  methods: {
    search: function() {
      this.getData(this.searchTerm);
    },

    selectedApplication(email) {
      this.$router.push('/dashboard/dog-detail/adoption/selected-application/' + email);
    },

    getData: function(searchTerm) {
      if (searchTerm == '') {
        return;
      }
      searchTerm = searchTerm.toLowerCase();
      this.searchResults = [];
      this.searchError = '';

      var url = "http://127.0.0.1:5000/api/eligibleapplication/" + searchTerm;

      var yourConfig = {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': 'JWT ' + this.$auth.getAccessToken()
       }
      }

      this.$axios
      .get(url, yourConfig)
      .then(response => {
        if (typeof response.data.email_address != 'undefined') {
           this.searchResults.push(response.data);
        } else {
            for (var i = 0; i < response.data.length; i++) {
                this.searchResults.push(response.data[i]);
            }
        }
      })
      .catch((error) => {
        if(error.response.status == 404) {
          if (typeof error.response.data.message != 'undefined') {
            this.searchError = error.response.data.message;
          } else {
            this.searchError = 'Records Not Found';
          }
        }
      })
    }
  }
};
</script>

<style>
  #search-results td{
    cursor: pointer;
    color: blue;
  }
  #search-results td:hover {
    text-decoration: underline;
  }
   #search-results tr:hover {
   background-color: rgb(192, 237, 255);
  }
</style>