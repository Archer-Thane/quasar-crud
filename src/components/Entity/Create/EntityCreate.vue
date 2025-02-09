<template>
  <portlet v-if="defaultLayout" ref="portlet" class="entity-create">
    <template #title>
      <slot name="title">
        {{ title }}
      </slot>
    </template>
    <template #toolbar>
      <slot name="toolbar">
        <q-btn v-if="showSaveButton" flat round icon="check" @click="runNeededMethod(onSaveButton, createEntity)">
          <q-tooltip>
            ذخیره
          </q-tooltip>
        </q-btn>
        <q-btn v-if="showCloseButton" flat round icon="close" @click="runNeededMethod(onCancelButton, goToIndexView)">
          <q-tooltip>
            لغو
          </q-tooltip>
        </q-btn>
        <q-btn v-if="showExpandButton" flat round :icon="(expanded) ? 'expand_less' : 'expand_more'" @click="expanded = !expanded">
          <q-tooltip>
            <span v-if="expanded">عدم نمایش فرم</span>
            <span v-else>نمایش فرم</span>
          </q-tooltip>
        </q-btn>
      </slot>
    </template>
    <template #content>
      <q-expansion-item v-model="expanded">
        <div class="slot-wrapper">
          <slot name="before-form-builder"></slot>
        </div>
        <entity-crud-form-builder ref="formBuilder"
                                  v-model:value="inputData"
                                  :disable="false"
                                  :copy-on-click="copyOnClick"
                                  @onInputClick="onInputClick"
                                  @onCopyToClipboard="onCopyToClipboard"
        />
        <div class="slot-wrapper">
          <slot name="after-form-builder"></slot>
        </div>
        <q-inner-loading :showing="loading">
          <q-spinner-ball color="primary" size="50px" />
        </q-inner-loading>
      </q-expansion-item>
    </template>
  </portlet>
  <div v-else>
    <entity-crud-form-builder
      :key="key"
      ref="formBuilder"
      v-model:value="inputData"
      :disable="false"
      :copy-on-click="copyOnClick"
      @onInputClick="onInputClick"
      @onCopyToClipboard="onCopyToClipboard"
    >
      <template #before-form-builder>
        <div class="slot-wrapper">
          <slot name="before-form-builder"></slot>
        </div>
      </template>
      <template #after-form-builder>
        <div class="slot-wrapper">
          <slot name="after-form-builder"></slot>
        </div>
      </template>
    </entity-crud-form-builder>
  </div>

</template>

<script>
import Portlet from '../../../components/Portlet'
import EntityMixin from '../../../mixins/EntityMixin'
import { inputMixin } from 'quasar-form-builder'
import EntityCrudFormBuilder from '../EntityCrudFormBuilder'

export default {
  name: 'EntityCreate',
  components: { EntityCrudFormBuilder, Portlet },
  mixins: [
    inputMixin,
    EntityMixin
  ],
  props: {
    value: {
      default: () => [],
      type: Array
    },
    title: {
      default: '',
      type: String
    },
    api: {
      default: '',
      type: String
    },
    entityIdKeyInResponse: {
      default: 'id',
      type: String
    },
    showRouteName: {
      default: '',
      type: String
    },
    showRouteParamKey: {
      default: 'id',
      type: String
    },
    indexRouteName: {
      default: '',
      type: String
    },
    table: {
      default: () => {
        return {
          columns: [],
          data: []
        }
      },
      type: Object
    },
     defaultLayout: {
      default: true,
      type: Boolean,
    },
  },
  data () {
    return {
      expanded: true,
      loading: false
    }
  },
  methods: {
    createEntity () {
      this.loading = true
      const formData = this.getFormData()
      this.beforeSendData(formData, this.setNewInputData)
      this.$axios.post(this.api, formData, { headers: this.getHeaders() })
          .then((response) => {
            this.loading = false
            const entityId = this.getValidChainedObject(response.data, this.entityIdKeyInResponse.split('.'))
            this.afterSendData(response)
            this.$router.push({ name: this.showRouteName, params: { [this.showRouteParamKey]: entityId } })
          })
          .catch(() => {
            this.loading = false
            this.getData()
          })
    }
  }
}
</script>

<style lang="sass">
.entity-create .q-expansion-item__container .q-item
  display: none

.entity-create .slot-wrapper .q-expansion-item__container .q-item
  display: flex
</style>
