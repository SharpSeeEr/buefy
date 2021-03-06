<template>
    <div class="b-clockpicker" :class="type">
        <b-dropdown
            v-if="!isMobile || inline"
            ref="dropdown"
            :position="position"
            :disabled="disabled"
            :inline="inline">
            <b-input
                v-if="!inline"
                ref="input"
                slot="trigger"
                autocomplete="off"
                :value="formatValue(dateSelected)"
                :placeholder="placeholder"
                :size="size"
                :icon="icon"
                :icon-pack="iconPack"
                :loading="loading"
                :disabled="disabled"
                :readonly="readonly"
                :rounded="rounded"
                v-bind="$attrs"
                @change.native="onChange($event.target.value)"
                @focus="$emit('focus', $event)"
                @blur="$emit('blur', $event) && checkHtml5Validity()"/>

            <div
                class="card"
                style="height: auto;"
                :disabled="disabled"
                custom>
                <header v-if="inline" class="card-header">
                    <div class="b-clockpicker-header card-header-title">
                        <div class="b-clockpicker-time">
                            <span
                                class="b-clockpicker-btn"
                                :class="{ active: isSelectingHour }"
                                @click="isSelectingHour = true">{{ hoursDisplay }}</span>
                            <span>:</span>
                            <span
                                class="b-clockpicker-btn"
                                :class="{ active: !isSelectingHour }"
                                @click="isSelectingHour = false">{{ minutesDisplay }}</span>
                        </div>
                        <div v-if="!isHourFormat24" class="b-clockpicker-period">
                            <div
                                class="b-clockpicker-btn"
                                :class="{ active: meridienSelected == AM }"
                                @click="onMeridienClick(AM)">am</div>
                            <div
                                class="b-clockpicker-btn"
                                :class="{ active: meridienSelected == PM }"
                                @click="onMeridienClick(PM)">pm</div>
                        </div>
                    </div>
                </header>
                <div class="card-content">
                    <div
                        class="b-clockpicker-body"
                        :style="{ width: faceSize + 'px', height: faceSize + 'px' }">
                        <div v-if="!inline" class="b-clockpicker-time">
                            <div
                                class="b-clockpicker-btn"
                                :class="{ active: isSelectingHour }"
                                @click="isSelectingHour = true">Hours</div>
                            <span
                                class="b-clockpicker-btn"
                                :class="{ active: !isSelectingHour }"
                                @click="isSelectingHour = false">Min</span>
                        </div>
                        <div v-if="!isHourFormat24 && !inline" class="b-clockpicker-period">
                            <div
                                class="b-clockpicker-btn"
                                :class="{ active: meridienSelected == AM }"
                                @click="onMeridienClick(AM)">{{ AM }}</div>
                            <div
                                class="b-clockpicker-btn"
                                :class="{ active: meridienSelected == PM }"
                                @click="onMeridienClick(PM)">{{ PM }}</div>
                        </div>
                        <b-clockpicker-face
                            :picker-size="faceSize"
                            :min="minFaceValue"
                            :max="maxFaceValue"
                            :face-numbers="isSelectingHour ? hours : minutes"
                            :disabled-values="faceDisabledValues()"
                            :double="isSelectingHour && isHourFormat24"
                            :value="isSelectingHour ? hoursSelected : minutesSelected"
                            @input="onClockInput"
                            @change="onClockChange" />
                    </div>
                </div>
                <footer
                    v-if="$slots.default !== undefined && $slots.default.length"
                    class="b-clockpicker-footer card-footer">
                    <slot/>
                </footer>
            </div>
        </b-dropdown>
    </div>

</template>

<script>
import TimepickerMixin from '../../utils/TimepickerMixin'
import ClockpickerFace from './ClockpickerFace'
import { Dropdown, DropdownItem } from '../dropdown'
import Input from '../input'
import Field from '../field'
import Icon from '../icon'

const outerPadding = 12

export default {
    name: 'BClockpicker',
    components: {
        [ClockpickerFace.name]: ClockpickerFace,
        [Input.name]: Input,
        [Field.name]: Field,
        [Icon.name]: Icon,
        [Dropdown.name]: Dropdown,
        [DropdownItem.name]: DropdownItem
    },
    mixins: [TimepickerMixin],
    props: {
        pickerSize: {
            type: Number,
            default: 290
        },
        hourFormat: {
            type: String,
            default: '12',
            validator: (value) => {
                return value === '24' || value === '12'
            }
        },
        incrementMinutes: {
            type: Number,
            default: 5
        },
        autoSwitch: {
            type: Boolean,
            default: true
        },
        type: {
            type: String,
            default: 'is-primary'
        }
    },
    data() {
        return {
            isSelectingHour: true,
            isDragging: false
        }
    },
    computed: {
        hoursDisplay() {
            if (this.hoursSelected == null) return '--'
            if (this.isHourFormat24) return this.pad(this.hoursSelected)

            let display = this.hoursSelected
            if (this.meridienSelected === this.PM) display -= 12
            if (display === 0) display = 12
            return display
        },
        minutesDisplay() {
            return this.minutesSelected == null ? '--' : this.pad(this.minutesSelected)
        },
        minFaceValue() {
            return this.isSelectingHour &&
                !this.isHourFormat24 &&
                this.meridienSelected === this.PM ? 12 : 0
        },
        maxFaceValue() {
            return this.isSelectingHour
                ? (!this.isHourFormat24 && this.meridienSelected === this.AM ? 11 : 23)
                : 59
        },
        faceFormatter() {
            return this.isSelectingHour && !this.isHourFormat24
                ? (val) => val
                : this.formatNumber
        },
        faceSize() {
            return this.pickerSize - (outerPadding * 2)
        }
    },
    methods: {
        onClockInput(value) {
            if (this.isSelectingHour) {
                this.hoursSelected = value
                this.onHoursChange(value)
            } else {
                this.minutesSelected = value
                this.onMinutesChange(value)
            }
        },
        onClockChange(value) {
            if (this.autoSwitch && this.isSelectingHour) {
                this.isSelectingHour = !this.isSelectingHour
            }
        },
        onMeridienClick(value) {
            if (this.meridienSelected !== value) {
                this.meridienSelected = value
                this.onMeridienChange(value)
            }
        },
        faceDisabledValues() {
            return this.isSelectingHour ? this.isHourDisabled : this.isMinuteDisabled
        },
        pad(value) {
            return (value < 10 ? '0' : '') + value
        }
    },
    created() {
        this.incrementMinutes = 5
    }
}
</script>

<style>

</style>
