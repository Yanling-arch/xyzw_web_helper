<template>
  <MyCard>
    <!-- 卡片标题插槽 -->
    <template #title>深海灯神模拟器</template>
    <!-- 输入框和按钮 -->
    <div class="flex items-center space-x-2 mb-2">
      <span>模拟次数：</span>
      <n-input-number v-model:value="simCount" :min="1" size="small" />
      <n-button type="primary" @click="runSimulation">开始模拟</n-button>
    </div>
    <!-- 模拟结果显示表格 -->
    <div v-if="result">
      <table class="table-auto border border-gray-200">
        <tr class="bg-gray-100">
          <th class="px-2 py-1 border">胜利次数</th>
          <th class="px-2 py-1 border">失败次数</th>
          <th class="px-2 py-1 border">胜率</th>
        </tr>
        <tr>
          <td class="px-2 py-1 border">{{ result.wins }}</td>
          <td class="px-2 py-1 border">{{ result.losses }}</td>
          <td class="px-2 py-1 border">{{ result.winRate }}</td>
        </tr>
      </table>
    </div>
  </MyCard>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useTokenStore } from '@/stores/tokenStore'
import MyCard from '@/components/Common/MyCard.vue'
import { NInputNumber, NButton } from 'naive-ui'

// 模拟次数，初始为100
const simCount = ref(100)
// 存储模拟结果：wins, losses, winRate
const result = ref<{wins: number, losses: number, winRate: string} | null>(null)

// 获取当前选中 token 的队伍信息
const tokenStore = useTokenStore()
const teamInfo = computed(() => {
  const preset = (tokenStore as any).gameData?.presetTeam
  if (!preset) return null
  // 参照 TeamFormation 组件的逻辑，找到当前使用的阵容
  const root = (preset as any).presetTeamInfo ?? preset
  const teamId = (root.useTeamId as number) ?? 1
  const teamsObj = (root.presetTeamInfo?.teams as any) ?? (root.teams as any)
  return teamsObj?.[teamId]?.teamInfo ?? null
})

// 简易战斗模拟函数：根据英雄等级、红孔数量、装备数量随机判定胜负
function runSimulation() {
  const heroes = teamInfo.value ? Object.values(teamInfo.value) : []
  let wins = 0
  for (let i = 0; i < simCount.value; i++) {
    // 计算当前阵容战力：简单地将每个英雄的等级、红孔和装备数累加
    let score = 0
    heroes.forEach((h: any) => {
      score += (h.level || 0) + (h.redSlot || 0) + (h.equipmentList?.length || 0)
    })
    // 以 score 相对 100 的概率判定胜利
    if (Math.random() < score / (score + 100)) {
      wins++
    }
  }
  const losses = simCount.value - wins
  const winRate = Math.round((wins / simCount.value) * 100) + '%'
  result.value = { wins, losses, winRate }
}
</script>

<style scoped>
/* 简单样式：例如使表格和 MyCard 风格一致 */
.table-auto { width: 100%; border-collapse: collapse; }
.border { border: 1px solid #ddd; }
.bg-gray-100 { background-color: #f7fafc; }
.px-2 { padding-left: 0.5rem; padding-right: 0.5rem; }
.py-1 { padding-top: 0.25rem; padding-bottom: 0.25rem; }
.mb-2 { margin-bottom: 0.5rem; }
.flex { display: flex; }
.items-center { align-items: center; }
.space-x-2 > :not(template) ~ :not(template) { margin-left: 0.5rem; }
</style>
