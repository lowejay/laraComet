<template>
    <div class="login-page">
        <div class="login">
            <div class="register-container auth-container">
                <div class="register-image-column">
                    <div class="image-holder">
                        <img src="#" alt="" />
                    </div>
                </div>
                <div class="register-form-column">
                    <form v-on:submit.prevent="registerAppUser">
                        <h3>Create Account</h3>
                        <div class="form-wrapper">
                            <label for="username">Username</label>
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
                        <div class="form-wrapper">
                            <label for="password_confirmation"
                                >Confirm Password</label
                            >
                            <input
                                type="password"
                                name="password_confirmation"
                                id="password_confirmation"
                                v-model="password_confirmation"
                                placeholder="Re-enter password"
                                class="form-control"
                                required
                            />
                        </div>
                        <button type="submit" class="mt-3">
                            Register &nbsp;&nbsp;<span
                                v-if="showSpinner"
                                class="fa fa-spin fa-spinner"
                            ></span>
                        </button>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            username: "",
            password: "",
            password_confirmation: "",
            showSpinner: false
        };
    },
    methods: {
        registerAppUser() {
            if (this.username && this.password && this.password_confirmation) {
                this.showSpinner = true;
                if (this.password && this.password_confirmation) {
                    let data = {
                        username: this.username,
                        password: this.password,
                        password_confirmation: this.password_confirmation
                    };
                    axios
                        .post(`/register`, data)
                        .then(response => {
                            console.log(response.data.status);
                            if (response.data.status) {
                                this.createUserOnCometChat(this.username);
                            }
                            this.showSpinner = false;
                        })
                        .catch(error => {
                            console.log(error.response.data.message);
                            this.showSpinner = false;
                        });
                }
            }
        },
        redirect() {
            window.location.href = "/login";
        },
        async createUserOnCometChat(username) {
            let url = `https://api-us.cometchat.io/v2.0/users`;
            let data = {
                uid: username,
                name: `${username}`,
                avatar:
                    "https://data-us.cometchat.io/assets/images/avatars/captainamerica.png"
            };
            try {
                const userDetails = await fetch(url, {
                    method: "POST",
                    headers: new Headers({
                        appid: process.env.MIX_COMMETCHAT_APP_ID,
                        apikey: process.env.MIX_COMMETCHAT_API_KEY,
                        "Content-Type": "application/json"
                    }),
                    body: JSON.stringify(data)
                });
                const userJson = await userDetails.json();
                // console.log('New User', userJson);
                this.createAuthTokenAndSaveForUser(username);
            } catch (error) {
                console.log("Error", error);
            }
        },
        async createAuthTokenAndSaveForUser(uid) {
            let url = `https://api-us.cometchat.io/v2.0/users/${uid}/auth_tokens`;
            try {
                const tokenDetails = await fetch(url, {
                    method: "POST",
                    headers: new Headers({
                        appid: process.env.MIX_COMMETCHAT_APP_ID,
                        apikey: process.env.MIX_COMMETCHAT_API_KEY,
                        "Content-Type": "application/json"
                    })
                });
                const tokenJSON = await tokenDetails.json();
                console.log(tokenJSON);
                this.sendTokenToServer(
                    tokenJSON.data.authToken,
                    tokenJSON.data.uid
                );
            } catch (error) {
                console.log("Error Token", error);
            }
        },
        sendTokenToServer(token, uid) {
            axios
                .post(`/update/token`, { token, uid })
                .then(response => {
                    // console.log("Token updated successfully", response);
                    this.redirect();
                })
                .catch(error => {
                    console.log(error.response.data.message);
                });
        }
    }
};
</script>
