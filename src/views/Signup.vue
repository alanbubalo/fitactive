<template>
  <!-- Signup -->
  <div class="header-image text-white">
    <img class="img" alt="FitActive Logo" src="@/assets/deadlift.jpg" />
    <div class="title">Sign Up</div>
  </div>
  <div class="container-fluid p-4">
    <div class="row">
      <div class="col-lg col-md"></div>
      <div class="col-lg-3 col-md-6">
        <div v-if="errorMessage" class="alert alert-danger" role="alert">
          {{ errorMessage }}
        </div>
        <form @submit.prevent="signup">
          <!-- Email Address -->
          <div class="form-group my-2">
            <label for="exampleInputEmail1" class="py-1">Email address</label>
            <input
              type="email"
              v-model="email"
              class="form-control box-shadow"
              id="exampleInputEmail1"
              aria-describedby="emailHelp"
              placeholder="Enter email"
            />
          </div>
          <!-- Password -->
          <div class="form-group my-2">
            <label for="exampleInputPassword1" class="py-1">Password</label>
            <div class="d-flex justify-content-between bd-highlight">
              <div class="flex-grow-1 bd-highlight me-3">
                <input
                  :type="type"
                  v-model="password"
                  class="form-control box-shadow"
                  id="exampleInputPassword1"
                  placeholder="Password"
                />
              </div>
              <div class="flex-1 bd-highlight">
                <button
                  class="btn small-btn-primary box-shadow py-1"
                  @click.prevent="showPassword"
                  type="button"
                  :class="btnText == 'Show' ? 'bg-white' : 'bg-primary'"
                >
                  {{ btnText }}
                </button>
              </div>
            </div>
          </div>
          <!-- Confirm Password -->
          <div class="form-group my-2">
            <label for="exampleInputConfirmPassword1" class="py-1"
              >Confirm Password</label
            >
            <input
              type="password"
              v-model="passwordConfirm"
              class="form-control box-shadow"
              id="exampleInputConfirmPassword1"
              placeholder="Confirm Password"
            />
          </div>
          <div v-if="!isLoading">
            <!-- Sign Up Button -->
            <button type="submit" class="btn my-btn bg-primary box-shadow">
              Sign up
            </button>
          </div>
          <div v-else>
            <button
              type="submit"
              class="btn my-btn bg-primary disabled box-shadow"
            >
              <div class="spinner-border spinner-border-sm" role="status">
                <span class="visually-hidden">Loading...</span>
              </div>
              Signing up...
            </button>
          </div>
        </form>
        <!-- Already have an account? -->
        <div class="d-flex justify-content-between bd-highlight">
          <p>
            Already have an account?
            <router-link
              to="/login"
              class="hover-left my-link-primary bd-highlight"
              replace
              >Log in</router-link
            >
          </p>
        </div>
      </div>
      <div class="col-lg col-md"></div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.img {
  width: 100%;
  height: 350px;
  object-fit: cover;
  object-position: 50% 30%;
  border-radius: 0 0 3rem 3rem;
}
</style>

<script>
// @ is an alias to /src
import { firebase, db } from "@/firebase";

export default {
  name: "Signup",
  data() {
    return {
      email: "",
      password: "",
      passwordConfirm: "",
      type: "password",
      btnText: "Show",
      errorMessage: "",
      isLoading: false,
    };
  },
  methods: {
    async signup() {
      if (this.password !== this.passwordConfirm) {
        this.errorMessage = "Passwords do not match. Try again.";
        return;
      }
      this.isLoading = true;
      firebase
        .auth()
        .setPersistence(firebase.auth.Auth.Persistence.SESSION)
        .then(() => {
          firebase
            .auth()
            .createUserWithEmailAndPassword(this.email, this.password)
            .then(() => {
              console.log("Created an account!");
              db.collection("waterIntake")
                .doc(this.email)
                .set({
                  glassesDrank: 0,
                  glassesTotal: 10,
                })
                .then((doc) => {
                  db.collection("notifications")
                    .doc(this.email)
                    .set({
                      notifs: [],
                    })
                    .then((doc) => {})
                    .catch((error) => {
                      console.error(error);
                    });
                })
                .catch((error) => {
                  console.error(error);
                });
            })
            .catch((error) => {
              this.isLoading = false;
              var mes = error.message.slice(10);
              var mesa = mes.split(" (auth");
              this.errorMessage = mesa[0];
              console.error(error);
            });
        });

      console.log("Signing up...");
    },
    showPassword() {
      this.type = this.type == "password" ? "text" : "password";
      this.btnText = this.btnText == "Show" ? "Hide" : "Show";
    },
  },
};
</script>
