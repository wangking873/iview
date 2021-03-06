<template>
    <Dropdown trigger="click" :transfer="transfer" :placement="placement">
        <div :class="wrapClasses">
            <i class="ivu-icon ivu-icon-arrow-down-b ivu-input-icon ivu-input-icon-normal"></i>
            <div :class="inputClasses">
                <div :class="[prefixCls + '-color']" style="background-color: rgb(32, 160, 255);"></div>
            </div>
        </div>
        <Dropdown-menu slot="list">
            <div :class="[prefixCls + '-picker']">
                <div :class="[prefixCls + '-picker-panel']">
                    <Saturation v-model="saturationColors" @change="childChange"></Saturation>
                </div>
                <div :class="[prefixCls + '-picker-hue-slider']">
                    <Hue v-model="saturationColors" @change="childChange"></Hue>
                </div>
                <div v-if="alpha" :class="[prefixCls + '-picker-alpha-slider']">
                    <Alpha v-model="saturationColors" @change="childChange"></Alpha>
                </div>
                <recommend-colors v-if="colors.length" :list="colors" :class="[prefixCls + '-picker-colors']"></recommend-colors>
                <recommend-colors v-if="!colors.length && recommend" :list="recommendedColor" :class="[prefixCls + '-picker-colors']"></recommend-colors>
                <Confirm></Confirm>
            </div>
        </Dropdown-menu>
    </Dropdown>
</template>
<script>
    import tinycolor from 'tinycolor2';

    import Dropdown from '../dropdown/dropdown.vue';
    import DropdownMenu from '../dropdown/dropdown-menu.vue';
    import RecommendColors from './recommend-colors.vue';
    import Confirm from '../date-picker/base/confirm.vue';

    import Saturation from './saturation.vue';
    import Hue from './hue.vue';
    import Alpha from './alpha.vue';

    import { oneOf } from '../../utils/assist';

    const prefixCls = 'ivu-color-picker';
    const inputPrefixCls = 'ivu-input';

    function _colorChange (data, oldHue) {
        const alpha = data && data.a;
        let color;

        // hsl is better than hex between conversions
        if (data && data.hsl) {
            color = tinycolor(data.hsl);
        } else if (data && data.hex && data.hex.length > 0) {
            color = tinycolor(data.hex);
        } else {
            color = tinycolor(data);
        }

        if (color && (color._a === undefined || color._a === null)) {
            color.setAlpha(alpha || 1);
        }

        const hsl = color.toHsl();
        const hsv = color.toHsv();

        if (hsl.s === 0) {
            hsv.h = hsl.h = data.h || (data.hsl && data.hsl.h) || oldHue || 0;
        }

        // when the hsv.v is less than 0.0164 (base on test)
        // because of possible loss of precision
        // the result of hue and saturation would be miscalculated
        if (hsv.v < 0.0164) {
            hsv.h = data.h || (data.hsv && data.hsv.h) || 0;
            hsv.s = data.s || (data.hsv && data.hsv.s) || 0;
        }

        if (hsl.l < 0.01) {
            hsl.h = data.h || (data.hsl && data.hsl.h) || 0;
            hsl.s = data.s || (data.hsl && data.hsl.s) || 0;
        }

        return {
            hsl: hsl,
            hex: color.toHexString().toUpperCase(),
            rgba: color.toRgb(),
            hsv: hsv,
            oldHue: data.h || oldHue || hsl.h,
            source: data.source,
            a: data.a || color.getAlpha()
        };
    }

    export default {
        name: 'ColorPicker',
        components: { Dropdown, DropdownMenu, Confirm, RecommendColors, Saturation, Hue, Alpha },
        props: {
            value: {
                type: Object
            },
            alpha: {
                type: Boolean,
                default: false
            },
            recommend: {
                type: Boolean,
                default: false
            },
            format: {
                validator (value) {
                    return oneOf(value, ['hsl', 'hsv', 'hex', 'rgb']);
                }
            },
            colors: {
                type: Array,
                default () {
                    return [];
                }
            },
            disabled: {
                type: Boolean,
                default: false
            },
            size: {
                validator (value) {
                    return oneOf(value, ['small', 'large', 'default']);
                }
            },
            placement: {
                validator (value) {
                    return oneOf(value, ['top', 'top-start', 'top-end', 'bottom', 'bottom-start', 'bottom-end', 'left', 'left-start', 'left-end', 'right', 'right-start', 'right-end']);
                },
                default: 'bottom'
            },
            transfer: {
                type: Boolean,
                default: false
            }
        },
        data () {
            return {
                val: _colorChange(this.value),
                prefixCls: prefixCls,
                recommendedColor: [
                    '#2d8cf0',
                    '#19be6b',
                    '#ff9900',
                    '#ed3f14',
                    '#00b5ff',
                    '#19c919',
                    '#f9e31c',
                    '#ea1a1a',
                    '#9b1dea',
                    '#00c2b1',
                    '#ac7a33',
                    '#1d35ea',
                    '#42bd82',
                    '#f16b62',
                    '#ea4ca3',
                    '#0d94aa',
                    '#febd79',
                    '#3b90fc',
                    '#000000',
                    '#ffffff'
                ]
            };
        },
        computed: {
            saturationColors: {
                get () {
                    return this.val;
                },
                set (newVal) {
                    this.val = newVal;
                    this.$emit('input', newVal);
                }
            },
            wrapClasses () {
                return [
                    `${prefixCls}-rel`,
                    `${prefixCls}-${this.size}`,
                    `${inputPrefixCls}-wrapper`,
                    `${inputPrefixCls}-wrapper-${this.size}`
                ];
            },
            inputClasses () {
                return [
                    `${prefixCls}-input`,
                    `${inputPrefixCls}`,
                    `${inputPrefixCls}-${this.size}`,
                    {
                        [`${inputPrefixCls}-disabled`]: this.disabled
                    }
                ];
            }
        },
        watch: {
            value (newVal) {
                this.val = _colorChange(newVal);
            }
        },
        methods: {
            childChange (data) {
                this.colorChange(data);
            },
            colorChange (data, oldHue) {
                this.oldHue = this.saturationColors.hsl.h;
                this.saturationColors = _colorChange(data, oldHue || this.oldHue);
            },
            isValidHex (hex) {
                return tinycolor(hex).isValid();
            },
            simpleCheckForValidColor (data) {
                const keysToCheck = ['r', 'g', 'b', 'a', 'h', 's', 'l', 'v'];
                let checked = 0;
                let passed = 0;

                for (let i = 0; i < keysToCheck.length; i++) {
                    const letter = keysToCheck[i];
                    if (data[letter]) {
                        checked++;
                        if (!isNaN(data[letter])) {
                            passed++;
                        }
                    }
                }

                if (checked === passed) {
                    return data;
                }
            }
        }
    };
</script>