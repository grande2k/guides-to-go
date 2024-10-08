<template>
    <section class="login page page--bg">
        <h2 class="login__title form-title" v-text="$t('login_title')"/>

        <form action="" class="login__form" @submit.prevent="submitForm">
            <div class="login__form-field">
                <label for="email" class="form-label" v-text="$t('email')"/>

                <input
                    type="email"
                    class="form-input"
                    :class="{ error: v$.email.$errors.length }"
                    id="email"
                    :placeholder="$t('email_placeholder')"
                    v-model="form_data.email">
            </div>

            <div class="login__form-field">
                <label for="password" class="form-label" v-text="$t('password')"/>

                <div class="form-input-wrapper">
                    <input
                        :type="isPasswordVisible ? 'text' : 'password'"
                        class="form-input"
                        :class="{ error: v$.password.$errors.length }"
                        id="password"
                        :placeholder="$t('password_placeholder')"
                        v-model="form_data.password">

                    <img v-if="!isPasswordVisible" src="@/assets/images/icons/eye-open.svg" @click="isPasswordVisible = true" alt="Show password">
                    <img v-else src="@/assets/images/icons/eye-close.svg" @click="isPasswordVisible = false" alt="Hide password">
                </div>

                <span v-if="v$.password.$errors.length && v$.password.minLength.$invalid" class="form-error" v-text="$t('error_password_length')"/>
            </div>

            <submit-button text="login_button" icon="login" :loading="response_loading"/>
        </form>

        <router-link to="/forgot-password" class="forgot-password" v-text="$t('forgot_password')"/>

        <div class="form-links">
            <router-link to="/signup" class="form-link" v-text="$t('signup_title')"/>
            
            <a :href="`mailto:${appStore.admin_email}`" class="form-link">
                <img src="@/assets/images/icons/mail.svg" alt="Mail">
                {{ $t('contact_admin') }}
            </a>
        </div>
    </section>
</template>

<script setup>
    import { ref, computed, reactive, inject, onMounted } from 'vue';
    import { useVuelidate } from '@vuelidate/core';
    import { required, email, minLength } from '@vuelidate/validators';
    import { useToast } from 'vue-toastification';
    import { useI18n } from 'vue-i18n';
    import { useRouter } from 'vue-router';
    import { useAppStore } from '@/stores/app';
    import { login } from '@/api';
    import SubmitButton from '@/components/SubmitButton.vue';

    const $cookies = inject('$cookies');
    const router = useRouter();
    const { t } = useI18n();
    const appStore = useAppStore();
    const toast = useToast();
    const isPasswordVisible = ref(false);

    const response_loading = ref(false);

    onMounted(() => {
        if ( $cookies.get("access_token") && $cookies.get("user_role") ) {
            router.push('/profile');
        }
    });

    const form_data = reactive({
        email: "",
        password: ""
    });

    const rules = computed(() => {
        return {
            email: { required, email },
            password: { required, minLength: minLength(6) },
        }
    });

    const v$ = useVuelidate(rules, form_data);

    const submitForm = async () => {
        const result = await v$.value.$validate();

        if (result) {
            response_loading.value = true;
            await login(form_data, router, t);
            response_loading.value = false;
        } else {
            toast.error( t('error_validation') );
        }
    }

    const clearForm = () => {
        form_data.email = "";
        form_data.password = "";
    }
</script>

<style lang="scss" scoped>
    .login {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        min-height: 650px;
        @media screen and (max-width: 480px) {
            min-height: unset;
            display: block;
        }
        &__form {
            &-field {
                &:not(:last-child) {
                    margin-bottom: 1rem;
                    @media screen and (max-width: 480px) {
                        margin-bottom: 0.75rem;
                    }
                }
            }
            .form-submit {
                margin-top: 2rem;
                @media screen and (max-width: 480px) {
                    margin-top: 1.25rem;
                }
            }
        }

        .forgot-password {
            display: block;
            text-align: center;
            text-decoration: none;
            color: $white;
            font-weight: 500;
            margin-top: 1.25rem;
            font-size: 1.25rem;
        }
    }
</style>