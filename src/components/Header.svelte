<script>
  import { onMount } from "svelte";
  import { fade, fly, slide } from "svelte/transition";

  // 导入 Lucide 图标
  import { Search, Moon, Sun, Menu, X, Server, Download, Globe, Shield, Info } from "lucide-svelte";

  // 状态管理
  let darkMode = false;
  let mobileMenuOpen = false;
  let searchQuery = "";
  let isScrolled = false;

  // 导航项
  const navItems = [
    { name: "首页", href: "/", icon: Globe },
    { name: "镜像列表", href: "/mirrors", icon: Server },
    { name: "下载中心", href: "/downloads", icon: Download },
    { name: "状态监控", href: "/status", icon: Shield },
    { name: "关于我们", href: "/about", icon: Info }
  ];

  // 切换主题
  function toggleTheme() {
    darkMode = !darkMode;

    // 更新 HTML 类和属性
    document.documentElement.classList.toggle("dark", darkMode);
    document.documentElement.setAttribute(
      "data-theme",
      darkMode ? "dark" : "corporate"
    );

    // 保存用户偏好
    if (typeof localStorage !== "undefined") {
      localStorage.setItem("theme", darkMode ? "dark" : "light");
    }
  }

  // 监听滚动事件
  function handleScroll() {
    isScrolled = window.scrollY > 10;
  }

  // 在组件挂载时检查用户的主题偏好和设置滚动监听
  onMount(() => {
    // 主题设置
    if (typeof localStorage !== "undefined" && localStorage.getItem("theme") === "dark") {
      darkMode = true;
      document.documentElement.classList.add("dark");
    } else if (typeof window !== "undefined" && window.matchMedia("(prefers-color-scheme: dark)").matches) {
      darkMode = true;
      document.documentElement.classList.add("dark");
    }

    // 滚动监听
    window.addEventListener("scroll", handleScroll);

    // 组件卸载时移除监听器
    return () => {
      window.removeEventListener("scroll", handleScroll);
    };
  });
</script>

<header
  class="sticky top-0 z-50 w-full transition-all duration-300 {isScrolled ? 'shadow-md' : ''}
  {isScrolled ? 'bg-base-100/95' : 'bg-base-100'} 
  backdrop-blur-sm border-b border-neutral-content/10">
  <div class="container mx-auto px-4 max-w-7xl">
    <div class="flex items-center gap-8 h-16 lg:h-20">
      <!-- 网站 Logo 和标题 -->
      <div class="flex items-center">
        <a href="/" class="flex items-center space-x-3 group">
          <div class="w-10 h-10 bg-neutral text-neutral-content flex items-center justify-center font-bold text-xl
                     transition-all duration-300 group-hover:shadow-sm">
            <Server size={20} strokeWidth={1.5} />
          </div>
          <div class="flex flex-col">
            <span class="text-lg font-semibold text-base-content tracking-wide uppercase">Mirror Center</span>
            <span class="text-xs text-base-content/70 -mt-1 tracking-wider"> 官方开源软件镜像站 </span>
          </div>
        </a>
      </div>

      <!-- 桌面导航 -->
      <nav class="hidden lg:flex items-center space-x-1">
        {#each navItems as item}
          <a
            href={item.href}
            class="flex items-center px-4 py-2 rounded text-base-content/80 hover:text-primary hover:bg-base-200/30 transition-all duration-200 font-medium"
          >
            <svelte:component this={item.icon} size={16} class="mr-2" strokeWidth={1.5} />
            {item.name}
          </a>
        {/each}
      </nav>

      <div class="flex items-center gap-2 ml-auto">
        <!-- 主题切换 -->
        <button
          class="btn btn-sm btn-ghost hover:bg-base-200/70"
          on:click={toggleTheme}
          aria-label={darkMode ? "切换到亮色模式" : "切换到暗色模式"}
        >
          {#if darkMode}
            <Sun size={18} strokeWidth={1.5} />
          {:else}
            <Moon size={18} strokeWidth={1.5} />
          {/if}
        </button>

        <!-- 移动端菜单按钮 -->
        <button
          class="btn btn-sm btn-ghost lg:hidden hover:bg-base-200/70"
          on:click={()=> (mobileMenuOpen = !mobileMenuOpen)}
          aria-label="菜单"
          aria-expanded={mobileMenuOpen}
        >
          {#if mobileMenuOpen}
            <X size={18} strokeWidth={1.5} />
          {:else}
            <Menu size={18} strokeWidth={1.5} />
          {/if}
        </button>
      </div>
    </div>
  </div>

  <!-- 移动端菜单 -->
  {#if mobileMenuOpen}
    <div
      class="lg:hidden bg-base-100 border-t border-neutral-content/10"
      transition:slide={{duration: 200}}
    >
      <!-- 移动端导航链接 -->
      {#each navItems as item}
        <a
          href={item.href}
          class="flex items-center px-4 py-3 rounded text-base-content/80 hover:text-primary hover:bg-base-200/30 transition-all duration-200 font-medium"
        >
          <svelte:component this={item.icon} size={18} class="mr-3" strokeWidth={1.5} />
          {item.name}
        </a>
      {/each}
    </div>
  {/if}
</header>

<style>
    /* 活动链接样式 */
    :global(a.active) {
        /*@apply text-primary bg-primary/5 font-semibold;*/
    }

    /* 平滑过渡 */
    a, button {
        @apply transition-all duration-200;
    }

    /* 自定义滚动条 */
    :global(::-webkit-scrollbar) {
        width: 8px;
        height: 8px;
    }

    :global(::-webkit-scrollbar-track) {
        /*@apply bg-base-200/30;*/
    }

    :global(::-webkit-scrollbar-thumb) {
        /*@apply bg-neutral-content/20 rounded-full hover:bg-neutral-content/30;*/
    }
</style>