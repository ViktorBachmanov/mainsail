<template>
    <tr>
        <td class="icon">
            <v-icon :color="iconColor" :class="iconClass" tabindex="-1" @click="showEditDialog = true">
                {{ mdiFan }}
            </v-icon>
        </td>
        <td class="name">
            <span class="cursor-pointer" @click="showEditDialog = true">{{ formatName }}</span>
        </td>
        <td class="text-no-wrap text-center" colspan="3">
            <temperature-panel-list-item-nevermore-value
                :printer-object="printerObject"
                :object-name="objectName"
                :small="false"
                key-name="gas" />
            <temperature-panel-list-item-nevermore-value
                v-for="keyName in nevermoreValues"
                :key="keyName"
                :printer-object="printerObject"
                :object-name="objectName"
                :key-name="keyName" />
            <div v-if="rpm !== null">
                <small :class="rpmClass">{{ rpm }} RPM</small>
            </div>
        </td>
        <temperature-panel-list-item-edit
            :bool-show="showEditDialog"
            :object-name="objectName"
            :name="name"
            :format-name="formatName"
            additional-sensor-name="nevermore"
            :icon="mdiFan"
            :color="color"
            @close-dialog="showEditDialog = false" />
    </tr>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins, Prop } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import { convertName } from '@/plugins/helpers'
import { mdiFan } from '@mdi/js'
import { opacityHeaterActive, opacityHeaterInactive } from '@/store/variables'

@Component
export default class TemperaturePanelListItemNevermore extends Mixins(BaseMixin) {
    mdiFan = mdiFan

    @Prop({ type: String, required: true }) readonly objectName!: string
    @Prop({ type: Boolean, required: true }) readonly isResponsiveMobile!: boolean

    showEditDialog = false
    nevermoreValues = ['temperature', 'pressure', 'humidity']

    get printerObject() {
        return this.$store.state.printer[this.objectName] ?? {}
    }

    get name() {
        const splits = this.objectName.split(' ')
        return splits.length === 1 ? splits[0] : splits[1]
    }

    get formatName() {
        return convertName(this.name)
    }

    get color() {
        return this.$store.state.gui?.view?.tempchart?.datasetSettings?.[this.objectName]?.color ?? '#ffffff'
    }

    get iconColor() {
        // set icon color to active, if no target exists (temperature_sensors) or a heater is active
        if (this.state === null || this.state > 0) return `${this.color}${opacityHeaterActive}`

        return `${this.color}${opacityHeaterInactive}`
    }

    get iconClass() {
        const classes = ['_no-focus-style', 'cursor-pointer']

        // add icon animation, when it is a fan and state > 0
        const disableFanAnimation = this.$store.state.gui?.uiSettings.disableFanAnimation ?? false

        if (!disableFanAnimation && (this.state ?? 0) > 0) classes.push('icon-rotate')

        return classes
    }

    get state(): number | null {
        return this.printerObject.speed ?? null
    }

    get rpm() {
        const rpm = this.printerObject.rpm ?? null

        // return null when rpm doesn't exist
        if (rpm === null) return null

        return parseInt(this.printerObject.rpm)
    }

    get rpmClass() {
        if (this.rpm === 0 && (this.printerObject.speed ?? 0) > 0) return 'red--text'

        return ''
    }
}
</script>

<style scoped>
::v-deep .v-icon._no-focus-style:focus::after {
    opacity: 0 !important;
}

::v-deep .cursor-pointer {
    cursor: pointer;
}
</style>
