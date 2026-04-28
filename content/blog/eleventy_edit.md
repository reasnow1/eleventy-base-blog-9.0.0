---
title: Eleventy 模板修改记录 Eleventy Temple changes
description: This is a post on My Blog about agile frameworks.
date: 2026-04-09
tags: Eleventy
---
# 11ty base blog 模板修改记录

本文档记录了针对 [eleventy-base-blog](https://github.com/11ty/eleventy-base-blog) 模板的主要修改内容，包括移除 feeds 和图片处理功能、添加静态资源前缀、修改路径前缀、增加手动主题切换、新增返回主站页面等。

## 1. 移除 feeds 与图片处理功能

### 修改 `eleventy.config.js`

删除 feeds 相关的样式文件复制：

```js
// 删除以下行
.addPassthroughCopy("./content/feed/pretty-atom-feed.xsl");
```

删除 `feedPlugin` 插件配置（整个代码块）：

```js
eleventyConfig.addPlugin(feedPlugin, {
  type: "atom",
  outputPath: "/feed/feed.xml",
  stylesheet: "pretty-atom-feed.xsl",
  templateData: {
    eleventyNavigation: {
      key: "Feed",
      order: 4
    }
  },
  collection: {
    name: "posts",
    limit: 10,
  },
  metadata: {
    language: "en",
    title: "Blog Title",
    subtitle: "This is a longer description about your blog.",
    base: "https://example.com/",
    author: {
      name: "Your Name"
    }
  }
});
```

删除图片处理插件 `eleventyImageTransformPlugin` 配置（整个代码块）：

```js
eleventyConfig.addPlugin(eleventyImageTransformPlugin, {
  extensions: "html",
  formats: ["avif", "webp", "auto"],
  defaultAttributes: {
    loading: "lazy",
    decoding: "async",
  }
});
```

> 这两部分分别对应 feeds 生成和图片处理，已完全移除。

## 2. 添加静态资源前缀短代码

在 `export default async function(eleventyConfig) {}` 内部增加以下短代码，用于后续从资源站引用图片等静态文件，并增加复制按钮：

```js
eleventyConfig.addPairedShortcode("staticBase", function (content) {
  return `${content}https://kdxc.uno:65432/studio/blog`;
});
```

使用时示例：

```njk
{% raw %}
![debian-terminal]({% staticBase %}{% endstaticBase %}/debian/debian.png)
{% endraw %}
```

该短代码会自动补全之前配置的静态资源前缀：`https://kdxc.uno:65432/studio/blog`

## 3. 修改路径前缀

```js
// 原配置
// pathPrefix: "/",

// 改为
pathPrefix: "/11_ty_blog/",
```

> 注意：不要直接使用 `/blog`，避免访问 `/blog/blog` 时路径重复覆盖。

## 4. 调整 `public` 文件夹

- 删除 `public/img` 文件夹
- 保留 `public/css` 文件夹

## 5. 增加自定义主题样式（`index.css`）

在 `public/css/index.css` 中添加以下内容：

```css
/* 手动亮色模式：强制使用亮色变量（覆盖系统暗色） */
[data-theme="light"] {
  --color-gray-20: #e0e0e0;
  --color-gray-50: #C0C0C0;
  --color-gray-90: #333;

  --background-color: #fff;

  --text-color: var(--color-gray-90);
  --text-color-link: #082840;
  --text-color-link-active: #5f2b48;
  --text-color-link-visited: #17050F;

  --syntax-tab-size: 2;
}

/* 手动暗色模式：强制使用暗色变量（覆盖系统亮色） */
[data-theme="dark"] {
  --color-gray-20: #e0e0e0;
  --color-gray-50: #C0C0C0;
  --color-gray-90: #dad8d8;

  --text-color-link: #1493fb;
  --text-color-link-active: #6969f7;
  --text-color-link-visited: #a6a6f8;

  --background-color: #15202b;
}

/* 让图片最大宽度为容器宽度，高度自适应 */
img {
  max-width: 100%;
  height: auto;
}

/* Wrapper for each code block */
.code-wrapper {
  position: relative;   /* Anchor for the absolute button */
  margin: 1rem 0;       /* Adjust spacing as needed */
}

/* Make the pre scrollable horizontally */
.code-wrapper pre {
  overflow-x: auto;
  margin: 0;            /* Remove default margins – wrapper handles spacing */
  padding: 0.75rem 0 0.75rem 1rem;
  border-radius: 6px;
  padding-right: 80px;  /* 根据按钮实际宽度调整，一般 70-80px 足够 */
}

/* Copy button – fixed at top‑right of the wrapper */
.code-wrapper .copy-button {
  position: absolute;
  top: 8px;
  right: 8px;
  background: #fff;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 0.8rem;
  padding: 4px 8px;
  cursor: pointer;
  z-index: 10;          /* Ensure it stays above the code */
}

.code-wrapper .copy-button:hover {
  background: #e0e0e0;
}
/* copy button end */
```

