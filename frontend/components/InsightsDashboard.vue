<template>
  <div class="mb-8 space-y-6">
    <!-- Summary Stats Cards -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
      <!-- Total Projects -->
      <div class="bg-gray-800 border border-violet-600 rounded-lg p-6 shadow-lg">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-gray-400 text-sm">Total Projects</p>
            <p class="text-3xl font-bold text-violet-300 mt-2">{{ stats.totalProjects }}</p>
          </div>
          <Icon name="mdi:package-variant" class="text-violet-600 text-4xl" />
        </div>
      </div>

      <!-- GitHub Projects -->
      <div class="bg-gray-800 border border-violet-600 rounded-lg p-6 shadow-lg">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-gray-400 text-sm">GitHub Projects</p>
            <p class="text-3xl font-bold text-violet-300 mt-2">{{ stats.githubProjects }}</p>
            <p class="text-xs text-gray-500 mt-1">{{ stats.githubPercentage }}%</p>
          </div>
          <Icon name="mdi:github" class="text-violet-600 text-4xl" />
        </div>
      </div>

      <!-- Total Contributors -->
      <div class="bg-gray-800 border border-violet-600 rounded-lg p-6 shadow-lg">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-gray-400 text-sm">Total Contributors</p>
            <p class="text-3xl font-bold text-violet-300 mt-2">{{ stats.totalContributors }}</p>
          </div>
          <Icon name="mdi:account-group" class="text-violet-600 text-4xl" />
        </div>
      </div>

      <!-- Total Open Issues -->
      <div class="bg-gray-800 border border-violet-600 rounded-lg p-6 shadow-lg">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-gray-400 text-sm">Total Open Issues</p>
            <p class="text-3xl font-bold text-violet-300 mt-2">{{ stats.totalOpenIssues }}</p>
          </div>
          <Icon name="mdi:alert-circle" class="text-violet-600 text-4xl" />
        </div>
      </div>
    </div>

    <!-- Charts and Top Projects -->
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
      <!-- GitHub vs Non-GitHub Pie Chart -->
      <div class="bg-gray-800 border border-violet-600 rounded-lg p-6 shadow-lg">
        <h3 class="text-xl font-bold text-violet-300 mb-4">Project Distribution</h3>
        <div class="h-64 flex items-center justify-center">
          <Pie :data="projectDistributionData" :options="chartOptions" />
        </div>
      </div>

      <!-- License Distribution Bar Chart -->
      <div class="bg-gray-800 border border-violet-600 rounded-lg p-6 shadow-lg">
        <h3 class="text-xl font-bold text-violet-300 mb-4">License Distribution</h3>
        <div class="h-64">
          <Bar :data="licenseDistributionData" :options="barChartOptions" />
        </div>
      </div>
    </div>

    <!-- Top Projects -->
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
      <!-- Most Contributors -->
      <div class="bg-gray-800 border border-violet-600 rounded-lg p-6 shadow-lg">
        <h3 class="text-xl font-bold text-violet-300 mb-4 flex items-center gap-2">
          <Icon name="mdi:trophy" class="text-yellow-500" />
          Most Contributors
        </h3>
        <div v-if="stats.mostContributors" class="space-y-2">
          <div class="flex justify-between items-center">
            <span class="text-violet-200">{{ stats.mostContributors.name }}</span>
            <span class="text-violet-400 font-bold">{{ stats.mostContributors.contributors }} contributors</span>
          </div>
        </div>
        <p v-else class="text-gray-500">No data available</p>
      </div>

      <!-- Least Contributors -->
      <div class="bg-gray-800 border border-violet-600 rounded-lg p-6 shadow-lg">
        <h3 class="text-xl font-bold text-violet-300 mb-4 flex items-center gap-2">
          <Icon name="mdi:account-alert" class="text-orange-500" />
          Least Contributors
        </h3>
        <div v-if="stats.leastContributors" class="space-y-2">
          <div class="flex justify-between items-center">
            <span class="text-violet-200">{{ stats.leastContributors.name }}</span>
            <span class="text-violet-400 font-bold">{{ stats.leastContributors.contributors }} contributors</span>
          </div>
        </div>
        <p v-else class="text-gray-500">No data available</p>
      </div>
    </div>

    <!-- Top 5 Open Issues -->
    <div class="bg-gray-800 border border-violet-600 rounded-lg p-6 shadow-lg">
      <h3 class="text-xl font-bold text-violet-300 mb-4 flex gap-2">
        <Icon name="mdi:alert-circle-outline" class="text-red-500" />
        Top Projects by Issues
      </h3>
      <div v-if="stats.topOpenIssues.length > 0" class="space-y-3">
        <div v-for="(project, index) in stats.topOpenIssues" :key="project.name"
             class="flex items-center justify-between p-3 bg-gray-700 rounded-lg">
          <div class="flex items-center gap-3">
            <span class="text-violet-400 font-bold text-lg">{{ index + 1 }}</span>
            <span class="text-violet-200">{{ project.name }}</span>
          </div>
          <span class="text-red-400 font-bold">{{ project.openIssues }} open</span>
        </div>
      </div>
      <p v-else class="text-gray-500">No data available</p>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { Pie, Bar } from 'vue-chartjs'
