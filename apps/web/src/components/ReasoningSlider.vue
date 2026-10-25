<script setup lang="ts">
/**
 * 推理强度滑块。
 * 档位数量跟随模型动态变化：levels 数组长度即档位数。
 * 拖动时实时显示当前档位名，松手（pointerup）后 emit 'commit' 由父级持久化。
 * 模型不支持思考时（levels 为空）由父级决定隐藏/置灰，本组件仅渲染 disabled 态。
 */
import { computed, ref, watch } from 'vue'
import type { ReasoningEffort } from '@/protocol/commands'

const props = defineProps<{
  levels: string[]
  modelValue: ReasoningEffort
  disabled?: boolean
}>()
const emit = defineEmits<{
  (e: 'update:modelValue', value: ReasoningEffort): void
  (e: 'commit', value: ReasoningEffort): void
}>()

const LABELS: Record<string, string> = {
  auto: '自动', low: '低', medium: '中', high: '高', xhigh: '超高',
}

/** 当前档位在 levels 中的索引；auto 视为 -1（滑块最左前的特殊位，实际放在 0）。 */
const levelIndex = computed(() => {
  if (!props.levels.length) return 0
  const idx = props.levels.indexOf(props.modelValue)
  return idx >= 0 ? idx : 0
})

/** 拖动过程中的临时索引（松手前不持久化）。 */
const dragIndex = ref(levelIndex.value)
watch(levelIndex, v => { dragIndex.value = v })

const displayIndex = computed(() => (props.disabled ? -1 : dragIndex.value))
const currentLevel = computed(() => props.levels[displayIndex.value] ?? '')
const currentLabel = computed(() => LABELS[currentLevel.value] ?? currentLevel.value)

function onInput(event: Event) {
  const idx = Number((event.target as HTMLInputElement).value)
  dragIndex.value = idx
  const level = props.levels[idx]
  if (level) emit('update:modelValue', level as ReasoningEffort)
}

function onCommit() {
  const level = props.levels[dragIndex.value]
  if (level) emit('commit', level as ReasoningEffort)
}
</script>

<template>
  <div class="reasoning-slider" :class="{ disabled }">
    <div class="slider-track">
      <input
        type="range"
        :min="0"
        :max="Math.max(0, levels.length - 1)"
        step="1"
        :value="dragIndex"
        :disabled="disabled || !levels.length"
        aria-label="推理强度"
        @input="onInput"
        @change="onCommit"
      />
      <div v-if="levels.length" class="ticks">
        <span v-for="(lv, i) in levels" :key="lv" :class="{ on: i === displayIndex }">{{ LABELS[lv] ?? lv }}</span>
      </div>
    </div>
    <div class="current">
      <span v-if="!disabled && currentLabel" class="badge">{{ currentLabel }}</span>
      <span v-else class="badge empty">不支持思考</span>
    </div>
  </div>
</template>

<style scoped>
.reasoning-slider { display: flex; flex-direction: column; gap: 4px; width: 100%; }
.slider-track { position: relative; padding: 0 2px; }
.reasoning-slider input[type='range'] {
  display: block; width: 100%; height: 26px; margin: 0;
  appearance: none; -webkit-appearance: none; background: transparent;
  cursor: pointer; touch-action: none;
}
.reasoning-slider input[type='range']:disabled { opacity: .45; cursor: not-allowed; }
.reasoning-slider input::-webkit-slider-runnable-track {
  height: 5px; border-radius: 3px; background: var(--border-strong);
}
.reasoning-slider input::-webkit-slider-thumb {
  appearance: none; -webkit-appearance: none;
  width: 20px; height: 20px; margin-top: -7.5px; border-radius: 50%;
  background: var(--blue); box-shadow: var(--shadow-1); border: none;
  transition: transform .12s;
}
.reasoning-slider input::-webkit-slider-thumb:active { transform: scale(1.12); }
.reasoning-slider input::-moz-range-track {
  height: 5px; border-radius: 3px; background: var(--border-strong);
}
.reasoning-slider input::-moz-range-thumb {
  width: 20px; height: 20px; border-radius: 50%;
  background: var(--blue); box-shadow: var(--shadow-1); border: none;
}
.ticks { display: flex; justify-content: space-between; margin-top: 2px; }
.ticks span { font-size: 10.5px; color: var(--text-3); transition: color .15s; }
.ticks span.on { color: var(--blue); font-weight: 650; }
.current { display: flex; justify-content: flex-end; margin-top: 2px; min-height: 18px; }
.badge { padding: 1px 8px; border-radius: 999px; background: var(--blue-soft); color: var(--blue); font-size: 11.5px; font-weight: 650; }
.badge.empty { background: transparent; color: var(--text-3); font-weight: 500; }
</style>
