<template>
    <div class="p-6 space-y-8">
      <h1 class="text-2xl font-bold">交易日志可视化</h1>
  
      <div>
        <h2 class="text-xl font-semibold mb-2">交易价格图</h2>
        <Line :data="priceChartData" :options="chartOptions" />
      </div>
  
      <div>
        <h2 class="text-xl font-semibold mb-2">盈亏柱状图</h2>
        <Bar :data="pnlChartData" :options="chartOptions" />
      </div>
  
      <div>
        <h2 class="text-xl font-semibold mb-2">交易明细表格</h2>
        <table class="w-full border">
          <thead class="bg-gray-100">
            <tr>
              <th class="border px-2 py-1">时间</th>
              <th class="border px-2 py-1">股票</th>
              <th class="border px-2 py-1">操作</th>
              <th class="border px-2 py-1">价格</th>
              <th class="border px-2 py-1">数量</th>
              <th class="border px-2 py-1">盈亏</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(trade, index) in trades" :key="index">
              <td class="border px-2 py-1">{{ trade.timestamp }}</td>
              <td class="border px-2 py-1">{{ trade.symbol }}</td>
              <td class="border px-2 py-1" :class="trade.side === 'BUY' ? 'text-green-600' : 'text-red-600'">
                {{ trade.side }}
              </td>
              <td class="border px-2 py-1">{{ trade.price }}</td>
              <td class="border px-2 py-1">{{ trade.quantity }}</td>
              <td class="border px-2 py-1">{{ trade.pnl !== null ? trade.pnl : '-' }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, computed } from 'vue';
  import { Line, Bar } from 'vue-chartjs';
  import {
    Chart as ChartJS,
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    BarElement,
    Tooltip,
    Legend
  } from 'chart.js';
  
  ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, BarElement, Tooltip, Legend);
  
  const trades = ref([]);
  
  onMounted(async () => {
    const res = await fetch('http://127.0.0.1:5000/api/trades');
    trades.value = await res.json();
  });
  
  const priceChartData = computed(() => {
    return {
      datasets: [
        {
          label: 'Buy',
          data: trades.value.filter(t => t.side === 'BUY').map(t => ({ x: t.timestamp, y: t.price })),
          borderColor: 'green',
          backgroundColor: 'green',
          showLine: false,
          pointRadius: 5
        },
        {
          label: 'Sell',
          data: trades.value.filter(t => t.side === 'SELL').map(t => ({ x: t.timestamp, y: t.price })),
          borderColor: 'red',
          backgroundColor: 'red',
          showLine: false,
          pointRadius: 5
        }
      ]
    };
  });
  
  const pnlChartData = computed(() => {
    const sells = trades.value.filter(t => t.side === 'SELL' && t.pnl !== null);
    return {
      labels: sells.map(t => `${t.symbol} @ ${t.timestamp}`),
      datasets: [
        {
          label: '盈亏',
          data: sells.map(t => t.pnl),
          backgroundColor: sells.map(t => t.pnl >= 0 ? 'green' : 'red')
        }
      ]
    };
  });
  
  const chartOptions = {
    responsive: true,
    plugins: {
      legend: {
        position: 'top'
      },
      tooltip: {
        mode: 'index'
      }
    },
    scales: {
      x: {
        type: 'category'
      },
      y: {
        beginAtZero: false
      }
    }
  };
  </script>
  
  <style scoped>
  /* 可选样式 */
  </style>
  