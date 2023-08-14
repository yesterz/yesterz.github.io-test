---
title: "示例页面"
excerpt: "Page not found. Your pixels are in another canvas."
sitemap: false
permalink: /demo
---

1. 比如说图片怎么弄上来
2. 流程图怎么弄上来
```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```  

<script src="https://unpkg.com/mermaid@8.9.3/dist/mermaid.min.js"></script>
<script>
  $(document).ready(function () {
    mermaid.initialize({
      startOnLoad:true,
      theme: "default",
    });
    window.mermaid.init(undefined, document.querySelectorAll('.language-mermaid'));
  });
</script>