import {
  Chart as ChartJS,
  ArcElement,
  CategoryScale,
  LinearScale,
  BarElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'

// Register Chart.js components
ChartJS.register(ArcElement, CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend)

const props = defineProps({
  scanResult: {
    type: Object,
    required: true
  }
})

const stats = computed(() => {
  const objects = props.scanResult.objects || []

  const totalProjects = objects.length
  const githubProjects = objects.filter(dep => dep.github).length
  const githubPercentage = totalProjects > 0 ? Math.round((githubProjects / totalProjects) * 100) : 0

  const totalContributors = objects.reduce((sum, dep) => {
    return sum + (dep.github?.no_of_contributors || 0)
  }, 0)

  const totalOpenIssues = objects.reduce((sum, dep) => {
    return sum + (dep.github?.open_issues || 0)
  }, 0)

  // Most and least contributors
  const projectsWithGithub = objects.filter(dep => dep.github)

  const mostContributors = projectsWithGithub.length > 0
    ? projectsWithGithub.reduce((max, dep) => {
        return (dep.github.no_of_contributors > (max.github?.no_of_contributors || 0)) ? dep : max
      })
    : null

  const leastContributors = projectsWithGithub.length > 0
    ? projectsWithGithub.reduce((min, dep) => {
        return (dep.github.no_of_contributors < (min.github?.no_of_contributors || Infinity)) ? dep : min
      })
    : null

  const topOpenIssues = [...projectsWithGithub]
    .sort((a, b) => (b.github?.open_issues || 0) - (a.github?.open_issues || 0))
    .slice(0, 5)
    .map(dep => ({
      name: dep.name,
      openIssues: dep.github.open_issues
    }))

  return {
    totalProjects,
    githubProjects,
    githubPercentage,
    totalContributors,
    totalOpenIssues,
    mostContributors: mostContributors ? {
      name: mostContributors.name,
      contributors: mostContributors.github.no_of_contributors
    } : null,
    leastContributors: leastContributors ? {
      name: leastContributors.name,
      contributors: leastContributors.github.no_of_contributors
    } : null,
    topOpenIssues
  }
})

// Project distribution pie chart data
const projectDistributionData = computed(() => ({
  labels: ['GitHub Projects', 'Non-GitHub Projects'],
  datasets: [{
    data: [stats.value.githubProjects, stats.value.totalProjects - stats.value.githubProjects],
    backgroundColor: ['#8900CA', '#4B5563'],
    borderColor: ['#A855F7', '#6B7280'],
    borderWidth: 2
  }]
}))

// License distribution bar chart data
const licenseDistributionData = computed(() => {
  const objects = props.scanResult.objects || []
  const licenseCounts = {}

  objects.forEach(dep => {
    const license = dep.license || 'Unknown'
    licenseCounts[license] = (licenseCounts[license] || 0) + 1
  })

  const sortedLicenses = Object.entries(licenseCounts)
    .sort((a, b) => b[1] - a[1])
    .slice(0, 10)

  return {
    labels: sortedLicenses.map(([license]) => license),
    datasets: [{
      label: 'Number of Projects',
      data: sortedLicenses.map(([, count]) => count),
      backgroundColor: '#8900CA',
      borderColor: '#A855F7',
      borderWidth: 1
    }]
  }
})

// Chart options
const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      position: 'bottom',
      labels: {
        color: '#D8B4FE',
        padding: 15,
        font: {
          size: 12
        }
      }
    },
    tooltip: {
      backgroundColor: '#1F2937',
      titleColor: '#D8B4FE',
      bodyColor: '#E9D5FF',
      borderColor: '#8900CA',
      borderWidth: 1
    }
  }
}

const barChartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      display: false
    },
    tooltip: {
      backgroundColor: '#1F2937',
      titleColor: '#D8B4FE',
      bodyColor: '#E9D5FF',
      borderColor: '#8900CA',
      borderWidth: 1
    }
  },
  scales: {
    x: {
      ticks: {
        color: '#D8B4FE',
        font: {
          size: 11
        }
      },
      grid: {
        color: '#374151'
      }
    },
    y: {
      ticks: {
        color: '#D8B4FE',
        font: {
          size: 11
        }
      },
      grid: {
        color: '#374151'
      }
    }
  }
}
</script>
