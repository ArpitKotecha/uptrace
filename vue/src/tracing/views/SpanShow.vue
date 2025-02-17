<template>
  <div>
    <v-progress-linear v-if="span.loading" absolute indeterminate />
    <SpanCard
      v-if="span.item"
      :date-range="dateRange"
      :span="span.item"
      :fluid="$vuetify.breakpoint.mdAndDown"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, computed, watch, proxyRefs } from 'vue'

// Composables
import { useTitle } from '@vueuse/core'
import { useRouter } from '@/use/router'
import { useDateRange } from '@/use/date-range'
import { useForceReload } from '@/use/force-reload'
import { useWatchAxios } from '@/use/watch-axios'

// Components
import SpanCard from '@/tracing/SpanCard.vue'

// Utilities
import { Span } from '@/models/span'

export default defineComponent({
  name: 'SpanShow',
  components: { SpanCard },

  setup() {
    const dateRange = useDateRange()
    const span = useSpan()

    watch(
      () => span.item,
      (span) => {
        if (span) {
          useTitle(span.name)
        }
      },
    )

    return { dateRange, span }
  },
})

function useSpan() {
  const { route } = useRouter()
  const { forceReloadParams } = useForceReload()

  const { loading, data } = useWatchAxios(() => {
    const { projectId, traceId, spanId } = route.value.params
    return {
      url: `/api/v1/tracing/${projectId}/traces/${traceId}/${spanId}`,
      params: forceReloadParams.value,
    }
  })

  const span = computed((): Span | undefined => {
    return data.value?.span
  })

  return proxyRefs({ loading, item: span })
}
</script>

<style lang="scss" scoped></style>
