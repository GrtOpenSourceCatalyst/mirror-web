<script>
  import { onMount } from "svelte";
  import { Toaster, createToaster } from "@skeletonlabs/skeleton-svelte";
  import { Scale, Calendar, ArrowDownWideNarrow, Grid, List, HardDrive, Clock } from "lucide-svelte";

  // 接收服务端渲染的初始数据
  export let initialMirrors = [];
  export let initialError = null;
  export let categories = [];

  // 状态变量
  let mirrors = initialMirrors;
  let loading = initialMirrors.length === 0 && !initialError;
  let error = initialError;
  let searchTerm = "";
  let sortBy = "name"; // 默认按名称排序
  let selectedCategory = "all"; // 默认显示所有类别
  let viewMode = "grid"; // 默认网格视图 ('grid' 或 'list')

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
        // 添加示例分类
        category: determineCategory(name),
      }));
      error = null;
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

  // 根据名称确定类别的辅助函数（示例实现）
  function determineCategory(name) {
    if (/ubuntu|debian|fedora|centos|arch|opensuse/i.test(name)) return "Linux发行版";
    if (/python|npm|maven|gradle|rust|cargo/i.test(name)) return "开发工具";
    if (/apache|nginx|tomcat|docker|kubernetes/i.test(name)) return "服务器软件";
    if (/gcc|llvm|clang|make|cmake/i.test(name)) return "编译工具";
    if (/mysql|postgresql|mongodb|redis|elasticsearch/i.test(name)) return "数据库";
    if (/anaconda|tensorflow|pytorch|scikit/i.test(name)) return "数据科学";
    if (/kernel|core|linux/i.test(name)) return "核心组件";
    if (/ubuntu|windows|ios|android/i.test(name)) return "操作系统";
    return "其他镜像";
  }

  // 获取镜像图标（简单实现，实际可能需要更复杂的逻辑或外部图标库）
  function getMirrorIcon(name) {
    if (/ubuntu/i.test(name)) return "🐧";
    if (/debian/i.test(name)) return "🔴";
    if (/fedora/i.test(name)) return "🔵";
    if (/windows/i.test(name)) return "🪟";
    if (/python/i.test(name)) return "🐍";
    if (/anaconda/i.test(name)) return "🐍";
    if (/npm|node/i.test(name)) return "📦";
    if (/docker/i.test(name)) return "🐳";
    if (/database|mysql|postgres|mongo/i.test(name)) return "🗄️";
    if (/maven|gradle/i.test(name)) return "☕";
    return "📁";
  }

  // 排序和过滤
  $: {
    let filtered = [...mirrors];
    
    // 应用类别过滤
    if (selectedCategory !== "all") {
      filtered = filtered.filter(m => m.category === selectedCategory);
    }
    
    // 应用搜索过滤
    if (searchTerm) {
      filtered = filtered.filter(m => 
        m.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
        m.category.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }
    
    // 应用排序
    if (sortBy === "name") {
      filtered.sort((a, b) => a.name.localeCompare(b.name));
    } else if (sortBy === "status") {
      filtered.sort((a, b) => a.status.localeCompare(b.status));
    } else if (sortBy === "date") {
      filtered.sort((a, b) => b.last_update_ts - a.last_update_ts);
    } else if (sortBy === "size") {
      filtered.sort((a, b) => {
        // 简单的大小比较，实际实现可能更复杂
        const sizeA = a.size || '0';
        const sizeB = b.size || '0';
        return sizeB.localeCompare(sizeA);
      });
    }
    
    filteredMirrors = filtered;
  }

  // 过滤后的镜像
  let filteredMirrors = [];

  // 获取状态对应的样式
  function getStatusBadgeClass(status) {
    switch (status) {
      case "success": return "badge-success";
      case "failed": return "badge-error";
      case "syncing": return "badge-warning";
      default: return "badge-surface";
    }
  }

  // 格式化状态文本
  function formatStatus(status) {
    switch (status) {
      case "success": return "正常";
      case "failed": return "失败";
      case "syncing": return "同步中";
      default: return status;
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
  <div class="card p-4">
    <!-- 过滤和控制区域 -->
    <div class="flex flex-col gap-4 mb-6">
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
        <!-- 搜索框 -->
        <div class="input-group input-group-divider grid-cols-[auto_1fr]">
          <div class="input-group-shim">
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-search" viewBox="0 0 16 16">
              <path d="M11.742 10.344a6.5 6.5 0 1 0-1.397 1.398h-.001c.03.04.062.078.098.115l3.85 3.85a1 1 0 0 0 1.415-1.414l-3.85-3.85a1.007 1.007 0 0 0-.115-.1zM12 6.5a5.5 5.5 0 1 1-11 0 5.5 5.5 0 0 1 11 0z"/>
            </svg>
          </div>
          <input bind:value={searchTerm} type="search" placeholder="搜索镜像..." class="input p-2" />
        </div>

        <!-- 类别过滤 -->
        <div class="input-group input-group-divider grid-cols-[auto_1fr]">
          <div class="input-group-shim">
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-filter" viewBox="0 0 16 16">
              <path d="M6 10.5a.5.5 0 0 1 .5-.5h3a.5.5 0 0 1 0 1h-3a.5.5 0 0 1-.5-.5zm-2-3a.5.5 0 0 1 .5-.5h7a.5.5 0 0 1 0 1h-7a.5.5 0 0 1-.5-.5zm-2-3a.5.5 0 0 1 .5-.5h11a.5.5 0 0 1 0 1h-11a.5.5 0 0 1-.5-.5z"/>
            </svg>
          </div>
          <select bind:value={selectedCategory} class="select p-2">
            <option value="all">全部类别</option>
            {#each categories as category}
              <option value={category}>{category}</option>
            {/each}
          </select>
        </div>

        <!-- 排序选项 -->
        <div class="input-group input-group-divider grid-cols-[auto_1fr]">
          <div class="input-group-shim">
            <ArrowDownWideNarrow size={16} />
          </div>
          <select bind:value={sortBy} class="select p-2">
            <option value="name">按名称排序</option>
            <option value="status">按状态排序</option>
            <option value="date">按更新时间排序</option>
            <option value="size">按大小排序</option>
          </select>
        </div>
      </div>

      <!-- 视图切换和刷新按钮 -->
      <div class="flex justify-between items-center">
        <div class="btn-group variant-filled-surface">
          <button class={`btn ${viewMode === 'grid' ? 'variant-filled-primary' : ''}`} on:click={() => viewMode = 'grid'}>
            <Grid size={18} />
            <span class="hidden md:inline ml-1">网格视图</span>
          </button>
          <button class={`btn ${viewMode === 'list' ? 'variant-filled-primary' : ''}`} on:click={() => viewMode = 'list'}>
            <List size={18} />
            <span class="hidden md:inline ml-1">列表视图</span>
          </button>
        </div>

        <div class="flex items-center gap-2">
          <span class="text-sm text-surface-600-300-token">共 {filteredMirrors.length} 个镜像</span>
          <button class="btn variant-filled-primary" on:click={fetchMirrors}>
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-arrow-clockwise" viewBox="0 0 16 16">
              <path fill-rule="evenodd" d="M8 3a5 5 0 1 0 4.546 2.914.5.5 0 0 1 .908-.417A6 6 0 1 1 8 2v1z"/>
              <path d="M8 4.466V.534a.25.25 0 0 1 .41-.192l2.36 1.966c.12.1.12.284 0 .384L8.41 4.658A.25.25 0 0 1 8 4.466z"/>
            </svg>
            <span class="ml-1">刷新</span>
          </button>
        </div>
      </div>
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
    <!-- 数据显示 - 网格视图 -->
    {:else if viewMode === 'grid'}
      {#if filteredMirrors.length > 0}
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
          {#each filteredMirrors as mirror}
            <a href={`/mirrors/${mirror.name}`} class="card p-4 hover:bg-surface-200-700-token transition-colors duration-200">
              <header class="flex items-center gap-3 mb-3">
                <div class="text-2xl">{getMirrorIcon(mirror.name)}</div>
                <h3 class="h3 line-clamp-1">{mirror.name}</h3>
              </header>
              <div class="space-y-2">
                <div class="flex justify-between">
                  <span class="badge {getStatusBadgeClass(mirror.status)}">{formatStatus(mirror.status)}</span>
                  <span class="badge variant-soft">{mirror.category}</span>
                </div>
                <div class="flex items-center gap-2 text-sm text-surface-600-300-token">
                  <HardDrive size={14} />
                  <span>{mirror.size || 'N/A'}</span>
                </div>
                <div class="flex items-center gap-2 text-sm text-surface-600-300-token">
                  <Clock size={14} />
                  <span>{mirror.lastUpdated}</span>
                </div>
              </div>
            </a>
          {/each}
        </div>
      {:else}
        <div class="flex flex-col items-center justify-center p-10 space-y-4">
          <div class="text-6xl">🔍</div>
          <p class="text-xl text-center text-surface-600-300-token">未找到符合条件的镜像</p>
          <button class="btn variant-filled-primary" on:click={() => {searchTerm = ''; selectedCategory = 'all';}}>
            清除过滤条件
          </button>
        </div>
      {/if}
    <!-- 数据显示 - 列表视图 -->
    {:else}
      {#if filteredMirrors.length > 0}
        <div class="table-container">
          <table class="table table-hover">
            <thead>
              <tr>
                <th>镜像名称</th>
                <th>类别</th>
                <th>状态</th>
                <th>大小</th>
                <th>最后更新时间</th>
              </tr>
            </thead>
            <tbody>
              {#each filteredMirrors as mirror}
                <tr 
                  class="{mirror.status === 'success' ? '' : 
                    mirror.status === 'failed' ? 'bg-red-500/10' :
                    mirror.status === 'syncing' ? 'bg-yellow-500/10' : ''} hover:bg-surface-200/50 transition-colors duration-300"
                  on:click={() => window.location.href = `/mirrors/${mirror.name}`}
                >
                  <td class="flex items-center gap-2">
                    <span class="text-xl">{getMirrorIcon(mirror.name)}</span>
                    <span class="space-grotesk-google">{mirror.name}</span>
                  </td>
                  <td><span class="badge variant-soft">{mirror.category}</span></td>
                  <td>
                    <span class="badge {getStatusBadgeClass(mirror.status)}">
                      {formatStatus(mirror.status)}
                    </span>
                  </td>
                  <td><span class="space-grotesk-google">{mirror.size || 'N/A'}</span></td>
                  <td><span class="space-grotesk-google">{mirror.lastUpdated}</span></td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      {:else}
        <div class="flex flex-col items-center justify-center p-10 space-y-4">
          <div class="text-6xl">🔍</div>
          <p class="text-xl text-center text-surface-600-300-token">未找到符合条件的镜像</p>
          <button class="btn variant-filled-primary" on:click={() => {searchTerm = ''; selectedCategory = 'all';}}>
            清除过滤条件
          </button>
        </div>
      {/if}
    {/if}
  </div>
</div>

<style>
  /* 美化卡片动画效果 */
  a.card {
    cursor: pointer;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }
  a.card:hover {
    transform: translateY(-2px);
    box-shadow: var(--shadow-lg);
  }
  
  /* 表格行动画效果 */
  tr {
    cursor: pointer;
  }
</style>

<Toaster {toaster}></Toaster>