## 6. 添加主题切换、返回顶部按钮（`base.njk`）

修改 `_includes/layouts/base.njk`，在合适位置添加按钮及切换脚本。

### 6.1 添加按钮

```njk
<button id="themeToggle" style="Background-color: transparent; color: var(--text-color);">🌓 切换主题Theme</button>
{#- 此处添加按钮 #}
　
<button id="backToTopBtn" title="Go to top" style="Background-color: transparent; color: var(--text-color);">▲</button>
```

### 6.2 添加切换脚本，复制按钮、返回顶部脚本

```html
{#- 手动亮暗 #}
<script>
  const btn = document.getElementById('themeToggle');
  const root = document.documentElement;
  const saved = localStorage.getItem('theme');
  if (saved) root.setAttribute('data-theme', saved);
  btn.onclick = () => {
    const cur = root.getAttribute('data-theme');
    const next = cur === 'dark' ? 'light' : 'dark';
    root.setAttribute('data-theme', next);
    localStorage.setItem('theme', next);
  };
</script>
{#- copy button #}
<script>
  (function() {
    // Wait for DOM to be fully loaded
    document.addEventListener('DOMContentLoaded', function() {
      // Find all pre > code blocks
      const blocks = document.querySelectorAll('pre:has(code)');
      const copyButtonLabel = 'Copy';

      blocks.forEach((pre) => {
        // Create wrapper
        const wrapper = document.createElement('div');
        wrapper.className = 'code-wrapper';
        pre.parentNode.insertBefore(wrapper, pre);
        wrapper.appendChild(pre);

        // Create button
        const button = document.createElement('button');
        button.innerText = copyButtonLabel;
        button.classList.add('copy-button');
        wrapper.appendChild(button);

        // Copy function using execCommand (works on HTTP)
        button.addEventListener('click', function() {
          const code = pre.querySelector('code');
          const text = code.innerText;

          // Create a temporary textarea to hold the text
          const textarea = document.createElement('textarea');
          textarea.value = text;
          // Make it invisible but part of the DOM
          textarea.style.position = 'fixed';
          textarea.style.top = '-9999px';
          textarea.style.left = '-9999px';
          document.body.appendChild(textarea);
          textarea.select();
          textarea.setSelectionRange(0, text.length); // For mobile

          let success = false;
          try {
            success = document.execCommand('copy');
          } catch (err) {
            console.error('Copy failed:', err);
          }
          document.body.removeChild(textarea);

          // Visual feedback
          if (success) {
            button.innerText = 'Copied';
          } else {
            button.innerText = 'Failed';
          }
          setTimeout(() => {
            button.innerText = copyButtonLabel;
          }, 700);
        });
      });
    });
  })();
</script>
<script>
	// Back to top button
(function() {
  const btn = document.getElementById('backToTopBtn');
  if (!btn) return;

  // Show/hide button based on scroll position
  window.addEventListener('scroll', function() {
    if (window.scrollY > 300) {  // Show after 300px of scrolling
      btn.classList.add('show');
    } else {
      btn.classList.remove('show');
    }
  });

  // Scroll to top when clicked
  btn.addEventListener('click', function() {
    window.scrollTo({
      top: 0,
      behavior: 'smooth'   // Smooth scrolling
    });
  });
})();
</script>
```

## 7. 修改站点标题（`metadata.js`）

编辑 `_data/metadata.js`，修改 `title` 字段为你的博客名称。

```js
title: "博客标题",
```

## 8. 新增返回主站页面（`go_back.njk`）

在 `content/` 目录下新建 `go_back.njk` 文件，内容如下：

```njk
---
js
const eleventyNavigation = {
  key: "返回主站Go back",
  order: 4
};
---

<meta http-equiv="refresh" content="0; url=http://www.tongda.xyz/" />
正在返回……<br/>
如返回失败，请点击<a href="http://www.tongda.xyz/">返回</a>
<script>
  window.location.href = "http://www.tongda.xyz/";
</script>
```

> 该页面会自动跳转到指定主页。

## 9. 关于页面改成标签页面

- 删除 `about.md` “关于”页面。
- 修改 `content/` 目录下 `tags.njk` 文件，头部增加内容如下：

```njk
---js
const eleventyNavigation = {
	key: "标签Tags",
	order: 3
};
---
```

## 10. 博客文章

- 博客文章存放在 `content/blog/` 目录下，按 Markdown 格式编写即可。

