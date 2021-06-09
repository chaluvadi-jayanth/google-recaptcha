<template>
  <div class="center">
    <b-form @submit.prevent="formSubmit">
      <b-form-group name="name">
        <label class="sr-only" for="inline-form-input-name">Name</label>
        <b-form-input
          id="inline-form-input-name"
          class="mb-2 mr-sm-2 mb-sm-0"
          v-model="$v.form.name.$model"
          placeholder="Jane Doe"
        ></b-form-input>
        <div class="error" v-if="!$v.form.name.required && submitted">
          Field is required
        </div>
        <div class="error" v-if="!$v.form.name.minLength">
          Name must have at least
          {{ $v.form.name.$params.minLength.min }} letters.
        </div>
      </b-form-group>

      <br />
      <b-form-group name="@userName">
        <label class="sr-only" for="inline-form-input-username">Username</label>
        <b-input-group prepend="@" class="mb-2 mr-sm-2 mb-sm-0">
          <b-form-input
            id="inline-form-input-username"
            v-model="$v.form.userName.$model"
            placeholder="Username"
          ></b-form-input>
        </b-input-group>
        <div class="error" v-if="!$v.form.userName.required && submitted">
          Field is required
        </div>
        <div class="error" v-if="!$v.form.userName.minLength">
          Name must have at least
          {{ $v.form.userName.$params.minLength.min }} letters.
        </div>
      </b-form-group>
      <br />

      <b-button type="submit" variant="primary">Save</b-button>
    </b-form>
  </div>
</template>

<script>
import { validationMixin } from "vuelidate";
import { required, minLength, maxLength } from "vuelidate/lib/validators";
import axios from "axios";
export default {
  data() {
    return {
      submitted: false,
      form: {
        name: "",
        userName: "",
      },
    };
  },
  methods: {
    async postData(url = "") {
      // Default options are marked with *
      axios.create({
        headers: {
          "x-amz-acl": "public-read",
          "Access-Control-Allow-Origin": "*",
          //Authorization: 'Bearer {token}',
          //timeout: 1000, // indicates, 1000ms ie. 1 second
          "Content-Type": "application/json",
        },
      });
      const response = await axios.post(
        url
        // method: "POST", // *GET, POST, PUT, DELETE, etc.
        // mode: "no-cors", // no-cors, *cors, same-origin
        // cache: "no-cache", // *default, no-cache, reload, force-cache, only-if-cached
        // credentials: "same-origin", // include, *same-origin, omit
        // headers: {
        //   "Content-Type": "application/json",
        //   // 'Content-Type': 'application/x-www-form-urlencoded',
        // },
        // redirect: "follow", // manual, *follow, error
        // referrerPolicy: "no-referrer", // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
        // // body: JSON.stringify(data), // body data type must match "Content-Type" header
      );
      return response.json(); // parses JSON response into native JavaScript objects
    },
    formSubmit() {
      this.$v.$touch();
      this.submitted = true;
      if (!this.$v.form.$anyError) {
        console.log("FORM IS VALID");
        console.log(this.form.name);
        console.log(this.form.userName);

        this.googlerecaptcha();
        // location.reload();
      } else console.log("invalid");
    },
    async googlerecaptcha() {
      await this.$recaptchaLoaded();
      const token = await this.$recaptcha("login");
      console.log(token);

      if (token) {
        // this.postData(
        //   `https://www.google.com/recaptcha/api/siteverify?secret=6LfHRAwbAAAAABB3dlImlVHnHay9sWjMOYtWwrq0&response=${token}`
        // ).then((data) => {
        //   console.log(data); // JSON data parsed by `data.json()` call
        // });
        fetch(
          `  https://www.google.com/recaptcha/api/siteverify?secret=6LfHRAwbAAAAABB3dlImlVHnHay9sWjMOYtWwrq0&response=${token} `,
          {
            mode: "no-cors",
            // credentials: "include",
            method: "POST",
            headers: {
              "Content-Type":
                "application/x-www-form-urlencoded,application/json",
            },
          }
        )
          .then((res) => {
            debugger;
            return res.json();
          })
          .then((data) => {
            debugger;
            if (data) console.log(data);
            else alert("no data");
          })
          .catch((err) => console.log(err));
      }
    },
  },
  mixins: [validationMixin],
  validations: {
    form: {
      name: {
        required,
        maxLength: maxLength(16),
        minLength: minLength(4),
      },
      userName: {
        required,
        maxLength: maxLength(16),
        minLength: minLength(4),
      },
    },
  },
};
</script>

<style>
.center {
  width: 100%;
  display: grid;
  place-items: center;
}
.error {
  color: #ff0000;
}
</style>
