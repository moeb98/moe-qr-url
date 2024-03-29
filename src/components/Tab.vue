<template>
  <section v-show="isActive"
           :aria-hidden="! isActive"
           :class="panelClass"
           :id="computedId"
           role="tabpanel"
           ref="tab"
  >
    <slot/>
  </section>
</template>

<script>
import {inject, watch, ref, onBeforeMount} from 'vue';

export default {
  name: 'Tab',
  props: {
    panelClass: {
      type: String,
      default: 'tabs-component-panel'
    },
    id: {
      type: String,
      default: null
    },
    name: {
      type: String,
      required: true
    },
    prefix: {
      type: String,
      default: ''
    },
    suffix: {
      type: String,
      default: ''
    },
    isDisabled: {
      type: Boolean,
      default: false
    },
  },

  setup(props) {
    const isActive = ref(false)

    const tabsProvider = inject('tabsProvider')

    const header = props.prefix + props.name + props.suffix
    const computedId = props.id ? props.id : props.name.toLowerCase().replace(/ /g, '-')
    const hash = '#' + (!props.isDisabled ? computedId : '')

    watch(
        () => tabsProvider.activeTabHash,
        () => {
          isActive.value = hash === tabsProvider.activeTabHash
        }
    )

    onBeforeMount(() => {
      tabsProvider.tabs.push({
        name: props.name,
        header: header,
        isDisabled: props.isDisabled,
        hash: hash,
        index: tabsProvider.tabs.length
      })
    })

    return {
      header,
      computedId,
      hash,
      isActive
    }
  }
};
</script>