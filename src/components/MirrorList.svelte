<script>
  import { onMount } from "svelte";
  import { Toaster, createToaster } from "@skeletonlabs/skeleton-svelte";
  import { RefreshCw, Search } from "lucide-svelte";

  // 接收服务端渲染的初始数据
  export let initialMirrors = [];
  export let initialError = null;

  // 状态变量
  let mirrors = initialMirrors;
  let loading = initialMirrors.length === 0 && !initialError;
  let error = initialError;
  let searchTerm = "";
  let sortBy = "name"; // 默认按名称排序

  // Toast 通知
  const toaster = createToaster();

  // 获取镜像数据
  async function fetchMirrors() {
    console.log("Fetching mirrors...");
    try {
      loading = true;
      const response = await fetch("http://localhost:8082/static/tunasync.json");
      console.log(response);

      if (!response.ok) {
        throw new Error(`HTTP 错误: ${response.status}`);
      }

      const data = await response.json();
      mirrors = Object.entries(data).map(([name, info]) => ({
        name,
        ...info,
        // 格式化最后更新时间
        lastUpdated: new Date(info.last_update_ts * 1000).toLocaleString("zh-CN")
      }));
      error = null;
    } catch (err) {
      error = err.message;
      console.error(err);
      toaster.error({
        title: "加载失败",
        description: ` 无法获取镜像列表: ${err.message}`,
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

  // 跳转到镜像详情页
  function goToMirrorPage(mirrorName) {
    window.location.href = `/mirror/${mirrorName}`;
  }

  onMount(() => {
    // 如果没有初始数据或有错误时，才在客户端请求数据
    if (initialMirrors.length === 0 && !initialError) {
      fetchMirrors();
    }
    console.log("Component mounted with initial data:", initialMirrors.length);
  });
</script>

<div class="container mx-auto p-4">
  <div class="card p-4 mb-4">
    <h2 class="h2 mb-4"> 镜像列表 </h2>

    <div class="flex flex-col sm:flex-row gap-3 w-full sm:w-auto">
      <div class="relative w-full sm:w-64">
        <input
          type="text"
          bind:value={searchTerm}
          placeholder="搜索镜像..."
          class="input input-bordered w-full pl-10 h-10"
        />
        <div class="absolute left-3 top-1/2 -translate-y-1/2 text-base-content/50">
          <Search size={16} strokeWidth={1.5} />
        </div>
      </div>

      <div class="flex gap-2">
        <select bind:value={sortBy} class="select select-bordered h-10 text-sm">
          <option value="name"> 名称</option>
          <option value="status"> 状态</option>
          <option value="date"> 更新时间</option>
          <option value="size"> 大小</option>
        </select>

        <button
          class="btn btn-ghost h-10 ml-auto"
          on:click={fetchMirrors}
          disabled={loading}
        >
          {#if loading}
            <span class="loading loading-spinner loading-xs"></span>
          {:else}
            <RefreshCw size={16} class="mr-1" />
            刷新
          {/if}
        </button>
      </div>
    </div>

    <!-- 加载状态 -->
    {#if loading}
      <div class="flex justify-center p-10">
        <div class="loader h-8 w-8"></div>
      </div>
      <!-- 错误信息 -->
    {:else if error}
      <div class="alert variant-filled-error">
        <p> 加载失败: {error}</p>
        <button class="btn variant-filled" on:click={fetchMirrors}> 重试</button>
      </div>
      <!-- 数据列表 -->
    {:else}
      <div class="table-container">
        <table class="table table-hover">
          <thead>
            <tr>
              <th> 镜像名称</th>
              <th> 状态</th>
              <th> 大小</th>
              <th> 最后更新时间</th>
            </tr>
          </thead>
          <tbody>
            {#each filteredMirrors as mirror (mirror.name)}
              <tr class="{mirror.status === 'success' ? '' :
                mirror.status === 'failed' ? 'bg-red-500/10' :
                mirror.status === 'syncing' ? 'bg-yellow-500/10' : ''} hover:bg-surface-200/50 transition-colors duration-300 cursor-pointer"
                  on:click={()=> goToMirrorPage(mirror.name)}>
                <td><span class="space-grotesk-google">{mirror.name}</span></td>
                <td>
                  <span class="badge {getStatusBadgeClass(mirror.status)} badge-soft">
                    {mirror.status === 'success' ? '正常' : mirror.status === 'failed' ? '失败' : mirror.status === 'syncing' ? '同步中' : mirror.status}
                  </span>
                </td>
                <td><span class="space-grotesk-google">{mirror.size || 'N/A'}</span></td>
                <td><span class="space-grotesk-google">{mirror.lastUpdated}</span></td>
              </tr>
            {:else}
              <tr>
                <td colspan="4" class="text-center py-4"> 未找到符合条件的镜像</td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
      <p class="text-sm mt-2 text-surface-500-400-token"> 共 {filteredMirrors.length} 个镜像 </p>
    {/if}
  </div>
</div>