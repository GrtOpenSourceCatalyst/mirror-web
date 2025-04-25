<script>
  import { Tabs,  } from "@skeletonlabs/skeleton-svelte";

  // 接收镜像数据作为属性
  export let mirror;

  // 复制代码到剪贴板的功能
  let copiedCode = "";
  let copyTimeout;

  function copyToClipboard(code) {
    navigator.clipboard.writeText(code)
      .then(() => {
        copiedCode = code;
        if (copyTimeout) clearTimeout(copyTimeout);
        copyTimeout = setTimeout(() => {
          copiedCode = "";
        }, 3000);
      })
      .catch(err => {
        console.error("无法复制代码: ", err);
      });
  }

  // 教程数据 - 在生产环境中应来自API或CMS
  const tutorialData = {
    // Ubuntu 镜像的教程
    ubuntu: {
      title: "Ubuntu 镜像使用帮助",
      description: "Ubuntu 是一个基于 Debian 的 Linux 发行版，具有良好的社区支持和丰富的软件包。",
      tabs: [
        {
          id: "apt-sources",
          label: "修改软件源",
          content: `
            <h3 class="h3">修改软件源</h3>
            <p>Ubuntu 的软件源配置文件是 <code>/etc/apt/sources.list</code>。将系统自带的该文件做个备份，再替换为下面内容：</p>
            <div class="code-block">
              <pre><code>deb http://mirrors.example.org/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.example.org/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.example.org/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.example.org/ubuntu/ focal-security main restricted universe multiverse</code></pre>
              <button class="copy-button">复制</button>
            </div>
            <p>然后执行更新：</p>
            <div class="code-block">
              <pre><code>sudo apt update</code></pre>
              <button class="copy-button">复制</button>
            </div>
          `
        },
        {
          id: "iso-download",
          label: "ISO 镜像下载",
          content: `
            <h3 class="h3">ISO 镜像下载</h3>
            <p>Ubuntu 发行版 ISO 镜像可以在以下位置找到：</p>
            <ul class="list-disc ml-6 mb-4">
              <li><a href="#" class="anchor">Ubuntu 22.04 LTS (Jammy Jellyfish)</a></li>
              <li><a href="#" class="anchor">Ubuntu 20.04 LTS (Focal Fossa)</a></li>
              <li><a href="#" class="anchor">Ubuntu 18.04 LTS (Bionic Beaver)</a></li>
            </ul>
            <p>推荐使用 LTS (长期支持) 版本以获取更稳定的体验。</p>
          `
        },
        {
          id: "faq",
          label: "常见问题",
          content: `
            <h3 class="h3">常见问题</h3>
            <div class="card p-4 mb-4">
              <h4 class="h4">无法更新软件源？</h4>
              <p>可能是由于网络问题或者软件源配置错误导致的。请检查您的网络连接，确保软件源地址正确。</p>
            </div>
            <div class="card p-4 mb-4">
              <h4 class="h4">如何切换回官方源？</h4>
              <p>您可以通过图形界面的"软件与更新"工具选择官方源，或者手动编辑 sources.list 文件恢复备份。</p>
            </div>
            <div class="card p-4">
              <h4 class="h4">为什么下载速度较慢？</h4>
              <p>下载速度受多种因素影响，包括当前的网络环境、服务器负载等。您可以尝试在不同时段下载或者切换到其他镜像源。</p>
            </div>
          `
        }
      ]
    },
    // Debian 镜像的教程
    debian: {
      title: "Debian 镜像使用帮助",
      description: "Debian 是一个自由的操作系统，由全球众多志愿者共同维护。",
      tabs: [
        {
          id: "apt-sources",
          label: "修改软件源",
          content: `
            <h3 class="h3">修改软件源</h3>
            <p>Debian 的软件源配置文件是 <code>/etc/apt/sources.list</code>。将系统自带的该文件做个备份，再替换为下面内容：</p>
            <div class="code-block">
              <pre><code>deb http://mirrors.example.org/debian/ bullseye main contrib non-free
deb http://mirrors.example.org/debian/ bullseye-updates main contrib non-free
deb http://mirrors.example.org/debian/ bullseye-backports main contrib non-free
deb http://security.debian.org/debian-security bullseye-security main contrib non-free</code></pre>
              <button class="copy-button">复制</button>
            </div>
            <p>然后执行更新：</p>
            <div class="code-block">
              <pre><code>sudo apt update</code></pre>
              <button class="copy-button">复制</button>
            </div>
          `
        },
        {
          id: "iso-download",
          label: "ISO 镜像下载",
          content: `
            <h3 class="h3">ISO 镜像下载</h3>
            <p>Debian 发行版 ISO 镜像可以在以下位置找到：</p>
            <ul class="list-disc ml-6 mb-4">
              <li><a href="#" class="anchor">Debian 11 (Bullseye)</a></li>
              <li><a href="#" class="anchor">Debian 10 (Buster)</a></li>
              <li><a href="#" class="anchor">Debian 9 (Stretch)</a></li>
            </ul>
          `
        },
        {
          id: "faq",
          label: "常见问题",
          content: `
            <h3 class="h3">常见问题</h3>
            <div class="card p-4 mb-4">
              <h4 class="h4">non-free 和 contrib 是什么？</h4>
              <p>Debian 将软件包分为 main、contrib 和 non-free 三个部分。main 包含符合 Debian 自由软件指导方针的软件包；contrib 包含的软件虽然是自由的，但它依赖于非自由软件；non-free 包含的软件不符合 Debian 自由软件指导方针。</p>
            </div>
          `
        }
      ]
    },
    // 默认通用教程模板
    default: {
      title: "镜像使用帮助",
      description: "此镜像源提供了各种软件包和资源的下载服务。",
      tabs: [
        {
          id: "usage",
          label: "使用方法",
          content: `
            <h3 class="h3">使用方法</h3>
            <p>请根据不同操作系统和软件的具体情况，参考下面的指南配置您的软件源：</p>
            <div class="card p-4 mb-4">
              <h4 class="h4"><Terminal size={20} class="inline-block mr-1" />命令行配置</h4>
              <p class="mb-2">您可以通过命令行工具配置此镜像源：</p>
              <div class="code-block">
                <pre><code># 替换为适合您系统的命令
echo "source_url=http://mirrors.example.org/${mirror?.name}" >> ~/.config/mirror.conf</code></pre>
                <button class="copy-button">复制</button>
              </div>
            </div>
            
            <div class="card p-4">
              <h4 class="h4">图形界面配置</h4>
              <p>在软件的设置页面中，找到"镜像源"或"软件源"设置，输入以下地址：</p>
              <div class="code-block">
                <pre><code>http://mirrors.example.org/${mirror?.name}</code></pre>
                <button class="copy-button">复制</button>
              </div>
            </div>
          `
        },
        {
          id: "faq",
          label: "常见问题",
          content: `
            <h3 class="h3">常见问题</h3>
            <div class="card p-4 mb-4">
              <h4 class="h4">如何检查镜像是否可用？</h4>
              <p>您可以通过访问镜像站首页或者尝试使用 ping 命令检测连通性：</p>
              <div class="code-block">
                <pre><code>ping mirrors.example.org</code></pre>
                <button class="copy-button">复制</button>
              </div>
            </div>
            
            <div class="card p-4">
              <h4 class="h4">遇到问题如何寻求帮助？</h4>
              <p>如果您在使用过程中遇到问题，可以通过以下方式寻求帮助：</p>
              <ul class="list-disc ml-6">
                <li>查看镜像站的常见问题解答</li>
                <li>在官方社区论坛发帖</li>
                <li>联系我们的技术支持团队</li>
              </ul>
            </div>
          `
        }
      ]
    }
  };

  // 根据镜像名称选择对应教程，如果没有特定教程则使用默认模板
  $: tutorial = tutorialData[mirror.name.toLowerCase()] || tutorialData.default;

  // 自动替换内容中的变量
  $: processedTabs = tutorial.tabs.map(tab => ({
    ...tab,
    content: tab.content.replace(/\${mirror\?.name}/g, mirror.name)
  }));

  // 更新时间
  $: lastUpdateDate = new Date(mirror.last_update_ts * 1000);
  $: lastUpdateFormatted = lastUpdateDate.toLocaleString("zh-CN", {
    year: "numeric",
    month: "long",
    day: "numeric",
    hour: "2-digit",
    minute: "2-digit"
  });
