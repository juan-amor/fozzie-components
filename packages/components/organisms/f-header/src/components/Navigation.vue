<template>
    <nav
        v-if="hasNavigationLinks"
        :class="[
            $style['c-nav'],
            $style['c-nav--global']
        ]"
        data-test-id="nav-container">
        <button
            :class="[
                'is-hidden--noJS',
                $style['c-nav-trigger'],
                $style['c-nav-toggle'],
                { [$style['c-nav-toggle--altColour']]: isAltColour },
                { [$style['is-open']]: navIsOpen }
            ]"
            :aria-expanded="navIsOpen ? 'true' : 'false'"
            :aria-label="copy.openMenuText"
            data-test-id="nav-toggle"
            type="button"
            @click="onNavToggle">
            <span :class="$style['c-nav-toggle-icon']" />
        </button>

        <input
            id="nav-trigger"
            v-model="navIsOpen"
            type="checkbox"
            :class="[
                'is-hidden',
                'is-shown--noJS',
                $style['c-nav-trigger']
            ]">

        <label
            :class="[
                'is-hidden',
                'is-shown--noJS',
                $style['c-nav-toggle'],
                { [$style['is-open']]: navIsOpen }
            ]"
            :aria-label="copy.openMenuText"
            for="nav-trigger">
            <span :class="$style['c-nav-toggle-icon']" />
        </label>

        <a
            v-if="showOffersLink && !navIsOpen"
            data-test-id="offers-iconLink"
            data-trak='{
                "trakEvent": "click",
                "category": "header",
                "action": "click - navigation",
                "label": "offers_icon"
            }'
            :href="copy.offers.url"
            :class="[
                $style['c-nav-featureLink'],
                $style['c-nav-featureLink--hideAboveMid']
            ]">
            <gift-icon
                :class="[
                    $style['c-nav-icon'],
                    $style['c-nav-icon--offers'],
                    { [$style['c-nav-icon--alt']]: isAltColour }
                ]" />
            <span class="is-visuallyHidden">
                {{ copy.offers.text }}
            </span>
        </a>

        <div
            :class="[
                $style['c-nav-container'],
                { [$style['is-visible']]: navIsOpen }
            ]">
            <ul
                :class="$style['c-nav-list']"
                data-test-id="nav-list">
                <li
                    v-if="showOffersLink"
                    :class="$style['c-nav-list-item--horizontallyAlignedAboveMid']">
                    <a
                        data-test-id="offers-link"
                        data-trak='{
                            "trakEvent": "click",
                            "category": "header",
                            "action": "click - navigation",
                            "label": "offers"
                        }'
                        :href="copy.offers.url"
                        :class="[
                            $style['c-nav-list-link'],
                            { [$style['c-nav-list-link--alt']]: isAltColour },
                            { [$style['c-nav-list-link--transparent']]: headerBackgroundTheme === 'transparent' }
                        ]">
                        <gift-icon
                            :class="[
                                $style['c-nav-icon'],
                                $style['c-nav-icon--offers'],
                                { [$style['c-nav-icon--alt']]: isAltColour }
                            ]" />
                        {{ copy.offers.text }}
                    </a>
                </li>
                <li
                    v-if="showDeliveryEnquiry"
                    :class="$style['c-nav-list-item--horizontallyAlignedAboveMid']"
                    data-test-id="delivery-enquiry">
                    <a
                        data-test-id="delivery-link"
                        :data-trak='`{
                            "trakEvent": "click",
                            "category": "engagement",
                            "action": "header",
                            "label": "${copy.deliveryEnquiry.gtm}"
                        }`'
                        :href="copy.deliveryEnquiry.url"
                        target="_blank"
                        :class="[
                            $style['c-nav-list-link'],
                            { [$style['c-nav-list-link--alt']]: isAltColour },
                            { [$style['c-nav-list-link--transparent']]: headerBackgroundTheme === 'transparent' }
                        ]">
                        <moped-icon
                            :class="[
                                $style['c-nav-icon'],
                                $style['c-nav-icon--delivery'],
                                { [$style['c-nav-icon--alt']]: isAltColour }
                            ]" />
                        {{ copy.deliveryEnquiry.text }}
                    </a>
                </li>
                <li
                    :class="[
                        $style['c-nav-list-item--horizontallyAlignedAboveMid'],
                        $style['has-sublist'], {
                            'is-hidden': !userInfo || !showLoginInfo,
                            [$style['is-open']]: userMenuIsOpen
                        }]"
                    data-test-id="user-info-icon"
                    v-on="isBelowMid ? null : { mouseover: openUserMenu, mouseleave: closeUserMenu }"
                    @keyup.esc="closeUserMenu">
                    <a
                        data-test-id="user-info-link"
                        :tabindex="isBelowMid ? -1 : 0"
                        :aria-expanded="!isBelowMid && userMenuIsOpen ? 'true' : 'false'"
                        :aria-haspopup="isBelowMid ? false : true"
                        :class="$style['c-nav-list-text']"
                        href="/"
                        @click.prevent="onNavToggle">
                        <profile-icon
                            :class="[
                                $style['c-nav-icon'],
                                $style['c-nav-icon--profile'],
                                { [$style['c-nav-icon--alt']]: isAltColour }
                            ]" />
                        <span
                            :class="[
                                $style['c-nav-list-text-sub'],
                                { [$style['c-nav-list-link--alt']]: isAltColour },
                                { [$style['c-nav-list-link--transparent']]: headerBackgroundTheme === 'transparent' }
                            ]">
                            {{ userInfo.friendlyName }}
                        </span>
                        <span
                            :class="[
                                $style['c-nav-list-text-sub'],
                                $style['u-showBelowMid']
                            ]">
                            {{ userInfo.email }}
                        </span>
                    </a>

                    <v-popover :class="$style['c-nav-popover']">
                        <user-navigation-panel
                            :is-open="navIsOpen"
                            :is-below-mid="isBelowMid"
                            :copy="copy"
                            :return-logout-url="returnLogoutUrl"
                            @activateNav="openUserMenu"
                            @deactivateNav="closeUserMenu" />
                    </v-popover>
                </li>

                <li
                    v-if="!userInfo && showLoginInfo"
                    :class="$style['c-nav-list-item--horizontallyAlignedAboveMid']"
                    data-test-id="login">
                    <a
                        :href="returnLoginUrl"
                        :data-trak='`{
                            "trakEvent": "click",
                            "category": "engagement",
                            "action": "header",
                            "label": "${copy.accountLogin.gtm}"
                        }`'
                        rel="nofollow"
                        :class="[
                            $style['c-nav-list-link'],
                            { [$style['c-nav-list-link--leftPaddingBelowMid']]: userInfo },
                            { [$style['c-nav-list-link--alt']]: isAltColour },
                            { [$style['c-nav-list-link--transparent']]: headerBackgroundTheme === 'transparent' }
                        ]"
                        data-test-id="login-link">
                        {{ copy.accountLogin.text }}
                    </a>
                </li>

                <li
                    v-if="showHelpLink"
                    :class="$style['c-nav-list-item--horizontallyAlignedAboveMid']">
                    <a
                        :href="copy.help.url"
                        :data-trak='`{
                                "trakEvent": "click",
                                "category": "engagement",
                                "action": "header",
                                "label": "${copy.help.gtm}"
                            }`'
                        :class="[
                            $style['c-nav-list-link'],
                            { [$style['c-nav-list-link--leftPaddingBelowMid']]: userInfo },
                            { [$style['c-nav-list-link--alt']]: isAltColour },
                            { [$style['c-nav-list-link--transparent']]: headerBackgroundTheme === 'transparent' }
                        ]"
                        data-test-id="help-link"
                        v-on="isBelowMid ? { blur: closeUserMenu, focus: openUserMenu } : null">
                        {{ copy.help.text }}
                    </a>
                </li>

                <li
                    v-if="userInfo && isBelowMid && showLoginInfo"
                    :class="$style['c-nav-list-item--horizontallyAlignedAboveMid']"
                    data-test-id="logout">
                    <a
                        :tabindex="navIsOpen ? 0 : -1"
                        :href="returnLogoutUrl"
                        :data-trak='`{
                                "trakEvent": "click",
                                "category": "engagement",
                                "action": "header",
                                "label": "${copy.accountLogout.gtm}"
                            }`'
                        :class="[
                            $style['c-nav-list-link'],
                            { [$style['c-nav-list-link--leftPaddingBelowMid']]: userInfo },
                            { [$style['c-nav-list-link--alt']]: isAltColour },
                            { [$style['c-nav-list-link--transparent']]: headerBackgroundTheme === 'transparent' }
                        ]"
                        v-on="isBelowMid ? { blur: closeUserMenu, focus: openUserMenu } : null">
                        {{ copy.accountLogout.text }}
                    </a>
                </li>

                <li
                    v-if="showCountrySelector"
                    data-test-id="country-selector"
                    :class="[
                        $style['c-nav-list-item--horizontallyAlignedAboveMid'],
                        $style['has-sublist'], {
                            [$style['is-open']]: countrySelectorIsOpen
                        }]"
                    v-on="isBelowMid ? null : { mouseover: openCountrySelector, mouseleave: closeCountrySelector }"
                    @keyup.esc="closeCountrySelector">
                    <button
                        type="button"
                        data-test-id="action-button-component"
                        :tabindex="isBelowMid && !navIsOpen ? -1 : 0"
                        :class="[
                            $style['c-nav-list-text'],
                            $style['c-nav-list-btn']
                        ]"
                        :aria-expanded="countrySelectorIsOpenOnDesktopView ? 'true' : 'false'"
                        :aria-haspopup="!isBelowMid"
                        :aria-label="copy.countrySelector.changeCurrentCountry"
                        @click="onCountrySelectorToggle"
                        v-on="countrySelectorIsClosedOnMobileView ? { blur: closeUserMenu, focus: openUserMenu } : null">
                        <span :class="$style['c-nav-list-iconWrapper']">
                            <flag-icon
                                data-test-id="current-flag-icon"
                                :country-code="copy.countrySelector.currentCountryKey"
                                :class="[
                                    $style['c-nav-list-icon--flag'],
                                    $style['c-nav-list-icon--flagCurrent']
                                ]" />
                        </span>
                        <span :class="$style['c-nav-list-title']">
                            {{ copy.countrySelector.selectYourCountryText }}
                        </span>
                    </button>

                    <v-popover
                        :class="[
                            $style['c-nav-popover'],
                            $style['c-nav-popover--countrySelector']
                        ]">
                        <country-selector-panel
                            :copy="copy"
                            :is-open="countrySelectorIsOpen"
                            @goBackButtonClick="closeCountrySelector"
                            @blurOnLink="closeCountrySelector"
                            @focusOnLink="openCountrySelector" />
                    </v-popover>
                </li>
            </ul>
        </div>
    </nav>
</template>

<script>
import {
    MopedIcon,
    GiftIcon,
    ProfileIcon
} from '@justeat/f-vue-icons';
import {
    axiosServices,
    windowServices
} from '@justeat/f-services';
import VPopover from '@justeat/f-popover';
import FlagIcon from './FlagIcon.vue';
import CountrySelectorPanel from './CountrySelectorPanel.vue';
import UserNavigationPanel from './UserNavigationPanel.vue';
import '@justeat/f-popover/dist/f-popover.css';
import { countries } from '../tenants';

export default {
    components: {
        CountrySelectorPanel,
        FlagIcon,
        GiftIcon,
        MopedIcon,
        ProfileIcon,
        UserNavigationPanel,
        VPopover
    },

    props: {
        copy: {
            type: Object,
            default: () => ({})
        },

        showDeliveryEnquiry: {
            type: Boolean,
            default: false
        },

        showOffersLink: {
            type: Boolean,
            default: false
        },

        showHelpLink: {
            type: Boolean,
            default: true
        },

        showLoginInfo: {
            type: Boolean,
            default: true
        },

        errorLog: {
            type: [Function, Boolean],
            default: false
        },

        userInfoProp: {
            type: [Object, Boolean],
            default: false
        },

        userInfoUrl: {
            type: String,
            default: '/api/account/details'
        },

        orderCountUrl: {
            type: String,
            default: '/analytics/ordercount'
        },

        isOrderCountSupported: {
            type: Boolean,
            default: true
        },

        headerBackgroundTheme: {
            type: String,
            default: 'white'
        },

        showCountrySelector: {
            type: Boolean,
            default: false
        }
    },

    data () {
        return {
            navIsOpen: false,
            userMenuIsOpen: false,
            currentScreenWidth: null,
            userInfo: this.userInfoProp,
            localOrderCountExpires: false,
            countrySelectorIsOpen: false,
            countries,
            midBreakpoint: 768
        };
    },

    computed: {
        isBelowMid () {
            return this.currentScreenWidth <= this.midBreakpoint;
        },

        returnUrl () {
            if (this.$route) {
                return encodeURIComponent(this.$route.path);
            }
            if (typeof document !== 'undefined') {
                return encodeURIComponent(document.location.pathname);
            }
            return encodeURIComponent('/');
        },

        returnLoginUrl () {
            return `${this.copy.accountLogin.url}?returnurl=${this.returnUrl}`;
        },

        returnLogoutUrl () {
            return `${this.copy.accountLogout.url}?returnurl=${this.returnUrl}`;
        },

        // if the order count is supported and there is no blob in local storage then return true
        isOrderCountValid () {
            return this.isOrderCountSupported && !this.getAnalyticsBlob;
        },

        isOrderCountOutOfDate () {
            const currentTime = new Date().getTime();
            return this.localOrderCountExpires < currentTime;
        },

        isAltColour () {
            return this.headerBackgroundTheme === 'highlight' || (this.headerBackgroundTheme === 'transparent' && !this.navIsOpen);
        },

        /**
         * Gets the analytic blob for order count.
         *
         * @return {object} from local storage containing analytic data.
         */
        getAnalyticsBlob () {
            return window.localStorage.getItem('je-analytics') || false;
        },

        hasNavigationLinks () {
            return this.showOffersLink ||
                this.showHelpLink ||
                this.showDeliveryEnquiry ||
                this.showLoginInfo;
        },

        countrySelectorIsClosedOnMobileView () {
            return this.isBelowMid && !this.countrySelectorIsOpen;
        },

        countrySelectorIsOpenOnDesktopView () {
            return !this.isBelowMid && this.countrySelectorIsOpen;
        }
    },

    watch: {
        userInfoProp (userInfo) {
            this.userInfo = userInfo;
        }
    },

    mounted () {
        if (this.showLoginInfo && !this.userInfo) {
            this.fetchUserInfo();
        }

        windowServices.addEvent('resize', this.onResize, 100);

        this.onResize();
    },

    destroyed () {
        windowServices.removeEvent('resize', this.onResize);
    },

    methods: {
        onNavToggle () {
            this.navIsOpen = !this.navIsOpen;
            if (this.showCountrySelector) {
                this.closeCountrySelector();
            }
            // This is added to remove the ability to scroll the page content when the mobile navigation is open
            this.handleMobileNavState();
        },

        closeUserMenu () {
            this.userMenuIsOpen = false;

            this.handleMobileNavState();
        },

        openUserMenu () {
            this.userMenuIsOpen = true;

            this.handleMobileNavState();
        },

        onCountrySelectorToggle () {
            this.countrySelectorIsOpen = !this.countrySelectorIsOpen;
        },

        closeCountrySelector () {
            this.countrySelectorIsOpen = false;
        },

        openCountrySelector () {
            this.countrySelectorIsOpen = true;
        },

        onResize () {
            this.currentScreenWidth = windowServices.getWindowWidth();
        },

        handleMobileNavState () {
            if (this.isBelowMid) {
                this.$emit('onMobileNavToggle', this.navIsOpen);

                if (typeof document !== 'undefined') {
                    document.documentElement.classList.toggle('is-navInView', this.navIsOpen);
                    document.documentElement.classList.toggle('is-navInView--noPad', this.navIsOpen && this.headerBackgroundTheme === 'transparent');
                }
            }
        },

        // If userInfoProp wasn't passed we make a call for userInfo on mounted hook
        fetchUserInfo () {
            const client = axiosServices.createClient({
                headers: {
                    credentials: 'same-origin'
                }
            });

            return client
                .get(this.userInfoUrl)
                .then(({ data }) => {
                    if (data.isAuthenticated) {
                        this.userInfo = data;

                        if (this.isOrderCountValid) {
                            this.fetchOrderCountAndSave();
                        }
                    } else {
                        this.userInfo = false;
                    }
                })
                .catch(err => {
                    if (this.errorLog) {
                        this.errorLog('Unable to get user information from "fetchUserInfo"', err);
                    }
                });
        },

        // fetches the order count for the user
        fetchOrderCountAndSave () {
            const client = axiosServices.createClient({
                headers: {
                    credentials: 'same-origin'
                }
            });

            return client
                .get(this.orderCountUrl)
                .then(data => {
                    if (data && this.isOrderCountOutOfDate) {
                        this.setAnalyticsBlob(data);
                        this.localOrderCountExpires = data.Expires;
                        this.enrichUserDataWithCount(data.Count);
                        this.pushToDataLayer(this.userInfo);
                    }
                })
                .catch(err => {
                    if (this.errorLog) {
                        this.errorLog('Unable to get order count from "fetchOrderCountAndSave', err);
                    }
                });
        },

        /**
         * Sets the analytic blob for order count.
         *
         * @param {object} containing the analytic data for order count.
         */
        setAnalyticsBlob (data) {
            window.localStorage.setItem('je-analytics', JSON.stringify(data));
        },

        // Updates the user information with the count
        enrichUserDataWithCount (count) {
            this.userInfo.orderCount = count;
        },

        // Pushes the user info to the windows data layer
        pushToDataLayer (data) {
            window.dataLayer.push(data);
        }
    }
};
</script>

<style lang="scss" module>
@import '../assets/scss/navigation.scss';

/**
 * Global Page Navigation
 * =================================
 * Styles relating to the global site navigation
 *
 */

$nav-text-size                     : 'body-l';
$nav-text-size--narrow             : 'body-s';
$nav-text-color                    : $color-link-default;

$nav-text-color--narrow            : $grey--dark;
$nav-text-color--transparent       : $white;
$nav-text-weight                   : $font-weight-bold;
$nav-text-subFont                  : $font-family-base;
$nav-icon-color                    : $color-secondary;
$nav-icon-color--transparent       : $white;
$nav-icon-color--mobileWhiteBg     : $grey--darkest;
$nav-transition-duration           : 250ms;
$nav-icon-size                     : 24px;

$nav-featureLinkIcon-width         : 28px;
$nav-featureLinkIcon-height        : 28px;

$nav-toggleIcon-left               : spacing(x2);
$nav-toggleIcon-width              : 21px;
$nav-toggleIcon-height             : 2px;
$nav-toggleIcon-borderRadius       : 1px;
$nav-toggleIcon-color              : $color-secondary;
$nav-toggleIcon-color--transparent : $white;
$nav-toggleIcon-bg                 : transparent;
$nav-toggleIcon-space              : 5px;

$nav-popover-width                 : 300px;

    // TODO: MAKE THIS NOT USE FLOATS
    // global modifier for list items horizontally aligned
    .c-nav-list-item--horizontallyAlignedAboveMid {
        @include media('>mid') {
            float: left;
        }
    }

    // Modifier to position a nav-list-item at the bottom of the mobile navigation
    // As an example, this is used on the logout link on the global site
    // Logout is in the popover list, but at the bottom of the
    .c-nav-list-item--forceLast {
        @include media('<=mid') {
            position: absolute;
            top: 100%;
            width: 100%;
        }
    }

        .c-nav-list-link,
        .c-nav-list-text {
            display: flex;
            align-items: center;
            padding: spacing(x1.5) spacing(x2);
            margin: 0;
            font-family: $nav-text-subFont;
            color: $nav-text-color--narrow;
            @include font-size($nav-text-size--narrow);
            font-weight: 300;
            text-decoration: none;
            border-bottom: 1px solid $grey--light;

            @include media('>mid') {
                @include font-size($nav-text-size);
                font-weight: $nav-text-weight;
                color: $nav-text-color;
                border-bottom: none;
                height: $header-height;
            }
        }

        .c-nav-list-link--alt {
            @include media('>mid') {
                color: $nav-text-color--transparent;
            }
        }

        .c-nav-list-link--transparent,
        .c-nav-list-link--alt {
            @include media('>mid') {
                &:hover,
                &:focus {
                    color: $nav-text-color--transparent;
                }
            }
        }

        .c-nav-list-text {
            @include media('<=mid') {
                display: block;
            }
        }

        .c-nav-list-text-sub {
            display: block;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
            max-width: 300px;

            &.u-showBelowMid {
                @include media('>mid') {
                    display: none !important;
                }
            }
        }

        .c-nav-list-link--leftPaddingBelowMid {
            @include media('<=mid') {
                padding-left: spacing(x6);
            }
        }

        .c-nav-list-btn {
            display: flex;
            align-items: center;
            justify-content: center;
            border: 0;
            background: transparent;
        }

    .has-sublist {
        // ensures the dropdown/popover is relative to the hover element, on wider views
        @include media('>mid') {
            position: relative;
            cursor: pointer;
        }
    }

    .c-nav-featureLink {
        display: block;
        width: $nav-featureLinkIcon-width;
        height: $nav-featureLinkIcon-height;

        @include media('<=mid') {
            position: absolute;
            top: 0;
            right: 0;
            width: spacing(x2) + $nav-featureLinkIcon-width + spacing(x2); // includes padding on both sides
            height: spacing(x2) + $nav-featureLinkIcon-height + spacing(x2);
            padding: spacing(x2);
        }
    }

    .c-nav-featureLink--hideAboveMid {
        @include media('>mid') {
            display: none;
        }
    }

    // Icons, such as the profile icon
    .c-nav-icon {
        float: left;
        margin-right: spacing();
        width: $nav-icon-size;
        height: $nav-icon-size;

        @include media('>mid') {
            & path {
                fill: $nav-icon-color;
            }
        }
    }

    .c-nav-icon--delivery,
    .c-nav-icon--offers,
    .c-nav-icon--profile {
        @include media('<=mid') {
            & path {
                fill: $nav-icon-color--mobileWhiteBg;
            }
        }
    }

    .c-nav-icon--alt {
        & path {
            fill: $nav-icon-color--transparent;
        }
    }

// Navigation Toggle
// Only shown at narrow widths (Hamburger Menu icon)
.c-nav-toggle {
    position: absolute;
    top: 0;
    left: 0;
    width: $nav-trigger-width;
    height: $nav-trigger-height;
    cursor: pointer;
    background-color: $nav-toggleIcon-bg;
    border: none;

    // hide on wider views
    @include media('>mid') {
        display: none;
    }
}

    // The Toggle Icon (hamburger icon)
    .c-nav-toggle-icon {
        display: block;
        top: 50%;
        left: $nav-toggleIcon-left;
        width: $nav-toggleIcon-width;
        text-indent: -150px;
        white-space: nowrap;
        transition: background-color $nav-transition-duration ease-in;

        // Apply these styles to the base icon element and its created pseudo elements
        &,
        &:before,
        &:after {
            position: absolute;
            height: $nav-toggleIcon-height;
            background-color: $nav-toggleIcon-color;
            border-radius: $nav-toggleIcon-borderRadius;

            .c-header--transparent & {
                background-color: $nav-toggleIcon-color--transparent;
            }

            .c-nav-toggle--altColour & {
                background-color: $white;
            }
        }

        &:before,
        &:after {
            content: '';
            width: 100%;
            left: 0;
            transition: all $nav-transition-duration ease-in-out;
        }
        &:before {
            top: -($nav-toggleIcon-space + $nav-toggleIcon-height);
        }
        &:after {
            top: ($nav-toggleIcon-space + $nav-toggleIcon-height);
        }
    }

    // Icon active state
    // Shown when the checkbox is checked or the `is-open` class is present.
    .c-nav-trigger:checked ~ .c-nav-toggle,
    .c-nav-toggle.is-open {
        & > .c-nav-toggle-icon {
            background-color: $nav-toggleIcon-bg;

            &:before,
            &:after {
                top: 0;
            }
            &:before {
                transform: rotate(45deg);
            }
            &:after {
                transform: rotate(-45deg);
            }
        }
    }

.c-nav-popover {
    @include media('>mid') {
        min-width: 300px;
        position: absolute;
        top: 100%;
        right: 99999px; // offscreen, so can’t ever be hovered over by default
        opacity: 0;
        z-index: -1;
        transition: opacity $nav-popover-transition-duration ease-in-out $nav-popover-transition-delay,
                    z-index 0s linear ($nav-popover-transition-delay + $nav-popover-transition-duration),
                    right 0s linear ($nav-popover-transition-delay + $nav-popover-transition-duration);

        // display the popover when our parent item is hovered(recieved class .open)
        .has-sublist.is-open & {
            opacity: 1;
            z-index: zIndex(high);
            right: 0;

            transition: opacity $nav-popover-transition-duration ease-in-out,
                        z-index 0s linear;
        }

        // tooltip arrow
        &:before {
            right: 10%;
        }
    }
}

.c-nav-popover.c-nav-popover--countrySelector {
    @include media('>mid') {
        // tooltip arrow
        &:before {
            right: 4%;
        }
    }
}

.c-nav-list-iconWrapper {
    height: $countrySelector-flag-height;
    width: $countrySelector-flag-width;

    @include media('<=mid') {
        margin-right: spacing();
    }

    @include media('>mid') {
        background-color: $white;
        width: 32px;
        height: 32px;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
    }
}

.c-nav-list-icon--flagCurrent {
    margin-right: 0;
}

.c-nav-list-title {
    width: 0;
    overflow: hidden;
    @include font-size(heading-s, true, narrow);

    @include media('<=mid') {
        width: auto;
    }
}
</style>
