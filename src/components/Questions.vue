<template>
  <section class="container">
    <div class="row">
      <div class="col-sm-12">
        <h1>Jollywise App</h1>

        <section :class="[(formState === 'step1') ? 'form--show' : 'form--hidden']">
          <!-- Form Step 1 -->
          <form id="step1" class="form form--questions">

            <div class="form-group" v-for="(value, key, index) in form.step1">
              <label :for="'question' + (index + 1)">Question {{index + 1}} : {{value.question}}</label>
              <section class="container">
                <input :id="'question' + (index + 1)" v-model="value.answer" type="text" class="form-control" placeholder="Enter your answer...">
              </section>
            </div>

            <button v-on:click.self.prevent="changeFormState()" type="submit" class="btn btn-primary">Next</button>

          </form>
        </section>

        <section :class="[(formState === 'step2') ? 'form--show' : 'form--hidden']">
          <!-- Step1 Answers + Form Validate -->
          <aside class="step1-answers">
            <ul class="step1-answers__list">
              <li class="step1-answers__answer" v-for="(value, key, index) in form.step1">
                <div>{{value.question}}</div><div>{{"Answer: " + value.answer}}</div>
              </li>
            </ul>
          </aside>

          <ul class="validate-errors" v-if="validateErrors.length">
            <li class="alert alert-warning" role="alert" v-for="error in validateErrors">
              {{ error }}
            </li>
          </ul>

          <section class="submittion-message" v-if="submitMessage.shouldShow === true">
            <div v-if="submitMessage.messageType === 'success'" class="alert alert-success" role="alert">
              {{submitMessage.message}}
            </div>
            <div v-if="submitMessage.messageType === 'error'" class="alert alert-warning" role="alert">
              {{submitMessage.message}}
            </div>
          </section>

          <!-- Form Step 2 -->
          <form id="step2" class="form form--sumbit">
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

            <section v-if="submitLoader === true" class="loader"></section>

          </form>
        </section>

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
      submitLoader: false,
      submitMessage: {
        shouldShow: false,
        message: '',
        messageType: ''
      },
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

      // Set a loader SVG whilst submitting
      this.submitLoader = true

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

      // Set up the formData to sumbit
      var formData = new FormData(postData);

      // Lets actually set the data to formData
      // This might actually help in submitting the data ** Cheers Paul!
      for ( var key in postData ) {
        formData.append(key, postData[key]);
      }


      // Lastly send the data
      HTTPS.post('ashtest', formData)
        .then(response => {
          if(response.data.result === "SUCCESS") {
            this.submitLoader = false;
            this.submitMessage.messageType = "success";
            this.submitMessage.shouldShow = true;

            this.submitMessage.message = "Your entire has been successfully sent!";

          } else if (response.data.result === "ERROR") {
            this.submitLoader = false;
            this.submitMessage.messageType = "error";
            this.submitMessage.shouldShow = true;

            this.submitMessage.message = "Unfortunately this wasn't successfully this time around.";

          }
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
.step1-answers {
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

#step2 {
  height: 100%;
  padding: {
    top: 20px;
    bottom: 20px;
  }
  position: relative;

  .loader {
    background: {
      color: hsla(0, 0%, 0%, 0.3);
      repeat: no-repeat;
      image: url('../assets/oval.svg');
      position: center center;
    }
    -webkit-border-radius: 30px;
    -moz-border-radius: 30px;
    border-radius: 30px;
    height: 100%;
    position: absolute;
    top: 0;
    width: 100%;
    left: 0;
  }
}

[v-cloak] {
  display: none;
}

</style>
