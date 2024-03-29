<template lang="pug">
.auth-section
  .logo
  .form-container
    .auth-form.auth-form__login
      .title-form
        h2.title-default.text-center Welcome
        p.text-center.text-20px Log in to your account
      .form
        .form-group(:class="getValidationClass($v, 'identifier')")
          label(for="email") Email
          input#email(
            placeholder="mail@gmail.com",
            type="email",
            v-model="form.identifier",
            @blur="$v.identifier.$touch()"
          )
          span.error(v-if="$v.identifier.required.$invalid") Required field
        .form-group(:class="getValidationClass($v, 'password')")
          .help-message
            label(for="password") Password
          input#password(
            placeholder="********",
            v-model="form.password",
            @blur="$v.password.$touch()"
          )
          button.btn-password.btn__icon(
            :class="{ hide: passwordType === 'text' }",
            @click="showPassword"
          )
          span.error(v-if="$v.password.required.$invalid") Required field
        button.btn__default.btn__action(@click="signIn") Log in

        p.text-center.text-12 New user?
          router-link.login-span(to="/registration") Sign up
</template>

<script lang="ts" setup>
import { computed, reactive } from "vue";

import { required } from "@vuelidate/validators";
import useVuelidate from "@vuelidate/core";

import router from "@/router";

import { addToast } from "@/composables/toaster";
import { passwordType, showPassword } from "@/composables/password";
import { UserInterface } from "@/types/UserInterface";
import { ToasterTypes } from "@/constants/toasterTypes";
import { checkValidation, getValidationClass } from "@/composables/validation";
import { login } from "@/services/authApi";
import { useUserStore } from "@/store/user";

const form = reactive({
  identifier: "",
  password: "",
});

const rules = computed(() => {
  const rules: { [key: string]: any } = {
    identifier: { required },
    password: { required },
  };
  return rules;
});

const $v = useVuelidate(rules, form);

const userStore = useUserStore();

const signIn = () => {
  const authorize = (res: { jwt: string; user: UserInterface }) => {
    if (res.jwt) {
      userStore.setAuthorized();
      userStore.setTokens({ accessToken: res.jwt });
      userStore.saveUser(res.user);
      router.push("dashboard/");
    }
  };
  if (!checkValidation($v.value)) {
    login(form)
      .then((res) => {
        if (res.data) {
          authorize(res.data);
          addToast({
            status: ToasterTypes.success,
            message: "User Authorized",
          });
        }
      })
      .catch(() => {
        addToast({
          status: ToasterTypes.error,
          message: "Invalid identifier or password",
        });
      });
  }
};
</script>

<style scoped lang="scss">
@import "@/scss/_media";
@import "@/scss/styles";
@import "@/scss/auth.scss";

.logo {
  position: absolute;
  height: 80px;
  width: 58px;
  top: 57px;
  right: 98px;
  background-image: url("@/assets/image/logo/logo.svg");
  background-repeat: no-repeat;

  @include media_mobile {
    top: 36px;
    right: inherit;
  }
}

.auth-section {
  position: relative;
  background-image: url("@/assets/image/background-1.svg"),
    url("@/assets/image/background-2.svg");
  background-repeat: no-repeat;
  background-position: top left, bottom right;

  @include media_mobile {
    background-image: url("@/assets/image/background-mobile.svg"),
      url("@/assets/image/background-mobile2.svg");
  }
}
.btn__outline {
  color: var(--black);
  border-color: var(--black);
}

.text-center {
  text-align: center;
}
.text-12 {
  font-family: Inter;
  margin-top: 10px;
  font-size: 12px;
  font-weight: 400;
  line-height: 20px;
  .login-span {
    font-size: 12px;
    text-decoration: underline;
    margin-left: 5px;
  }
}
.text-20px {
  font-family: Inter;
  font-size: 20px;
  font-weight: 500;
  line-height: 26px;

  @include media_mobile {
    font-size: 16px;
    line-height: 20px;
  }
}
.error {
  input {
    border-color: var(--red);
  }

  span {
    display: block;
  }
}
</style>
