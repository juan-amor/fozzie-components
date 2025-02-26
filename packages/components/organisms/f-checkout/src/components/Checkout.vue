<template>
    <div>
        <component
            :is="messageType.name"
            v-if="message"
            ref="errorMessage"
            v-bind="messageType.props"
        >
            <span>{{ messageType.content }}</span>
        </component>

        <div
            v-if="shouldShowSpinner"
            :class="$style['c-spinner-wrapper']"
            data-test-id="checkout-loading-spinner"
        >
            <div :class="$style['c-spinner']" />
        </div>
        <div
            v-else-if="shouldShowCheckoutForm"
            data-theme="jet"
            data-test-id="checkout-component"
        >
            <card
                is-rounded
                has-outline
                is-page-content-wrapper
                card-heading-position="center"
                data-test-id="checkout-card-component"
                :class="$style['c-checkout']"
            >
                <checkout-header :login-url="loginUrl" />

                <form
                    method="post"
                    :class="$style['c-checkout-form']"
                    @submit.prevent="onFormSubmit"
                >
                    <guest-block v-if="!isLoggedIn" />

                    <form-field
                        :value="customer.mobileNumber"
                        name="mobile-number"
                        input-type="tel"
                        :label-text="$t('labels.mobileNumber')"
                        :has-error="!isMobileNumberValid"
                        @input="updateCustomerDetails({ mobileNumber: $event })"
                    >
                        <template #error>
                            <error-message
                                v-if="!isMobileNumberValid"
                                data-js-error-message
                                data-test-id="error-mobile-number"
                            >
                                {{ $t('validationMessages.mobileNumber.requiredError') }}
                            </error-message>
                        </template>
                    </form-field>

                    <address-block
                        v-if="isCheckoutMethodDelivery"
                        data-test-id="address-block"
                    />

                    <form-selector />

                    <user-note
                        data-test-id="user-note"
                        @input="updateUserNote($event.target.value)"
                    />

                    <f-button
                        :class="$style['c-checkout-submitButton']"
                        button-type="primary"
                        button-size="large"
                        is-full-width
                        action-type="submit"
                        data-test-id="confirm-payment-submit-button"
                    >
                        {{ $t('buttonText') }}
                    </f-button>
                </form>

                <checkout-terms-and-conditions v-if="!isLoggedIn" />
            </card>
        </div>

        <error-page v-else-if="shouldShowErrorPage" />
    </div>
</template>

<script>
import { validationMixin } from 'vuelidate';
import { required, email } from 'vuelidate/lib/validators';
import { mapActions, mapState } from 'vuex';
import Alert from '@justeat/f-alert';
import '@justeat/f-alert/dist/f-alert.css';
import FButton from '@justeat/f-button';
import '@justeat/f-button/dist/f-button.css';
import Card from '@justeat/f-card';
import '@justeat/f-card/dist/f-card.css';
import ErrorMessage from '@justeat/f-error-message';
import '@justeat/f-error-message/dist/f-error-message.css';
import FormField from '@justeat/f-form-field';
import '@justeat/f-form-field/dist/f-form-field.css';
import { validations } from '@justeat/f-services';
import { VueGlobalisationMixin } from '@justeat/f-globalisation';
import VueScrollTo from 'vue-scrollto';
import AddressBlock from './Address.vue';
import CheckoutHeader from './Header.vue';
import CheckoutTermsAndConditions from './TermsAndConditions.vue';
import FormSelector from './Selector.vue';
import GuestBlock from './Guest.vue';
import UserNote from './UserNote.vue';
import ErrorDialog from './ErrorDialog.vue';
import ErrorPage from './Error.vue';
import exceptions from '../exceptions/exceptions';
import {
    ANALYTICS_ERROR_CODE_INVALID_MODEL_STATE,
    CHECKOUT_METHOD_DELIVERY,
    TENANT_MAP,
    VALIDATIONS,
    VUEX_CHECKOUT_ANALYTICS_MODULE,
    VUEX_CHECKOUT_MODULE
} from '../constants';
import checkoutValidationsMixin from '../mixins/validations.mixin';
import loggerMixin from '../mixins/logger.mixin';
import EventNames from '../event-names';
import tenantConfigs from '../tenants';
import { mapUpdateCheckoutRequest, mapAnalyticsNames } from '../services/mapper';

const {
    CreateGuestUserError,
    UpdateCheckoutError,
    PlaceOrderError
} = exceptions;

