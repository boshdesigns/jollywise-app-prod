<template>
  <section class="container">
    <div class="row">
      <div class="col-sm-12">
        <h1>Jollywise App</h1>

        <!-- Form Step 1 -->
        <form id="step1" class="form form--questions" :class="[(formState === 'step1') ? 'form--show' : 'form--hidden']">

          <div class="form-group" v-for="(value, key, index) in form.step1">
            <label :for="'question' + (index + 1)">Question {{index + 1}} : {{value.question}}</label>
            <section class="container">
              <input :id="'question' + (index + 1)" v-model="value.answer" type="text" class="form-control" placeholder="Enter your answer...">
            </section>
          </div>

          <button v-on:click.self.prevent="changeFormState()" type="submit" class="btn btn-primary">Next</button>

        </form>


        <!-- Form Step 2 -->
        <form id="step2" class="form form--sumbit" :class="[(formState === 'step2') ? 'form--show' : 'form--hidden']">
          <aside>
            <ul>
              <li v-for="(value, key, index) in form.step1">
                <div>{{value.question}}</div><div>{{"Answer: " + value.answer}}</div>
              </li>
            </ul>
          </aside>

          <ul class="validate-errors" v-if="validateErrors.length">
            <li class="alert alert-warning" role="alert" v-for="error in validateErrors">
              {{ error }}
            </li>
          </ul>

          <div class="form-group" v-for="(value, key, index) in form.step2">
            <label :for="key">Your {{ key | capitalise }}</label>
            <section class="container" v-if="key === 'firstname'">
              <input :id="value.value" v-model="value.value" type="text" class="form-control" placeholder="Enter your firstname...">
            </section>
            <section class="container" v-else-if="key === 'lastname'">
              <input :id="value.value" v-model="value.value" type="text" class="form-control" placeholder="Enter your lastname...">
            </section>
            <section class="container" v-else-if="key === 'email'">
              <input :id="value.value" v-model="value.value" type="text" class="form-control" placeholder="Enter your email address...">
            </section>
          </div>

          <button v-on:click.self.prevent="changeFormState()" type="submit" class="btn btn-primary">Back</button>
          <button v-on:click.self.prevent="validate()" type="submit" class="btn btn-primary">Submit</button>
        </form>
      </div>
    </div>
  </section>
</template>

<script>
// Import the baseURL from https-config
import {HTTPS} from '@/config/https-config.js'

export default {
  name: 'Questions',
  data() {
    return {
      formState: 'step1',
      validateErrors: [],
      form: {
        step1: {
          q1: {
            question: 'What is you name?',
            answer: ''
          },
          q2: {
            question: 'What is your quest?',
            answer: ''
          },
          q3: {
            question: 'What is your favorite colour?',
            answer: ''
          },
          q4: {
            question: 'What is the capital of Assyria?',
            answer: ''
          },
          q5: {
            question: 'What is the air-speed velocity of an unladen swallow?',
            answer: ''
          },
        },
        step2: {
          firstname: {
            value: ''
          },
          lastname: {
            value: ''
          },
          email: {
            value: ''
          }
        }
      }
    }
  },

  methods: {
    changeFormState: function () {
      // TODO: make this dynamic by checking against event.srcElement.form.id
      (this.formState == 'step1') ? this.formState = 'step2' : this.formState = 'step1'
    },

    toObject: function (arr) {
      var rv = {};
      for (var i = 0; i < arr.length; ++i)
        if (arr[i] !== undefined) rv[i] = {"answer":arr[i]};
      return rv;
    },

    // Create simple validation
    validate: function (event) {
      this.validateErrors = [];
      if(!this.form.step2.firstname.value) this.validateErrors.push("Firstname required.");
      if(!this.form.step2.lastname.value) this.validateErrors.push("Lastname required.");
      if(!this.form.step2.email.value) {
        this.validateErrors.push("Email required.");
      } else if(!this.validEmail(this.form.step2.email.value)) {
        this.validateErrors.push("Valid email required.");
      }
      // if validation is all good trigger the post
      if(!this.validateErrors.length) {
        this.submitResponse()
      }
    },

    validEmail: function (email) {
      var re = /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
      return re.test(email);
    },

    submitResponse: function () {

      // Rip out the answers so these sit in their own object
      let answers = []

      // add the answers to the empty array
      // We will change these back to an object
      for (var key in this.form.step1) {
        answers.push(this.form.step1[key].answer);
      }

      // Setup the data set to Post
      const postData = {
        "social_type": "none",
        "access_token": "none",
        "access_token_secret": "none",
        "device_id": "none",
        "email": this.form.step2.email.value,
        "first_name": this.form.step2.firstname.value,
        "last_name": this.form.step2.lastname.value,
        "dob": "none",
        "entrant_data": this.toObject(answers),
        "entrant_private_data": "none",
        "data": "none",
        "private_data": "none",
        "searchable": "none",
        "_xss_cookie": "none"
      }

      // test setting up FormData
      var formData = new FormData();
      formData.append('file', 'file');

      HTTPS.post('ashtest', formData)
        .then(response => {
          console.log(response.data);
        })
        .catch(e => {
          this.postErrors.push(e)
        })
    },
  },
  filters: {
    capitalise: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
aside {
  background-color: white;
  -webkit-box-shadow: 0px 0px 5px 0px rgba(0,0,0,0.15);
  -moz-box-shadow: 0px 0px 5px 0px rgba(0,0,0,0.15);
  box-shadow: 0px 0px 5px 0px rgba(0,0,0,0.15);

  margin: {
    top: 20px;
    right: auto;
    bottom: 20px;
    left: auto;
  }
  padding: 15px;


  text-align: center;
  max-width: 600px;

  ul {
    margin: 0;
    padding: 0;
    li {
      list-style: none;
      margin-bottom: 10px;
    }
  }
}

h1 {
  color: hsl(211, 100%, 75%);
  margin-bottom: 80px;
}

.form {
  &--hidden {
    opacity: 0;
    height: 0;
    overflow: hidden;
    visibility: hidden;
    -webkit-transition: all 0.1s ease-in-out;
    -moz-transition: all 0.1s ease-in-out;
    -o-transition: all 0.1s ease-in-out;
    transition: all 0.1s ease-in-out;
  }

  &--show {
    opacity: 1;
    height: auto;
    visibility: visible;
    -webkit-transition: all 0.7s ease-in-out;
    -moz-transition: all 0.7s ease-in-out;
    -o-transition: all 0.7s ease-in-out;
    transition: all 0.7s ease-in-out;
  }
}

.container {
  max-width: 720px;
}

label {
  padding: {
    right: 15px;
    left: 15px;
  }
  text-align: center;
  margin{
    right: auto;
    left: auto;
  }
  width: 100%;
}

.field {
  margin-bottom: 10px;
}

input {
  // TODO: setup autoprefixer
  border-style: none;
  -webkit-border-radius: 30px;
  -moz-border-radius: 30px;
  border-radius: 30px;
  -webkit-box-shadow: 0px 0px 5px 0px rgba(0,0,0,0.15);
  -moz-box-shadow: 0px 0px 5px 0px rgba(0,0,0,0.15);
  box-shadow: 0px 0px 5px 0px rgba(0,0,0,0.15);

  padding: {
    top: 10px;
    right: 10px;
    bottom: 10px;
    left: 20px;
  }

  min-height: 30px;
  width: 100%;

  &::placeholder {
    color: hsl(0, 0%, 80%);
  }
}

.validate-errors {
  list-style: none;
  margin: {
    top: 20px;
    bottom: 20px;
  }
  padding: 0;
}

[v-cloak] {
  display: none;
}

</style>
