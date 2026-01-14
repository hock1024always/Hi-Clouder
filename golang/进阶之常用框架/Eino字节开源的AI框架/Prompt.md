# 提示词工程从0到1

Prompting Engineer 提示工程是一种相对较新的学科，用于开发和优化提示，以高效地使用语言模型（LM）进行各种应用和研究主题。提示工程技能有助于更好地理解大型语言模型（LLM）的能力和局限性。研究人员使用提示工程来改善LLM在各种常见和复杂任务（如问答和算术推理）上的能力。开发人员使用提示工程来设计与LLM和其他工具交互的稳健有效的提示技术。

## Prompt基础介绍

### 概念

- Prompt是**给 AI 模型的指令**，它可以是一个问题、一段文字描述，甚至可以是带有一堆参数的文字描述。AI 模型会基于 prompt 所提供的信息，生成对应的文本，亦或者图片。
- 比如，我们在 ChatGPT 里输入 `What is the capital of China?` (中国的首都是什么？)，这个问题就是 prompt。
- 目前的 AI 产品还比较早期，因为各种原因，产品设置了很多限制，如果你想要绕过一些限制，或者更好地发挥 AI 的能力，也需要用到 Prompt Engineering 技术。

### 基本原则

当我们尝试写一个更好的 Prompt 时，我们可以参考 Elavis Saravia 总结的优质 prompt 需包含的元素：

- **Context（选填）**： 背景信息，或者说是上下文信息，这可以引导模型做出更好的反应。
- **Instruction（必须）**： 指令，即你希望模型执行的具体任务。
- **Input Data（选填）**： 输入数据，告知模型需要处理的数据。
- **Output Indicator（选填）**： 输出指示器，告知模型我们要输出的类型或格式。

只要你按照这个框架写 prompt ，模型返回的结果都不会差。例如：

```
我想让你做一个 AI 写作导师。你的任务是使用人工智能工具（例如自然语言处理）向学生提供有关如何改进其作文的反馈。你还应该利用你在有效写作技巧方面的修辞知识和经验来建议学生可以更好地以书面形式表达他们的想法和想法的方法。//Context 背景信息
请以要点加上举例的格式回答。//输出指示
我的第一个请求是“请帮我修改下面这段文字：XXXXXXXXXXX”。//指令和输入
```

### 进阶知识

**链式思考（Chain-of-Thought Prompting）：** 是一种简单且广泛适用的方法，可提高大语言模型执行各种推理任务的能力。通过对算术和常识推理的实验，我们发现思维链式启发是模型规模的一种新兴属性。利用这种 Prompt 的方式，可以让你的大语言模型拥有更强的推理能力。

### 从一个例子看提示词工程

下面这个例子可以很好的解释提示词工程的用处

```
// 输入
Sky is //天空是
// 输出
the expanse of atmosphere and outer space seen from the surface of a planet, usually the Earth. The sky appears blue during the day due to the scattering of sunlight by the Earth's atmosphere, and black at night, with stars and other celestial objects visible. The sky is also a subject of art, poetry, and mythology, and has cultural and spiritual significance in many human societies.//从行星表面，通常是地球上看到的大气层和外层空间的广阔区域。白天，由于太阳光在地球大气层中的散射，天空呈现出蓝色，而晚上则是黑色的，可以看到星星和其他天体。天空也是艺术、诗歌和神话的主题，对许多人类社会具有文化和精神上的重要意义。
```

这段回答解释的很全面，但是正因为太全面了，可能会出乎我们的预料
```
// 输入
Complete the sentence: 
Sky is//天空是
// 输出
a beautiful and ever-changing canvas that inspires awe and wonder in those who gaze upon it.//一个美丽且不断变化的画布，它能够激发那些凝视它的人心中的敬畏和惊奇。
```

上面这个案例就是我们加入了提示词之后，将回答锁定到一定的范围之内

## Prompt实践

如果我们想要学习下面的这段话，下面的这两个输入

```
将下面这段话翻译成中文：
All the bright, precious things fade so fast. And they don't come back.
```

```
请你扮演一名资深的英语教师，先将下面这段英文文本给我教学，再将其中关键的单词挑选出来详细解释并补充例句。
英文文本：
All the bright, precious things fade so fast. And they don't come back.
```

这两种输入产生的差异如下：
```
好的，让我来教你学习这段英文文本。
这句话是由英国作家奥斯卡·王尔德创作的，它的意思是：所有明亮、珍贵的东西都会很快消逝，而它们不会再回来。
现在，让我们来仔细研究一下这句话中的一些重要单词：
- Bright：这个词的意思是 "明亮的"，通常用来形容光线或色彩鲜艳的事物。例如：The sun is bright today.（今天阳光明媚。）
- Precious：这个词的意思是 "珍贵的"，通常用来形容非常重要或有价值的东西。例如：Family is a precious thing.（家庭是一件珍贵的事情。）
- Fade：这个词的意思是 "逐渐消失"，通常用来形容颜色、光线或声音逐渐减弱或消失。例如：The flowers faded quickly in the hot sun.（花在烈日下很快就枯萎了。）
- Come back：这个词组的意思是 "回来"，通常用来表示某件事物返回原来的地方或状态。例如：I hope my lost wallet will come back to me.（我希望我丢失的钱包能回到我身边。）
这些单词在英语中都是非常常见的，可以应用在不同的场合中。希望这些例子能够帮助你更好地理解这些单词的意思和用法。
```

