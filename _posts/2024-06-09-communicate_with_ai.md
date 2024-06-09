---
layout:     post
title:      AI并非万能，有效沟通是关键
subtitle:
date:       2024-06-09
author:     Bruce Wong
header-img: img/IMG_2772.png
catalog: true
tags:
    - Agile
    - 敏捷
    - 随笔
    - AI
    - ProductOwner
---

最近，我与一个产品团队合作，参与了一些与大模型相关的开发工作。今天分享一些与AI（这里指大语言模型ChatGPT）合作实现软件功能的一些感受和经验。

## 一句话需求一样不好用

许多人可能认为，AI的强大能力能解决软件工程中的所有问题，例如“一句话需求”。一些演示中，简单的描述似乎就能生成丰富的功能代码。然而，真实情况并非如此。以下是一个例子，展示了两个版本的提示词在同样模型和参数条件下生成的质量差距。

**需求背景：**该功能是根据用户的一段文字生成一定数量的单选题，同时附带正确选项。

**修改前：**

```markdown
You will be provided with a document, and answer using @Language@ language, use the following content to write {{@QuestionNumber@}} easy multiple choice questions with 1 correct answer and the others are wrong answers, please note that the answer should be provided in JSON format:
[{
    "Question":"What is the capital of France?",
    "Options": [
        {"option": "Berlin", "isCorrect": false},
        {"option": "Lyon", "isCorrect": false},
        {"option": "Paris", "isCorrect": true},
        {"option": "Amsterdam", "isCorrect": false}
    ]
}
{...if more quiz,please list here as JSON format...}
]

following question:
@Content@
```

**修改后：**

```markdown
system:
You are an AI assistant tasked with generating multiple-choice questions based on document content. Your goal is to create questions that are relevant and challenging, using the details provided in the document.
###Input Document Content:
@content@
###Task:
1. Generate exactly @questionnumber@ multiple-choice questions based on the content provided.
2. Each question must have one correct answer and three distractors (incorrect options).
3. Maintain the language consistency throughout the questions to match the original document content.

###Language Handling:
Ensure all parts of the question, including options, are in the same language as the input content to maintain relevance and context integrity.
###Response Format:
Respond in JSON format, structuring each question as an object within an array. Each object should contain the question text and a list of options, each marked for correctness.
###Example Response:
[{
    "Question":"What is the capital of France?",
    "Options": [
        {"option": "Berlin", "isCorrect": false},
        {"option": "Lyon", "isCorrect": false},
        {"option": "Paris", "isCorrect": true},
        {"option": "Amsterdam", "isCorrect": false}
    ]
},
{
    "Question": "What is the famouse place in Paris?",
    "Options": [
        {"option": "Historic Capitals", "isCorrect": false},
        {"option": "Modern Metropolises:", "isCorrect": false},
        {"option": "Island Paradises", "isCorrect": false},
        {"option": "Eiffel Tower", "isCorrect": true}
    ]
}
]
###Important:
1. Make sure to generate exactly @questionnumber@ questions.
2. The language using for the questions must same as the original document content.
```

修改前的提示词生成效果有如下问题经常发生：
1. 使用非英语文本生成题目，经常会变成英文题目。
2. 生成题目的个数少于要求。
3. 超过5道题之后，题目内容开始与指定的文章无关。
4. 返回的json格式不正确。

修改后的提示词，上面的问题都解决了。主要区别是提示词描述更清楚，文档有格式，无论是人还是AI都能分清主次，生成的功能更符合需求。

注意改之前也有例子，为什么输出格式仍然有错误，一个小细节是例子里面的json格式错了。见下图：
![IMG_jsonerror.jpeg](/img/AI/cowork/IMG_jsonerror.jpeg)

## 需求不清一样无法完成

模糊的需求描述不仅对人类工程师不友好，对AI也是如此。例如，下面的例子是两个评分标准，用户可以自定义评分标准。每一行是一个评分标准名字，每一列是不同分数段的打分规则。如果这些名字和描述本身就是是没有意义的字母组合，那么AI同样无法理解和处理。

![IMG_rubirc.png](/img/AI/cowork/IMG_rubirc.png)

你的期待是让AI能够根据打分规则进行打分。不过因为名字都没有意义的随意字母组合。那么结果会怎样呢？看下图的效果：

![IMG_rubrics2.png](/img/AI/cowork/IMG_rubrics2.png)

结果自然不理想。AI会尽可能提供其认为你需要的内容，但效果可能并不准确。如果是人类工程师，可能会直接告知无法完成任务。因此，提供清晰、具体的需求描述至关重要。

## Few Shots和思维链(CoT)组合的威力

如果你的应用以提示词为主，提示词的质量至关重要。Few shots和思维链（Chain of Thought, CoT）是两个有效的提示词技巧。参看最上面的提示词内容：

1. **任务描述：**在Task节中描述任务的工作步骤，告诉模型如何推理完成任务。
2. **输出格式的例子：**使用few shots的方式，让模型知道你期望的输出格式。

## 总结
很多现实工作中和人类工程师的沟通技巧，对AI同样有效。例如：
1. 就算是AI，也需要清晰的需求描述。和人类对需求文档的要求一样，越清晰越明确越好。而且AI更需要清晰的任务分解，否则固执的执行会导致错误的结果。
2. 现有的需求澄清和需求分解技巧，对AI一样有效。Few Shots本身就是实例化需求的一个例子。
3. CoT是通过分解任务步骤，让AI理解推理过程，并能够举一反三。
4. 同人类工程师协作中可能会有的问题，同AI的协作同样会遇到。有些会更加严重。

*用技术改变生活。欢迎关注我的公众号，交流落地经验。*

## 参考
- [少样本提示](https://www.promptingguide.ai/zh/techniques/fewshot)
- [Chain of Thought: CoT](https://www.promptingguide.ai/zh/techniques/cot)
