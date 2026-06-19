<template>
  <div class="event-timeline-container">
    <div class="timeline-header">
      <h3 class="text-lg font-bold text-cyan-400">实时事件时间线</h3>
      <el-button type="danger" size="small" text @click="handleClear" v-if="store.eventTimeline.length > 0">
        清空
      </el-button>
    </div>

    <div class="filter-tabs">
      <el-radio-group v-model="activeFilter" size="small" @change="handleFilterChange">
        <el-radio-button value="all">全部</el-radio-button>
        <el-radio-button value="connection">连接</el-radio-button>
        <el-radio-button value="subscription">订阅</el-radio-button>
        <el-radio-button value="alarm">报警</el-radio-button>
        <el-radio-button value="alarm_ack">确认</el-radio-button>
      </el-radio-group>
    </div>

    <div class="timeline-stats">
      <el-tag type="success" size="small">连接: {{ connectionCount }}</el-tag>
      <el-tag type="primary" size="small">订阅: {{ subscriptionCount }}</el-tag>
      <el-tag type="danger" size="small">报警: {{ alarmCount }}</el-tag>
      <el-tag type="info" size="small">确认: {{ ackCount }}</el-tag>
    </div>

    <div class="timeline-list">
      <div v-if="filteredEvents.length === 0" class="no-events">
        <el-empty description="暂无事件记录" :image-size="60" />
      </div>

      <el-timeline v-else>
        <el-timeline-item
          v-for="event in filteredEvents"
          :key="event.id"
          :timestamp="formatTime(event.timestamp)"
          :type="getTimelineType(event.type)"
          :icon="getEventIcon(event.type)"
          size="large"
        >
          <div class="event-item" :class="`event-${event.type}`">
            <div class="event-header">
              <span class="event-type-tag">{{ getEventTypeLabel(event.type) }}</span>
              <span class="event-time">{{ formatTime(event.timestamp) }}</span>
            </div>
            <div class="event-message">{{ event.message }}</div>
            <div v-if="event.details" class="event-details">
              <el-tag v-if="event.details.severity" :type="getSeverityType(event.details.severity)" size="small" effect="dark">
                {{ getSeverityLabel(event.details.severity) }}
              </el-tag>
              <span v-if="event.details.nodeName" class="detail-item">
                节点: {{ event.details.nodeName }}
              </span>
              <span v-if="event.details.value !== undefined" class="detail-item">
                值: {{ event.details.value }}
              </span>
              <span v-if="event.details.threshold !== undefined" class="detail-item">
                阈值: {{ event.details.threshold }}
              </span>
            </div>
          </div>
        </el-timeline-item>
      </el-timeline>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useOpcuaStore } from '../store/opcua'
import {
  Monitor,
  Bell,
  CircleCheck,
  Connection,
  DataLine
} from '@element-plus/icons-vue'
import type { EventType } from '../types'

const store = useOpcuaStore()
const activeFilter = ref<string>('all')

const filteredEvents = computed(() => {
  if (activeFilter.value === 'all') {
    return store.eventTimeline
  }
  return store.eventTimeline.filter(e => e.type === activeFilter.value)
})

const connectionCount = computed(() =>
  store.eventTimeline.filter(e => e.type === 'connection').length
)

const subscriptionCount = computed(() =>
  store.eventTimeline.filter(e => e.type === 'subscription').length
)

const alarmCount = computed(() =>
  store.eventTimeline.filter(e => e.type === 'alarm').length
)

const ackCount = computed(() =>
  store.eventTimeline.filter(e => e.type === 'alarm_ack').length
)

function handleFilterChange() {
  // 切换筛选时的处理
}

function handleClear() {
  store.clearEventTimeline()
}

function getTimelineType(type: EventType): string {
  switch (type) {
    case 'connection': return 'success'
    case 'subscription': return 'primary'
    case 'alarm': return 'danger'
    case 'alarm_ack': return 'warning'
    default: return 'info'
  }
}

function getEventIcon(type: EventType) {
  switch (type) {
    case 'connection': return Connection
    case 'subscription': return DataLine
    case 'alarm': return Bell
    case 'alarm_ack': return CircleCheck
    default: return Monitor
  }
}

function getEventTypeLabel(type: EventType): string {
  switch (type) {
    case 'connection': return '连接变化'
    case 'subscription': return '订阅变化'
    case 'alarm': return '报警产生'
    case 'alarm_ack': return '报警确认'
    default: return '未知'
  }
}

function getSeverityType(severity: string): string {
  switch (severity) {
    case 'Critical': return 'danger'
    case 'High': return 'danger'
    case 'Medium': return 'warning'
    case 'Low': return 'info'
    case 'Info': return 'info'
    default: return 'info'
  }
}

function getSeverityLabel(severity: string): string {
  switch (severity) {
    case 'Critical': return '严重'
    case 'High': return '高'
    case 'Medium': return '中'
    case 'Low': return '低'
    case 'Info': return '信息'
    default: return '未知'
  }
}

function formatTime(timestamp: number): string {
  return new Date(timestamp).toLocaleString('zh-CN', { hour12: false })
}
</script>

<style scoped>
.event-timeline-container {
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 12px;
  background: rgba(30, 41, 59, 0.6);
}

.timeline-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;
}

.filter-tabs {
  margin-bottom: 12px;
}

:deep(.el-radio-button__inner) {
  background: rgba(30, 41, 59, 0.8);
  border-color: rgba(71, 85, 105, 0.5);
  color: #94a3b8;
}

:deep(.el-radio-button__orig-radio:checked + .el-radio-button__inner) {
  background: #06b6d4;
  border-color: #06b6d4;
  color: white;
}

.timeline-stats {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
  flex-wrap: wrap;
}

.timeline-list {
  flex: 1;
  overflow-y: auto;
}

.event-item {
  background: rgba(30, 41, 59, 0.8);
  border: 1px solid rgba(71, 85, 105, 0.5);
  border-radius: 6px;
  padding: 10px 12px;
}

.event-connection {
  border-left: 3px solid #10b981;
}

.event-subscription {
  border-left: 3px solid #3b82f6;
}

.event-alarm {
  border-left: 3px solid #ef4444;
  background: rgba(239, 68, 68, 0.05);
}

.event-alarm_ack {
  border-left: 3px solid #f59e0b;
}

.event-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 6px;
}

.event-type-tag {
  font-size: 11px;
  font-weight: 600;
  color: #64748b;
  text-transform: uppercase;
}

.event-time {
  font-size: 11px;
  color: #64748b;
  font-family: monospace;
}

.event-message {
  font-size: 13px;
  color: #e2e8f0;
  margin-bottom: 8px;
  line-height: 1.4;
}

.event-details {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  align-items: center;
}

.detail-item {
  font-size: 11px;
  color: #94a3b8;
}

.no-events {
  display: flex;
  justify-content: center;
  padding-top: 40px;
}

:deep(.el-timeline-item__timestamp) {
  color: #64748b;
  font-size: 11px;
  font-family: monospace;
}

:deep(.el-timeline-item__content) {
  color: #e2e8f0;
}
</style>
