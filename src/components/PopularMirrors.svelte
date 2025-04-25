<script>
  import { onMount } from "svelte";
  import { Toaster, createToaster, Progress } from "@skeletonlabs/skeleton-svelte";
  import { Download, Star, TrendingUp, Award, Users, ArrowUp, ArrowDown } from "lucide-svelte";

  // 接收服务端渲染的初始数据
  export let initialMirrors = [];
  export let initialError = null;

  // 状态变量
  let mirrors = initialMirrors;
  let loading = initialMirrors.length === 0 && !initialError;
  let error = initialError;
  let timeRange = "month"; // 'week', 'month', 'year'

  // Toast通知
  const toaster = createToaster();

  // 获取镜像数据
  async function fetchMirrors() {
    try {
      loading = true;
      const response = await fetch("http://localhost:8082/static/tunasync.json");

      if (!response.ok) {
        throw new Error(`HTTP错误: ${response.status}`);
      }

      const data = await response.json();

      mirrors = Object.entries(data).map(([name, info]) => ({
        name,
        ...info,
        // 格式化最后更新时间
        lastUpdated: new Date(info.last_update_ts * 1000).toLocaleString("zh-CN"),
        // 添加模拟的下载量数据
        downloads: Math.floor(Math.random() * 10000),
        // 模拟周下载量
        weeklyDownloads: Math.floor(Math.random() * 2000),
        // 模拟月下载量
        monthlyDownloads: Math.floor(Math.random() * 8000),
        // 模拟年下载量
        yearlyDownloads: Math.floor(Math.random() * 50000),
        // 模拟变化趋势（正值表示上升，负值表示下降）
        trend: Math.floor(Math.random() * 200) - 100
      }));

      // 根据选定的时间范围排序
      sortMirrorsByTimeRange();

      error = null;
    } catch (err) {
      error = err.message;
      toaster.error({
        title: "加载失败",
        description: `无法获取镜像列表: ${err.message}`,
        timeout: 5000
      });
    } finally {
      loading = false;
    }
  }

  // 根据时间范围排序镜像
  function sortMirrorsByTimeRange() {
    if (timeRange === "week") {
      mirrors.sort((a, b) => b.weeklyDownloads - a.weeklyDownloads);
    } else if (timeRange === "month") {
      mirrors.sort((a, b) => b.monthlyDownloads - a.monthlyDownloads);
    } else if (timeRange === "year") {
      mirrors.sort((a, b) => b.yearlyDownloads - a.yearlyDownloads);
    }
  }

  // 格式化大数字
  function formatNumber(num) {
    if (num >= 1000000) {
      return (num / 1000000).toFixed(1) + "M";
    } else if (num >= 1000) {
      return (num / 1000).toFixed(1) + "K";
    }
    return num + "";
  }

  // 获取状态对应的样式
  function getStatusBadgeClass(status) {
    switch (status) {
      case "success":
        return "badge-success";
      case "failed":
        return "badge-error";
      case "syncing":
        return "badge-warning";
      default:
        return "badge-surface";
    }
  }

  // 获取趋势图标和颜色
  function getTrendInfo(trend) {
    if (trend > 0) {
      return { icon: ArrowUp, color: "text-success-500", text: `上升 ${trend}%` };
    } else if (trend < 0) {
      return { icon: ArrowDown, color: "text-error-500", text: `下降 ${Math.abs(trend)}%` };
    } else {
      return { icon: null, color: "", text: "无变化" };
    }
  }

  // 当时间范围变化时重新排序
  $: {
    if (mirrors.length > 0) {
      sortMirrorsByTimeRange();
    }
  }

  onMount(() => {
    // 如果没有初始数据或有错误时，才在客户端请求数据
    if (initialMirrors.length === 0 && !initialError) {
      fetchMirrors();
    }
  });
</script>

