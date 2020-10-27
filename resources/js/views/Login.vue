<template>
    <div class="login-page">
        <div class="login">
            <div class="login-container auth-container">
                <div class="login-form-column">
                    <form v-on:submit.prevent="authLoginAppUser">
                        @csrf
                        <h3>Enter user credentials</h3>
                        <div class="form-wrapper">
                            <label>Username</label>
                            <input
                                type="text"
                                name="username"
                                id="username"
                                v-model="username"
                                class="form-control"
                                required
                            />
                        </div>
                        <div class="form-wrapper">
                            <label for="password">Password</label>
                            <input
                                type="password"
                                name="password"
                                id="password"
                                v-model="password"
                                class="form-control"
                                required
                            />
                        </div>
                        <button type="submit" class="mt-3">
                            Login &nbsp;&nbsp;<span
                                v-if="showSpinner"
                                class="fa fa-spin fa-spinner"
                            ></span>
                        </button>
                    </form>
                </div>
                <div class="login-image-column">
                    <div class="image-holder">
                        <img src="#" alt="" />
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { CometChat } from "@cometchat-pro/chat";
export default {
    data() {
        return {
            username: "",
            password: "",
            showSpinner: false,
            token: ""
        };
    },
    methods: {
        authLoginAppUser() {
            let userData = {
                username: this.username,
                password: this.password
            };
            this.showSpinner = true;
            if (this.username && this.password) {
                axios
                    .post(`/login`, userData)
                    .then(response => {
                        this.logUserInToCometChat(response.data.user.authToken);
                        this.showSpinner = false;
                    })
                    .catch(error => {
                        console.log(error);
                        this.showSpinner = false;
                    });
            } else {
                console.log("Please fill the required fields");
            }
        },
        logUserInToCometChat(token) {
            CometChat.login(token).then(
                () => {
                    // console.log("successfully login user");
                    window.location.href = "/home";
                },
                error => {
                    this.showSpinner = false;
                    alert("Error occured");
                    console.log("Login failed with error:", error.code);
                }
            );
        }
    }
};
</script>