export default {
    name: 'VueCheckout',

    components: {
        AddressBlock,
        Alert,
        FButton,
        Card,
        CheckoutHeader,
        CheckoutTermsAndConditions,
        ErrorPage,
        ErrorMessage,
        FormField,
        FormSelector,
        GuestBlock,
        UserNote,
        ErrorDialog
    },

    mixins: [
        validationMixin,
        VueGlobalisationMixin,
        checkoutValidationsMixin,
        loggerMixin
    ],

    props: {
        getCheckoutUrl: {
            type: String,
            required: true
        },

        checkoutAvailableFulfilmentUrl: {
            type: String,
            required: true
        },

        createGuestUrl: {
            type: String,
            required: true
        },

        getBasketUrl: {
            type: String,
            required: true
        },

        placeOrderUrl: {
            type: String,
            required: true
        },

        paymentPageUrlPrefix: {
            type: String,
            required: true
        },

        updateCheckoutUrl: {
            type: String,
            required: true
        },

        checkoutTimeout: {
            type: Number,
            required: false,
            default: 10000
        },

        spinnerTimeout: {
            type: Number,
            required: false,
            default: 500
        },

        authToken: {
            type: String,
            default: ''
        },

        otacToAuthExchanger: {
            type: Function,
            default: () => {
                throw new Error('otacToAuthExchanger is not implemented');
            }
        },

        loginUrl: {
            type: String,
            required: true
        },

        getAddressUrl: {
            type: String,
            required: true
        },

        applicationName: {
            type: String,
            required: true
        },

        getGeoLocationUrl: {
            type: String,
            required: true
        }
    },

    data () {
        return {
            tenantConfigs,
            hasCheckoutLoadedSuccessfully: true,
            shouldShowSpinner: false,
            isLoading: false
        };
    },

    /*
    * Provide/Inject allows nested `Address` component to inherit `Checkout`
    * validator scope, `$v`.
    */
    provide () {
        const $v = {};

        Object.defineProperty($v, VALIDATIONS.address, {
            enumerable: true,
            get: () => this.$v.address
        });

        Object.defineProperty($v, VALIDATIONS.guest, {
            enumerable: true,
            get: () => this.$v.customer
        });

        return { $v };
    },

    computed: {
        ...mapState(VUEX_CHECKOUT_MODULE, [
            'availableFulfilment',
            'address',
            'basket',
            'customer',
            'errors',
            'geolocation',
            'isGuestCreated',
            'hasAsapSelected',
            'id',
            'isFulfillable',
            'isLoggedIn',
            'messages',
            'message',
            'notices',
            'orderId',
            'restaurant',
            'serviceType',
            'time',
            'userNote'
        ]),

        isMobileNumberValid () {
            /*
            * Validation methods return true if the validation conditions
            * have not been met and the field has been `touched` by a user.
            * The $dirty boolean changes to true when the user has focused/lost
            * focus on the input field.
            */
            return !this.$v.customer.mobileNumber.$dirty || this.$v.customer.mobileNumber.isValidPhoneNumber;
        },

        isCheckoutMethodDelivery () {
            return this.serviceType === CHECKOUT_METHOD_DELIVERY;
        },

        tenant () {
            return TENANT_MAP[this.$i18n.locale];
        },

        shouldLoadAddress () {
            return this.isLoggedIn &&
                this.isCheckoutMethodDelivery &&
                (!this.address || !this.address.line1);
        },

        shouldShowCheckoutForm () {
            return !this.isLoading && this.hasCheckoutLoadedSuccessfully;
        },

        shouldShowErrorPage () {
            return !this.hasCheckoutLoadedSuccessfully;
        },

        eventData () {
            return {
                isLoggedIn: this.isLoggedIn,
                serviceType: this.serviceType
            };
        },

        messageType () {
            return this.message && this.message.shouldShowInDialog
                ? this.dialogMessage
                : this.alertMessage;
        },

        dialogMessage () {
            return {
                name: 'error-dialog'
            };
        },

        alertMessage () {
            return {
                name: 'alert',
                props: {
                    type: 'danger',
                    class: this.$style['c-checkout-alert'],
                    heading: this.$t('errorMessages.errorHeading')
                },
                content: this.message
            };
        }
    },

    watch: {
        /**
         *
         * Only reload checkout if the token changes as below:
         * Truthy > Falsey
         * Falsey > Truthy
         *
         * Do not reload checkout if the token changes from one token to another token
         * as it will always be valid.
         *
         * */
        async authToken (newTokenVal, oldTokenVal) {
            this.setAuthToken(this.authToken);

            if ((!newTokenVal && oldTokenVal) || (!oldTokenVal && newTokenVal)) {
                await this.initialise();
            }
        }
    },

    async mounted () {
        this.setAuthToken(this.authToken);

        await this.initialise();
        this.trackInitialLoad();
    },

    methods: {
        ...mapActions(VUEX_CHECKOUT_MODULE, [
            'createGuestUser',
            'getAvailableFulfilment',
            'getAddress',
            'getCustomerName',
            'getBasket',
            'getCheckout',
            'getGeoLocation',
            'placeOrder',
            'setAuthToken',
            'updateCheckout',
            'updateCustomerDetails',
            'updateMessage',
            'updateUserNote'
        ]),

        ...mapActions(VUEX_CHECKOUT_ANALYTICS_MODULE, [
            'trackFormErrors',
            'trackFormInteraction',
            'trackInitialLoad'
        ]),

        /**
         * Loads the necessary data to render a meaningful checkout component.
         *
         */
        async initialise () {
            this.isLoading = true;

            this.startSpinnerCountdown();

            const promises = this.isLoggedIn
                ? [this.loadBasket(), this.loadCheckout(), this.loadAvailableFulfilment()]
                : [this.loadBasket(), this.loadAvailableFulfilment()];

            await Promise.all(promises);
            this.resetLoadingState();

            if (this.shouldLoadAddress) {
                await this.loadAddress();
            }
        },

        /**
         * Process all the information to submit the checkout and place an order
         * while emitting events to communicate its success or failure.
         *
         */
        async submitCheckout () {
            try {
                if (!this.isLoggedIn && !this.isGuestCreated) {
                    await this.setupGuestUser();
                }

                await this.lookupGeoLocation();

                await this.handleUpdateCheckout();

                if (this.isFulfillable) {
                    await this.submitOrder();

                    this.redirectToPayment();
                } else {
                    this.handleNonFulfillableCheckout();
                }
            } catch (error) {
                this.handleErrorState(error);
            }
        },

        /**
         * Display and track issues when updating checkout even though the request was successful.
         * (e.g. request is correct, but the restaurant is now offline).
         */
        handleNonFulfillableCheckout () {
            if (this.errors) {
                this.trackFormErrors();

                this.logInvoker({
                    message: 'Consumer Checkout Not Fulfillable',
                    data: this.eventData,
                    logMethod: this.$logger.logWarn
                });

                this.$emit(EventNames.CheckoutUpdateFailure, this.eventData);
            }
        },

        /**
         * Handles call of `updateCheckout` and catches and throws any returned errors.
         * 1. Maps checkout data.
         * 2. Call `updateCheckout`.
         * 3. If `updateCheckout` call succeeds but issues are returned, it will be handled by
         *    its parent method
         * 4. If `updateCheckout` call fails, throw an UpdateCheckoutError.
         */
        async handleUpdateCheckout () {
            try {
                const data = mapUpdateCheckoutRequest({
                    address: this.address,
                    customer: this.customer,
                    isCheckoutMethodDelivery: this.isCheckoutMethodDelivery,
                    time: this.time,
                    userNote: this.userNote,
                    geolocation: this.geolocation,
                    asap: this.hasAsapSelected
                });

                await this.updateCheckout({
                    url: this.updateCheckoutUrl,
                    data,
                    timeout: this.checkoutTimeout
                });

                this.$emit(EventNames.CheckoutUpdateSuccess, this.eventData);
            } catch (e) {
                throw new UpdateCheckoutError(e.message);
            }
        },

        /**
         * Redirect to the payment page.
         */
        redirectToPayment () {
            window.location.assign(`${this.paymentPageUrlPrefix}/${this.orderId}`);
        },

        /**
         * Place the order, emit the expected events, and throw a new PlaceOrderError if the process fails.
         */
        async submitOrder () {
            try {
                const data = {
                    basketId: this.basket.id,
                    customerNotes: {
                        noteForRestaurant: this.userNote
                    },
                    referralState: 'ReferredByWeb'
                };

                await this.placeOrder({
                    url: this.placeOrderUrl,
                    data,
                    timeout: this.checkoutTimeout
                });

                this.$emit(EventNames.CheckoutPlaceOrderSuccess, this.eventData);
                this.$emit(EventNames.CheckoutSuccess, this.eventData);

                this.logInvoker({
                    message: 'Consumer Checkout Successful',
                    data: this.eventData,
                    logMethod: this.$logger.logInfo
                });
            } catch (e) {
                const { errorCode } = e.response.data;

                throw new PlaceOrderError(e.message, errorCode);
            }
        },

        /**
         * Setup a new guest user account. This method will be called when isLoggedIn is false.
         * Events emitted to communicate success or failure.
         * Throw a CreateGuestUserError if it fails.
         */
        async setupGuestUser () {
            try {
                const createGuestData = {
                    emailAddress: this.customer.email,
                    firstName: this.customer.firstName,
                    lastName: this.customer.lastName,
                    registrationSource: 'Guest'
                };

                await this.createGuestUser({
                    url: this.createGuestUrl,
                    tenant: this.tenant,
                    data: createGuestData,
                    timeout: this.checkoutTimeout,
                    otacToAuthExchanger: this.otacToAuthExchanger
                });

                this.$emit(EventNames.CheckoutSetupGuestSuccess);
            } catch (e) {
                throw new CreateGuestUserError(e.message);
            }
        },

        /**
         * Load the checkout details while emitting events to communicate its success or failure.
         *
         */
        async loadCheckout () {
            try {
                await this.getCheckout({
                    url: this.getCheckoutUrl,
                    timeout: this.checkoutTimeout
                });

                this.$emit(EventNames.CheckoutGetSuccess);
            } catch (error) {
                this.$emit(EventNames.CheckoutGetFailure, error);
                this.hasCheckoutLoadedSuccessfully = false;

                this.logInvoker({
                    message: 'Get Checkout Failure',
                    data: this.eventData,
                    logMethod: this.$logger.logError,
                    error
                });
            }
        },

        /**
         * Load the basket details while emitting events to communicate its success or failure.
         *
         */
        async loadBasket () {
            try {
                await this.getBasket({
                    url: this.getBasketUrl,
                    tenant: this.tenant,
                    language: this.$i18n.locale,
                    timeout: this.checkoutTimeout
                });

                this.$emit(EventNames.CheckoutBasketGetSuccess);
            } catch (error) {
                this.$emit(EventNames.CheckoutBasketGetFailure, error);
                this.hasCheckoutLoadedSuccessfully = false;

                this.logInvoker({
                    message: 'Get Checkout Basket Failure',
                    data: this.eventData,
                    logMethod: this.$logger.logError,
                    error
                });
            }
        },

        /**
         * Load the available fulfilment details while emitting events to communicate its success or failure.
         *
         */
        async loadAvailableFulfilment () {
            try {
                await this.getAvailableFulfilment({
                    url: this.checkoutAvailableFulfilmentUrl,
                    timeout: this.checkoutTimeout
                });

                this.$emit(EventNames.CheckoutAvailableFulfilmentGetSuccess);
            } catch (error) {
                this.$emit(EventNames.CheckoutAvailableFulfilmentGetFailure, error);
                this.hasCheckoutLoadedSuccessfully = false;

                this.logInvoker({
                    message: 'Get Checkout Available Fulfilment Times Failure',
                    data: this.eventData,
                    logMethod: this.$logger.logError,
                    error
                });
            }
        },

        /**
         * Load the customer address while emitting events to communicate its success or failure.
         *
         */
        async loadAddress () {
            try {
                await this.getAddress({
                    url: this.getAddressUrl,
                    tenant: this.tenant,
                    language: this.$i18n.locale,
                    timeout: this.checkoutTimeout
                });

                this.$emit(EventNames.CheckoutAddressGetSuccess);
            } catch (error) {
                this.$emit(EventNames.CheckoutAddressGetFailure, error);

                this.logInvoker({
                    message: 'Get checkout address failure',
                    data: this.eventData,
                    logMethod: this.$logger.logWarn,
                    error
                });
            }
        },

        /**
         * Look up the geo details for the customers address, skip on failure.
         *
         */
        async lookupGeoLocation () {
            try {
                const locationData = {
                    addressLines: Object.values(this.address).filter(addressLine => !!addressLine)
                };

                if (locationData.addressLines.length) {
                    await this.getGeoLocation({
                        url: this.getGeoLocationUrl,
                        postData: locationData,
                        timeout: this.checkoutTimeout
                    });
                }
            } catch (error) {
                this.logInvoker({
                    message: 'Geo Location Lookup Failed',
                    data: this.eventData,
                    logMethod: this.$logger.logWarn,
                    error
                });
            }
        },

        /**
         * Emit, log and track the error based on the parameters
         * encapsulated within the 'error' class.
         * Set the `message` for the user to see.
         */
        handleErrorState (error) {
            const message = this.$t(error.messageKey) || this.$t('errorMessages.genericServerError');
            const eventToEmit = error.eventToEmit || EventNames.CheckoutFailure;
            const logMessage = error.logMessage || 'Consumer Checkout Failure';

            this.$emit(eventToEmit, { ...this.eventData, error });

            this.logInvoker({
                message: logMessage,
                data: this.eventData,
                logMethod: this.$logger.logError,
                error
            });

            this.trackFormInteraction({ action: 'error', error: `error_${error.message}` });

            if (!error.shouldShowInDialog) {
                this.updateMessage(message);

                this.$nextTick(() => {
                    this.scrollToElement(this.$refs.errorMessage.$el);
                });
            }
        },

        /**
         * Scroll to a ref element in the screen.
        */
        scrollToElement (element, options = { offset: -20 }) {
            const scrollingDurationInMilliseconds = 650;

            if (element) {
                VueScrollTo.scrollTo(element, scrollingDurationInMilliseconds, options);
            }
        },

        /**
         * Check form is valid - no inline messages
         * 1. If form is valid try to call `submitCheckout`.
         * 2. If the form is invalid process error tracking and logging via `onInvalidCheckoutForm()`.
         */
        async onFormSubmit () {
            this.trackFormInteraction({ action: 'submit' });
            this.updateMessage();

            if (this.isFormValid()) {
                await this.submitCheckout();
            } else {
                this.onInvalidCheckoutForm();
            }
        },

        /**
         * Fired when `isFormValid` returns falsey via `onFormSubmit()` call.
         * 1. Emit `CheckoutValidationError` for consuming application.
         * 2. Process tracking with action type and error fields
         * 3. Log warn.
         *
         * */
        onInvalidCheckoutForm () {
            const validationState = validations.getFormValidationState(this.$v);
            const invalidFields = mapAnalyticsNames(validationState.invalidFields);

            this.scrollToFirstInlineError();

            this.$emit(EventNames.CheckoutValidationError, validationState);
            this.trackFormInteraction({
                action: 'inline_error',
                error: invalidFields
            });

            this.trackFormInteraction({
                action: 'error',
                error: ANALYTICS_ERROR_CODE_INVALID_MODEL_STATE
            });

            this.logInvoker({
                message: 'Checkout Validation Error',
                data: { ...this.eventData, validationState },
                logMethod: this.$logger.logWarn
            });
        },

        /**
         * Scroll to the first inline error, no matter which one it is.
         */
        scrollToFirstInlineError () {
            this.$nextTick(() => {
                const firstInlineError = document.querySelector('[data-js-error-message]');

                this.scrollToElement(firstInlineError, { offset: -100 });
            });
        },

        /**
         * Check to see if any `Vuelidate` validation errors
         */
        isFormValid () {
            this.$v.$touch();
            return !this.$v.$invalid;
        },

        /**
         * Use phone validation in `f-services` to check if customer number is
         * valid in current locale
         */
        isValidPhoneNumber () {
            return validations.isValidPhoneNumber(this.customer.mobileNumber, this.$i18n.locale);
        },

        /**
         * Use postcode validation in `f-services` to check if customer postcode is
         * valid in current locale
         */
        isValidPostcode () {
            return validations.isValidPostcode(this.address.postcode, this.$i18n.locale);
        },

        resetLoadingState () {
            this.isLoading = false;
            this.shouldShowSpinner = false;
        },

        startSpinnerCountdown () {
            setTimeout(() => {
                if (this.isLoading) {
                    this.shouldShowSpinner = true;
                }
            }, this.spinnerTimeout);
        }
    },

    validations () {
        const deliveryDetails = {
            customer: {
                mobileNumber: {
                    isValidPhoneNumber: this.isValidPhoneNumber
                }
            }
        };

        if (!this.isLoggedIn) {
            deliveryDetails.customer = {
                ...deliveryDetails.customer,
                firstName: {
                    required
                },
                lastName: {
                    required
                },
                email: {
                    required,
                    email
                }
            };
        }

        if (this.isCheckoutMethodDelivery) {
            deliveryDetails.address = {
                line1: {
                    required
                },
                locality: {
                    required
                },
                postcode: {
                    required,
                    isValidPostcode: this.isValidPostcode
                }
            };
        }

        return deliveryDetails;
    }
};
</script>

<style lang="scss" module>
@include loadingIndicator('large');

.c-spinner-wrapper {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);

    .c-spinner {
        margin: 0 auto;
    }
}

.c-checkout {
    padding-top: spacing(x6);
    padding-bottom: spacing(x6);

    @include media('<=narrow') {
        border: none;
        padding-top: spacing(x3);
        padding-bottom: spacing(x5);
        margin-top: 0;
    }
}

.c-checkout-form {
    margin-top: spacing(x2);
}

.c-checkout-alert {
    width: $checkout-width;
    margin-left: auto;
    margin-right: auto;
}
/* If these stay the same then just rename the class to something more generic */
.c-checkout-submitButton {
    margin: spacing(x4) 0 spacing(x0.5);
}
</style>
