<script>
  import { onMount } from "svelte";
  import { Toaster, createToaster } from "@skeletonlabs/skeleton-svelte";

  // 状态变量
  let mirrors = [];
  let loading = true;
  let error = null;
  let searchTerm = "";
  let sortBy = "name"; // 默认按名称排序

  // Toast通知
  const toaster = createToaster();

  // 获取镜像数据
  async function fetchMirrors() {
    console.log("Fetching mirrors...");
    try {
      loading = true;
      const response = await fetch("http://localhost:8082/static/tunasync.json");
      console.log(response);

      if (!response.ok) {
        throw new Error(`HTTP错误: ${response.status}`);
      }

      const data = await response.json();
      mirrors = Object.entries(data).map(([name, info]) => ({
        name,
        ...info,
        // 格式化最后更新时间
        lastUpdated: new Date(info.last_update_ts * 1000).toLocaleString("zh-CN")
      }));

    } catch (err) {
      error = err.message;
      console.error(err);
      toaster.error({
        title: "加载失败",
        description: `无法获取镜像列表: ${err.message}`,
        timeout: 5000
      });
    } finally {
      loading = false;
    }
  }

  // 排序镜像
  $: sortedMirrors = [...mirrors].sort((a, b) => {
    if (sortBy === "name") return a.name.localeCompare(b.name);
    if (sortBy === "status") return a.status.localeCompare(b.status);
    if (sortBy === "date") return b.last_update_ts - a.last_update_ts;
    return 0;
  });

  // 过滤镜像
  $: filteredMirrors = sortedMirrors.filter(mirror =>
    mirror.name.toLowerCase().includes(searchTerm.toLowerCase())
  );

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

  onMount(() => {
    fetchMirrors();
    console.log("Mounted and fetching mirrors...");
  });
</script>

<div class="container mx-auto p-4">
  <div class="card p-4 mb-4">
    <h2 class="h2 mb-4">镜像列表</h2>

    <!-- 搜索和排序控制 -->
    <div class="flex flex-col sm:flex-row items-center gap-4 mb-4">
      <div class="input-group input-group-divider grid-cols-[auto_1fr] w-full sm:w-auto">
        <input
          type="search"
          bind:value={searchTerm}
          placeholder="搜索镜像..."
          class="input"
        />
      </div>

      <div class="flex items-center gap-2">
        <span>排序:</span>
        <select bind:value={sortBy} class="select">
          <option value="name">名称</option>
          <option value="status">状态</option>
          <option value="date">更新时间</option>
        </select>
      </div>

      <button class="btn variant-filled-primary ml-auto" on:click={fetchMirrors}>
        刷新列表
      </button>
    </div>

    <!-- 加载状态 -->
    {#if loading}
      <div class="flex justify-center p-10">
        <div class="loader h-8 w-8"></div>
      </div>
      <!-- 错误信息 -->
    {:else if error}
      <div class="alert variant-filled-error">
        <p>加载失败: {error}</p>
        <button class="btn variant-filled" on:click={fetchMirrors}>重试</button>
      </div>
      <!-- 数据列表 -->
    {:else}
      <div class="table-container">
        <table class="table table-hover">
          <thead>
            <tr>
              <th>镜像名称</th>
              <th>状态</th>
              <th>大小</th>
              <th>最后更新时间</th>
            </tr>
          </thead>
          <tbody>
            {#each filteredMirrors as mirror (mirror.name)}
              <tr class="{mirror.status === 'success' ? '' :
                mirror.status === 'failed' ? 'bg-red-500/10' :
                mirror.status === 'syncing' ? 'bg-yellow-500/10' : ''} hover:bg-surface-200/50 transition-colors duration-300 cursor-pointer">
                <td><span class="space-grotesk-google">{mirror.name}</span></td>
                <td>
        <span class="badge {getStatusBadgeClass(mirror.status)}">
          {mirror.status === 'success' ? '正常' : mirror.status === 'failed' ? '失败' : mirror.status === 'syncing' ? '同步中' : mirror.status}
        </span>
                </td>
                <td><span class="space-grotesk-google">{mirror.size || 'N/A'}</span></td>
                <td><span class="space-grotesk-google">{mirror.lastUpdated}</span></td>
              </tr>
            {:else}
              <tr>
                <td colspan="4" class="text-center py-4">未找到符合条件的镜像</td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
      <p class="text-sm mt-2 text-surface-500-400-token">共 {filteredMirrors.length} 个镜像</p>
    {/if}
  </div>
</div>