</script>

<div class="card p-6 variant-glass">
  <header class="mb-6">
    <h1 class="h2 mb-3">{mirror.name} <span
      class="badge {mirror.status === 'success' ? 'badge-success' : mirror.status === 'failed' ? 'badge-error' : 'badge-warning'}">{mirror.status === 'success' ? '正常' : mirror.status === 'failed' ? '失败' : '同步中'}</span>
    </h1>
    <p class="text-lg">{tutorial.description}</p>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mt-4">
      <div class="card p-3 flex items-center gap-3">
        <div class="bg-primary-500/20 p-2 rounded-token">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="text-primary-500"
               viewBox="0 0 16 16">
            <path
              d="M8 4.754a3.246 3.246 0 1 0 0 6.492 3.246 3.246 0 0 0 0-6.492zM5.754 8a2.246 2.246 0 1 1 4.492 0 2.246 2.246 0 0 1-4.492 0z" />
            <path
              d="M9.796 1.343c-.527-1.79-3.065-1.79-3.592 0l-.094.319a.873.873 0 0 1-1.255.52l-.292-.16c-1.64-.892-3.433.902-2.54 2.541l.159.292a.873.873 0 0 1-.52 1.255l-.319.094c-1.79.527-1.79 3.065 0 3.592l.319.094a.873.873 0 0 1 .52 1.255l-.16.292c-.892 1.64.901 3.434 2.541 2.54l.292-.159a.873.873 0 0 1 1.255.52l.094.319c.527 1.79 3.065 1.79 3.592 0l.094-.319a.873.873 0 0 1 1.255-.52l.292.16c1.64.893 3.434-.902 2.54-2.541l-.159-.292a.873.873 0 0 1 .52-1.255l.319-.094c1.79-.527 1.79-3.065 0-3.592l-.319-.094a.873.873 0 0 1-.52-1.255l.16-.292c.893-1.64-.902-3.433-2.541-2.54l-.292.159a.873.873 0 0 1-1.255-.52l-.094-.319zm-2.633.283c.246-.835 1.428-.835 1.674 0l.094.319a1.873 1.873 0 0 0 2.693 1.115l.291-.16c.764-.415 1.6.42 1.184 1.185l-.159.292a1.873 1.873 0 0 0 1.116 2.692l.318.094c.835.246.835 1.428 0 1.674l-.319.094a1.873 1.873 0 0 0-1.115 2.693l.16.291c.415.764-.42 1.6-1.185 1.184l-.291-.159a1.873 1.873 0 0 0-2.693 1.116l-.094.318c-.246.835-1.428.835-1.674 0l-.094-.319a1.873 1.873 0 0 0-2.692-1.115l-.292.16c-.764.415-1.6-.42-1.184-1.185l.159-.291A1.873 1.873 0 0 0 1.945 8.93l-.319-.094c-.835-.246-.835-1.428 0-1.674l.319-.094A1.873 1.873 0 0 0 3.06 4.377l-.16-.292c-.415-.764.42-1.6 1.185-1.184l.292.159a1.873 1.873 0 0 0 2.692-1.115l.094-.319z" />
          </svg>
        </div>
        <div>
          <h3 class="text-base font-medium">状态</h3>
          <p>{mirror.status === 'success' ? '正常运行中' : mirror.status === 'failed' ? '同步失败' : '正在同步'}</p>
        </div>
      </div>

      <div class="card p-3 flex items-center gap-3">
        <div class="bg-tertiary-500/20 p-2 rounded-token">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="text-tertiary-500"
               viewBox="0 0 16 16">
            <path
              d="M16 8A8 8 0 1 1 0 8a8 8 0 0 1 16 0zM8 3.5a.5.5 0 0 0-1 0V9a.5.5 0 0 0 .252.434l3.5 2a.5.5 0 0 0 .496-.868L8 8.71V3.5z" />
          </svg>
        </div>
        <div>
          <h3 class="text-base font-medium">最后更新</h3>
          <p>{lastUpdateFormatted}</p>
        </div>
      </div>

      <div class="card p-3 flex items-center gap-3">
        <div class="bg-secondary-500/20 p-2 rounded-token">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="text-secondary-500"
               viewBox="0 0 16 16">
            <path
              d="M5.338 1.59a61.44 61.44 0 0 0-2.837.856.481.481 0 0 0-.328.39c-.554 4.157.726 7.19 2.253 9.188a10.725 10.725 0 0 0 2.287 2.233c.346.244.652.42.893.533.12.057.218.095.293.118a.55.55 0 0 0 .101.025.615.615 0 0 0 .1-.025c.076-.023.174-.061.294-.118.24-.113.547-.29.893-.533a10.726 10.726 0 0 0 2.287-2.233c1.527-1.997 2.807-5.031 2.253-9.188a.48.48 0 0 0-.328-.39c-.651-.213-1.75-.56-2.837-.855C9.552 1.29 8.531 1.067 8 1.067c-.53 0-1.552.223-2.662.524zM5.072.56C6.157.265 7.31 0 8 0s1.843.265 2.928.56c1.11.3 2.229.655 2.887.87a1.54 1.54 0 0 1 1.044 1.262c.596 4.477-.787 7.795-2.465 9.99a11.775 11.775 0 0 1-2.517 2.453 7.159 7.159 0 0 1-1.048.625c-.28.132-.581.24-.829.24s-.548-.108-.829-.24a7.158 7.158 0 0 1-1.048-.625 11.777 11.777 0 0 1-2.517-2.453C1.928 10.487.545 7.169 1.141 2.692A1.54 1.54 0 0 1 2.185 1.43 62.456 62.456 0 0 1 5.072.56z" />
          </svg>
        </div>
        <div>
          <h3 class="text-base font-medium">镜像大小</h3>
          <p>{mirror.size || '暂无数据'}</p>
        </div>
      </div>
    </div>
  </header>

  <Tabs.Group>
    {#each processedTabs as tab, i}
      <Tabs value={i}>{tab.label}</Tabs>
    {/each}

    {#each processedTabs as tab, i}
      <Tabs.Panel name={`panel-${i}`} class="guide-content">
        {@html tab.content}
      </Tabs.Panel>
    {/each}
  </Tabs.Group>
</div>

<style>
    .guide-content :global(h3) {
        margin-bottom: 1rem;
    }

    .guide-content :global(p) {
        margin-bottom: 1rem;
    }

    .guide-content :global(.code-block) {
        position: relative;
        margin-bottom: 1.5rem;
        background-color: var(--color-surface-900);
        border-radius: var(--theme-rounded-base);
        overflow: hidden;
    }

    .guide-content :global(pre) {
        padding: 1rem;
        overflow-x: auto;
        font-family: 'Space Mono', monospace;
        font-size: 0.875rem;
    }

    .guide-content :global(.copy-button) {
        position: absolute;
        top: 0.5rem;
        right: 0.5rem;
        background: var(--color-primary-500);
        color: white;
        border: none;
        border-radius: var(--theme-rounded-base);
        padding: 0.25rem 0.5rem;
        font-size: 0.75rem;
        cursor: pointer;
        transition: background 0.2s;
    }

    .guide-content :global(.copy-button:hover) {
        background: var(--color-primary-600);
    }

    .guide-content :global(code) {
        background-color: var(--color-surface-800);
        padding: 0.125rem 0.25rem;
        border-radius: var(--theme-rounded-base);
        font-family: 'Space Mono', monospace;
        font-size: 0.875em;
    }
</style>