<div class="container mx-auto">
  <div class="card p-4 mb-6">
    <div class="flex flex-col sm:flex-row justify-between items-center gap-4 mb-4">
      <div class="btn-group variant-filled-surface">
        <button class={`btn ${timeRange === 'week' ? 'variant-filled-primary' : ''}`}
                on:click={() => timeRange = 'week'}>
          周榜
        </button>
        <button class={`btn ${timeRange === 'month' ? 'variant-filled-primary' : ''}`}
                on:click={() => timeRange = 'month'}>
          月榜
        </button>
        <button class={`btn ${timeRange === 'year' ? 'variant-filled-primary' : ''}`}
                on:click={() => timeRange = 'year'}>
          年榜
        </button>
      </div>

      <button class="btn variant-filled-primary" on:click={fetchMirrors}>
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-arrow-clockwise"
             viewBox="0 0 16 16">
          <path fill-rule="evenodd" d="M8 3a5 5 0 1 0 4.546 2.914.5.5 0 0 1 .908-.417A6 6 0 1 1 8 2v1z" />
          <path
            d="M8 4.466V.534a.25.25 0 0 1 .41-.192l2.36 1.966c.12.1.12.284 0 .384L8.41 4.658A.25.25 0 0 1 8 4.466z" />
        </svg>
        <span class="ml-1">刷新数据</span>
      </button>
    </div>

    <!-- 加载状态 -->
    {#if loading}
      <div class="flex justify-center p-10">
        <div class="loader h-10 w-10"></div>
      </div>
      <!-- 错误信息 -->
    {:else if error}
      <div class="alert variant-filled-error">
        <p>加载失败: {error}</p>
        <button class="btn variant-filled" on:click={fetchMirrors}>重试</button>
      </div>
      <!-- 数据显示 -->
    {:else}
      <div class="space-y-6">
        <!-- 前三名突出显示 -->
        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
          {#each mirrors.slice(0, 3) as mirror, i}
            <a href={`/mirror/${mirror.name}`}
               class="card p-4 hover:bg-surface-200-700-token transition-all duration-200 hover:-translate-y-1">
              <header class="flex items-center gap-2 mb-3">
                {#if i === 0}
                  <div class="text-2xl text-yellow-500">🥇</div>
                {:else if i === 1}
                  <div class="text-2xl text-slate-300">🥈</div>
                {:else if i === 2}
                  <div class="text-2xl text-amber-600">🥉</div>
                {/if}
                <h3 class="h3 line-clamp-1">{mirror.name}</h3>
              </header>

              <div class="flex justify-between items-center mb-2">
                <span class="badge {getStatusBadgeClass(mirror.status)}">
                  {mirror.status === 'success' ? '正常' : mirror.status === 'failed' ? '失败' : mirror.status === 'syncing' ? '同步中' : mirror.status}
                </span>

                {#if mirror.trend !== undefined}
                  {@const trendInfo = getTrendInfo(mirror.trend)}
                  <div class="flex items-center gap-1 {trendInfo.color}">
                    {#if trendInfo.icon}
                      <svelte:component this={trendInfo.icon} size={14} />
                    {/if}
                    <span class="text-xs">{trendInfo.text}</span>
                  </div>
                {/if}
              </div>

              <div class="mb-4">
                <div class="flex justify-between text-xs mb-1">
                  <span>下载量</span>
                  <span class="font-medium">
                    {timeRange === 'week' ? formatNumber(mirror.weeklyDownloads) :
                      timeRange === 'month' ? formatNumber(mirror.monthlyDownloads) :
                        formatNumber(mirror.yearlyDownloads)}
                  </span>
                </div>
                <Progress
                  value={i === 0 ? 100 : 
                        i === 1 ? Math.floor(Math.random() * 30) + 60 : 
                        Math.floor(Math.random() * 20) + 40}
                  max={100}
                  height="h-2"
                  meter="bg-primary-500"
                  track="bg-surface-300-600-token"
                />
              </div>

              <div class="flex justify-between text-surface-600-300-token text-xs">
                <span>大小: {mirror.size || 'N/A'}</span>
                <span>更新: {mirror.lastUpdated}</span>
              </div>
            </a>
          {/each}
        </div>

        <!-- 排行榜列表 -->
        <div class="card p-4 variant-glass">
          <h3 class="h4 mb-4">更多热门镜像</h3>

          <div class="table-container">
            <table class="table table-hover">
              <thead>
                <tr>
                  <th class="w-16">排名</th>
                  <th>镜像名称</th>
                  <th>状态</th>
                  <th class="text-right">下载量</th>
                  <th class="text-right">趋势</th>
                  <th>最后更新</th>
                </tr>
              </thead>
              <tbody>
                {#each mirrors.slice(3, 20) as mirror, i}
                  <tr class="hover:bg-surface-200-700-token transition-colors duration-200 cursor-pointer"
                      on:click={() => window.location.href = `/mirror/${mirror.name}`}>
                    <td class="font-bold">{i + 4}</td>
                    <td>{mirror.name}</td>
                    <td>
                      <span class="badge {getStatusBadgeClass(mirror.status)}">
                        {mirror.status === 'success' ? '正常' : mirror.status === 'failed' ? '失败' : mirror.status === 'syncing' ? '同步中' : mirror.status}
                      </span>
                    </td>
                    <td class="text-right font-mono">
                      {timeRange === 'week' ? formatNumber(mirror.weeklyDownloads) :
                        timeRange === 'month' ? formatNumber(mirror.monthlyDownloads) :
                          formatNumber(mirror.yearlyDownloads)}
                    </td>
                    <td class="text-right">
                      {#if mirror.trend !== undefined}
                        {@const trendInfo = getTrendInfo(mirror.trend)}
                        <div class="flex items-center justify-end gap-1 {trendInfo.color}">
                          {#if trendInfo.icon}
                            <svelte:component this={trendInfo.icon} size={14} />
                          {/if}
                          <span class="text-xs">{trendInfo.text}</span>
                        </div>
                      {/if}
                    </td>
                    <td>{mirror.lastUpdated}</td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        </div>
      </div>
    {/if}
  </div>
</div>

<style>
    /* 美化卡片动画效果 */
    a.card {
        cursor: pointer;
        box-shadow: var(--shadow-md);
    }

    /* 表格行动画效果 */
    tr {
        cursor: pointer;
    }
</style>

<Toaster {toaster}></Toaster>
