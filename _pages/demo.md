---
title: "示例页面"
excerpt: "Page not found. Your pixels are in another canvas."
sitemap: false
mermaid: true
permalink: /demo
---

1. 比如说图片怎么弄上来
2. 流程图怎么弄上来
第一个图
<div class="mermaid">
flowchart LR
     id1((a)) --> id2((b)) & c--> d
</div>
第二个图
<div class="mermaid">
flowchart LR
	 A[The JVM Run-Time Data Areas]
	 A --> A1[Shared Data Areas]
   A --> A2[Per-thread Data Areas]
	 A1 --> B[Method Area]
   A2 --> C[VM Stack]
	 A2 --> D[Native Method Stack]
	 A1 --> E[Heap]
   A2 --> F[Program Counter Register]
   A1 --> G[Run-Time Constant Pool]
</div>
第三个图
<div class="mermaid">
mindmap
  root((mindmap))
    Origins
      Long history
      ::icon(fa fa-book)
      Popularisation
        British popular psychology author Tony Buzan
    Research
      On effectiveness<br/>and features
      On Automatic creation
        Uses
            Creative techniques
            Strategic planning
            Argument mapping
    Tools
      Pen and paper
      Mermaid
</div>
第四个图
<div class="mermaid">
sequenceDiagram
    Alice->>John: Hello John, how are you?
    John-->>Alice: Great!
    Alice-)John: See you later!
</div>