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
      const token = await this.$recaptcha("save");
      console.log(token);

      if (token) {
        fetch(
          `  https://www.google.com/recaptcha/api/siteverify?secret=6LfHRAwbAAAAABB3dlImlVHnHay9sWjMOYtWwrq0&response=${token} `,
          {
            mode: "no-cors",
            // credentials: "include",
            method: "POST",
            headers: { "Content-Type": "application/x-www-form-urlencoded" },
          }
        )
          .then((res) => res.json())
          .then((data) => console.log(data))
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
