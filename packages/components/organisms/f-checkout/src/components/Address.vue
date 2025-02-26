<template>
    <div data-test-id='address-component'>
        <fieldset :class="$style['c-address-group']">
            <legend
                :class="$style['c-address-label']">
                {{ $t('labels.addressGroup') }}
            </legend>
            <form-field
                :value="address.line1"
                :class="$style['c-address-formField']"
                name="address-line-1"
                :label-text="$t('labels.line1')"
                label-style="inline"
                is-grouped
                :has-error="isAddressLine1Empty"
                @input="updateAddressDetails({ ['line1']: $event })">
                <template #error>
                    <error-message
                        v-if="isAddressLine1Empty"
                        data-js-error-message
                        :class="$style['c-address-error']"
                        data-test-id="error-address-line1-empty">
                        {{ $t('validationMessages.addressLine1.requiredError') }}
                    </error-message>
                </template>
            </form-field>

            <form-field
                :value="address.line2"
                :class="$style['c-address-formField']"
                name="address-line-2"
                :label-text="$t('labels.line2')"
                is-grouped
                label-style="inline"
                @input="updateAddressDetails({ ['line2']: $event })" />
        </fieldset>

        <form-field
            :value="address.locality"
            name="address-locality"
            :label-text="$t('labels.locality')"
            :has-error="isAddressLocalityEmpty"
            @input="updateAddressDetails({ ['locality']: $event })">
            <template #error>
                <error-message
                    v-if="isAddressLocalityEmpty"
                    data-js-error-message
                    data-test-id="error-address-locality-empty">
                    {{ $t('validationMessages.locality.requiredError') }}
                </error-message>
            </template>
        </form-field>

        <form-field
            :value="address.postcode"
            name="address-postcode"
            :label-text="$t('labels.postcode')"
            :has-error="!isAddressPostcodeValid"
            @input="updateAddressDetails({ ['postcode']: $event })">
            <template #error>
                <error-message
                    v-if="isAddressPostcodeEmpty"
                    data-js-error-message
                    data-test-id="error-address-postcode-empty">
                    {{ $t('validationMessages.postcode.requiredError') }}
                </error-message>
                <error-message
                    v-else-if="!isAddressPostcodeValid"
                    data-js-error-message
                    data-test-id="error-address-postcode-type-error">
                    {{ $t('validationMessages.postcode.invalidCharError') }}
                </error-message>
            </template>
        </form-field>
    </div>
</template>

<script>
import ErrorMessage from '@justeat/f-error-message';
import '@justeat/f-error-message/dist/f-error-message.css';
import FormField from '@justeat/f-form-field';
import '@justeat/f-form-field/dist/f-form-field.css';
import { mapState, mapActions } from 'vuex';
import checkoutValidationsMixin from '../mixins/validations.mixin';
import { VALIDATIONS, VUEX_CHECKOUT_MODULE } from '../constants';

export default {
    components: { FormField, ErrorMessage },

    mixins: [checkoutValidationsMixin],

    /*
    * Provide/Inject allows nested `Address` component to inherit `Checkout`
    * validator scope, `$v`.
    */
    inject: ['$v'],

    computed: {
        ...mapState(VUEX_CHECKOUT_MODULE, [
            'address'
        ]),

        /*
        * Validation methods return true if the validation conditions
        * have not been met and the field has been `touched` by a user.
        * The $dirty boolean changes to true when the user has focused/lost
        * focus on the input field.
        */
        isAddressLine1Empty () {
            return this.isFieldEmpty(VALIDATIONS.address, 'line1');
        },

        isAddressLocalityEmpty () {
            return this.isFieldEmpty(VALIDATIONS.address, 'locality');
        },

        isAddressPostcodeEmpty () {
            return this.isFieldEmpty(VALIDATIONS.address, 'postcode');
        },

        /*
        * Checks that postcode is a valid postcode and that the field is not empty.
        */
        isAddressPostcodeValid () {
            return (!this.$v[VALIDATIONS.address].postcode.$dirty || this.$v[VALIDATIONS.address].postcode.isValidPostcode) && !this.isAddressPostcodeEmpty;
        }
    },

    methods: {
        ...mapActions(VUEX_CHECKOUT_MODULE, [
            'updateAddressDetails'
        ])
    }
};
</script>

<style lang="scss" module>
$address-colour          : $color-text;
$address-fontSize        : 'body-s';
$address-weight-bold     : $font-weight-bold;

.c-address-label {
    color: $address-colour;
    @include font-size($address-fontSize);
    font-weight: $address-weight-bold;
    margin: spacing(x2) 0 spacing();
}

.c-address-group {
    margin: spacing(x2) 0;
    padding: 0;
    border: none;
    @include font-size($address-fontSize);

    .c-address-formField {
        &:focus-within,
        &:active {
            z-index: zIndex(high);
            position: relative;
        }
    }
}

.c-address-error {
    margin-bottom: spacing(x2);
}
</style>
