<template>
    <div
        data-test-id="card-component"
        :class="[
            $style['c-card'],
            (isRounded ? $style['c-card--rounded'] : ''),
            (hasOutline ? $style['c-card--outline'] : ''),
            (isPageContentWrapper ? $style['c-card--pageContentWrapper'] : '')
        ]"
    >
        <component
            :is="cardHeadingTag"
            v-if="cardHeading"
            :class="[
                $style['c-card-heading'],
                (cardHeadingPosition !== 'left' ? $style[`c-card--${cardHeadingPosition}`] : '')
            ]"
            data-test-id="card-heading"
        >
            {{ cardHeading }}
        </component>
        <slot />
    </div>
</template>

<script>
export default {
    name: 'CardComponent',
    components: {},
    props: {
        cardHeading: {
            type: String,
            default: ''
        },
        cardHeadingPosition: {
            type: String,
            default: 'left',
            validator: value => ['left', 'right', 'center'].indexOf(value) !== -1
        },
        cardHeadingTag: {
            type: String,
            default: 'h1',
            validator: value => ['h1', 'h2', 'h3', 'h4', 'h5', 'h6'].indexOf(value) !== -1
        },
        hasOutline: {
            type: Boolean,
            default: false
        },
        isPageContentWrapper: {
            type: Boolean,
            default: false
        },
        isRounded: {
            type: Boolean,
            default: false
        }
    }
};
</script>

<style lang="scss" module>

$card-bgColor                             : $color-bg--component;
$card-borderColor                         : $color-border;
$card-borderRadius                        : $border-radius;
$card-padding                             : spacing(x2);

$card--pageContentWrapper-width           : 472px; // so that it falls on our 8px spacing grid

.c-card {
    background-color: $card-bgColor;
    display: block;
    padding: $card-padding;
}

    .c-card--separated {
        margin-bottom: spacing();
    }

    .c-card--rounded {
        border-radius: $card-borderRadius;
    }

    .c-card--outline {
        border: solid 1px $card-borderColor;
    }

    // .c-card--pageContentWrapper Modifier
    // Used for displaying full page content appropriately
    // Default is fullWidth on narrow devices
    // above $card-pageContentWrapper-width, the width of the card is restricted and centred on the page
    .c-card--pageContentWrapper {
        width: 100%;
        transition: 250ms padding ease-in-out;
        padding-left: 6%;
        padding-right: 6%;
        margin: spacing(x5) 0;

        @include media('>=narrow') {
            padding-left: 10%;
            padding-right: 10%;
        }

        @include media('>=#{$card--pageContentWrapper-width}') {
            width: $card--pageContentWrapper-width;
            margin: spacing(x5) auto;
            padding-left: spacing(x10);
            padding-right: spacing(x10);
        }
    }

    .c-card-heading {
        margin-bottom: spacing(x2);
    }

    .c-card--center {
        text-align: center;
    }

    .c-card--right {
        text-align: right;
    }
</style>
