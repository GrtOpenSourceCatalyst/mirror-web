<script>
  import { onMount } from "svelte";
  import { Toaster, createToaster } from "@skeletonlabs/skeleton-svelte";

  // 导入Lucide图标
  import { Search, Moon, Sun, Menu, X } from "lucide-svelte";

  // 状态管理
  let darkMode = false;
  let mobileMenuOpen = false;
  let searchQuery = "";

  // Toast通知
  const toaster = createToaster();

  // 切换主题
  function toggleTheme() {
    darkMode = !darkMode;
    document.documentElement.classList.toggle("dark", darkMode);

    // // 保存用户偏好
    // if (typeof localStorage !== "undefined") {
    //   localStorage.setItem("theme", darkMode ? "dark" : "light");
    // }

    // 显示主题切换通知
    toaster.info({
      type: "info",
      title: "主题切换",
      description: `已切换到${darkMode ? "暗色" : "亮色"}主题`,
      background: "variant-filled-primary",
      timeout: 2000
    });
  }

  // 处理搜索
  function handleSearch(e) {
    if (e.key === "Enter" || e.type === "click") {
      if (searchQuery.trim()) {
        toaster.trigger({
          message: `正在搜索: ${searchQuery}`,
          background: "variant-filled-secondary",
          timeout: 2000
        });
        // 这里可以添加实际的搜索逻辑
        searchQuery = "";
      }
    }
  }

  // // 在组件挂载时检查用户的主题偏好
  // onMount(() => {
  //   if (typeof localStorage !== "undefined" && localStorage.getItem("theme") === "dark") {
  //     darkMode = true;
  //     document.documentElement.classList.add("dark");
  //   } else if (typeof window !== "undefined" && window.matchMedia("(prefers-color-scheme: dark)").matches) {
  //     darkMode = true;
  //     document.documentElement.classList.add("dark");
  //   }
  // });
</script>

<header
  class="sticky top-0 z-50 w-full transition-all duration-300 bg-background/80 backdrop-blur-2xl border-b border-surface-300-600-token shadow-md">
  <div class="container mx-auto px-4">
    <div class="flex items-center justify-between h-16">
      <!-- 网站Logo和标题 -->
      <div class="flex items-center">
        <a href="/" class="flex items-center space-x-2">
          <div class="w-8 h-8 rounded-full bg-primary-500 flex items-center justify-center text-white font-bold">M</div>
          <span
            class="text-xl font-bold tracking-wider bg-gradient-to-r from-primary-500 to-tertiary-500 bg-clip-text text-transparent">我的镜像站</span>
        </a>
      </div>

      <!-- 桌面导航 -->
      <nav class="hidden md:flex items-center space-x-1">
        <a href="/" class="btn btn-sm variant-ghost-surface">首页</a>
        <a href="/browse" class="btn btn-sm variant-ghost-surface">浏览</a>
        <a href="/popular" class="btn btn-sm variant-ghost-surface">热门</a>
        <a href="/new" class="btn btn-sm variant-ghost-surface">最新</a>
        <a href="/about" class="btn btn-sm variant-ghost-surface">关于</a>
      </nav>

      <!-- 搜索和主题切换 -->
      <div class="flex items-center space-x-2">
        <!-- 搜索框 -->
        <div class="relative hidden md:block">
          <input
            type="search"
            bind:value={searchQuery}
            on:keydown={handleSearch}
            placeholder="搜索镜像..."
            class="input input-sm pl-9 pr-4 rounded-full w-40 lg:w-64 focus:w-72 transition-all duration-300"
          />
          <button
            class="absolute left-2 top-1/2 transform -translate-y-1/2 text-surface-500-400-token"
            on:click={handleSearch}
          >
            <Search size={18} />
          </button>
        </div>

        <!-- 主题切换 -->
        <button
          class="btn btn-sm variant-ghost-surface aspect-square"
          on:click={toggleTheme}
          aria-label="切换主题"
        >
          {#if darkMode}
            <Sun size={18} />
          {:else}
            <Moon size={18} />
          {/if}
        </button>

        <!-- 移动端菜单按钮 -->
        <button
          class="btn btn-sm variant-ghost-surface md:hidden aspect-square"
          on:click={() => mobileMenuOpen = !mobileMenuOpen}
          aria-label="菜单"
        >
          {#if mobileMenuOpen}
            <X size={18} />
          {:else}
            <Menu size={18} />
          {/if}
        </button>
      </div>
    </div>
  </div>
  <Toaster {toaster}></Toaster>

  <!-- 移动端菜单 -->
  {#if mobileMenuOpen}
    <div class="md:hidden bg-surface-100-800-token border-t border-surface-300-600-token">
      <div class="container mx-auto px-4 py-3 space-y-2">
        <a href="/" class="btn btn-sm w-full variant-ghost-surface justify-start">首页</a>
        <a href="/browse" class="btn btn-sm w-full variant-ghost-surface justify-start">浏览</a>
        <a href="/popular" class="btn btn-sm w-full variant-ghost-surface justify-start">热门</a>
        <a href="/new" class="btn btn-sm w-full variant-ghost-surface justify-start">最新</a>
        <a href="/about" class="btn btn-sm w-full variant-ghost-surface justify-start">关于</a>

        <!-- 移动端搜索框 -->
        <div class="relative w-full mt-2">
          <input
            type="search"
            bind:value={searchQuery}
            on:keydown={handleSearch}
            placeholder="搜索镜像..."
            class="input input-sm pl-9 pr-4 rounded-full w-full"
          />
          <button
            class="absolute left-2 top-1/2 transform -translate-y-1/2 text-surface-500-400-token"
            on:click={handleSearch}
          >
            <Search size={18} />
          </button>
        </div>
      </div>
    </div>
  {/if}
</header>

<style>
    /*!* 活动链接样式 *!*/
    /*:global(a.active) {*/
    /*    @apply bg-primary-500/20 text-primary-500;*/
    /*}*/

    /*!* 链接悬停效果 *!*/
    /*a.btn:hover {*/
    /*    @apply bg-primary-500/10;*/
    /*}*/

    /*!* 平滑过渡 *!*/
    /*a, button {*/
    /*    @apply transition-all duration-200;*/
    /*}*/
</style>