---
layout:     post
title:      元提示之——推理标签
subtitle:   Meta‑prompts 工具解析
date:       2025-06-29
author:     Bruce Wong
header-img: img/IMG_1515.WEBP
catalog: true
tags:
    - Agile
    - 敏捷
    - prompt engineering
    - AI
---

上一篇介绍了使用元提示(meta-prompt)，基于你的idea创建高效提示词的方法。元提示还有一个版本，是修改已经存在的提示词，并尽可能保留提示词中的关键信息，例如：你预设的一些参数符号等，这样可以尽可能减少修改后的提示词对已有系统集成的影响。

分析元提示的内容，你可以看到里面有一个神奇的小标签——<Reasoning>标签，他可以引导普通模型（非推理）进行主动推理。推理过程可以理解为CoT（思维链）的过程。
```
<reasoning>
- Simple Change: (yes/no) Is the change description explicit and simple? (If so, skip the rest of these questions.)
- Reasoning: (yes/no) Does the current prompt use reasoning, analysis, or chain of thought?
    - Identify: (max 10 words) if so, which section(s) utilize reasoning?
    - Conclusion: (yes/no) is the chain of thought used to determine a conclusion?
    - Ordering: (before/after) is the chain of though located before or after
- Structure: (yes/no) does the input prompt have a well defined structure
- Examples: (yes/no) does the input prompt have few-shot examples
    - Representative: (1-5) if present, how representative are the examples?
- Complexity: (1-5) how complex is the input prompt?
    - Task: (1-5) how complex is the implied task?
    - Necessity: ()
- Specificity: (1-5) how detailed and specific is the prompt? (not to be confused with length)
- Prioritization: (list) what 1-3 categories are the MOST important to address.
- Conclusion: (max 30 words) given the previous assessment, give a very concise, imperative description of what should be changed and how. this does not have to adhere strictly to only the categories listed
</reasoning>
```
通过这个标签，你的本地模型（Phi，Llama，Mistral等）秒变推理模型（OpenAI O3/Deepseek R1）。看一下下面的视频吧。
---

<iframe
  src="//player.bilibili.com/player.html?bvid=BV12BgRz3EHD&cid=30750739574&p=1"
  width="100%"
  height="500"
  frameborder="0"
  allowfullscreen>
</iframe>

---

### 参考链接

[OpenAI Prompt Generation Guide](https://platform.openai.com/docs/guides/prompt-generation#prompt-edits)

