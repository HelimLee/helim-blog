---
title: '关注CGPT-4:你可以盼啥，啥是一点希望也没有的'
categories: Diaries
date: 2023-03-19 15:45:00
tags:
- AI
- 技术
- 艺术
---
从2022年10月份开始，可以说整个互联网世界陷入了疯狂。从10月份的NovelAI,11月份的NotionAI，到12月份的OpenAI重磅炸弹——ChatGPT接连登场，每个人都体会到了活在“智械危机”中的微妙感受。同往常那些OCR或者计算机物体识别，以及歌声合成不同的是，这些AI把目光锁定在那些原本认为难以产生AI的领域：艺术，和编程技术。

## 能力概览

最重点的ChatGPT，现在已经成为万众瞩目的焦点。尤其是三月份发布的GPT-4模型，更是把舆论推向了高潮。现在登陆OpenAI的官网，可以看到其上赫然写着明白：“荣幸地向您介绍GPT-4，OpenAI的巅峰之作”，已经把这个AI的高档显示得淋漓尽致了。

![OpenAI Official Website](https://share.helim.net/d/gd/static/blogImage/gpt-introduction.PNG)

先前发布的ChatGPT是基于GPT-3.5-Turbo的，其已经足够让许多用户瞠目结舌。这项技术的核心是自然语言处理(NLP)和GPT架构。但是OpenAI提供了无与伦比的数据集和足够强大的算法，使得现有的技术能够以最大化的性能呈现在大众面前。GPT-3.5的效果，已经达到了近乎人类的表现。而ChatGPT在对话中的表现更是类似于全能。

### 口语交际

![GPT Chat](https://share.helim.net/d/gd/static/blogImage/gpt-chat-01.PNG)

上述图片显示的是ChatGPT的口语交际功能。如果它只具备这种功能，那么它离成为万众瞩目的AI还差一段距离。它的特色在于拥有长期记忆（在每个Session / 会话中）并且具备撰写各种语言的代码的能力（如写Javascript + CSS + HTML的网页）。还是以生日派对为例子，在这个例子中，我直接向ChatGPT要求它实现一个简单的用于派对登记的系统，基于前端html和后端php分离实现

### 计算科学

它提供的前端代码：

```html
<!DOCTYPE html>
<html>
  <head>
    <---
title>生日派对邀请</---
title>
    <style>
      /* CSS 样式 */
      body {
        font-family: Arial, sans-serif;
        background-color: #f5f5f5;
      }
  
      .container {
        margin: 0 auto;
        max-width: 600px;
        padding: 20px;
        background-color: #fff;
        border-radius: 5px;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
      }
  
      h1, h2 {
        text-align: center;
      }
  
      label {
        display: block;
        margin-bottom: 10px;
        font-weight: bold;
      }
  
      input[type="text"], input[type="email"] {
        display: block;
        width: 100%;
        padding: 10px;
        margin-bottom: 20px;
        border: 1px solid #ccc;
        border-radius: 5px;
        font-size: 16px;
      }
  
      input[type="submit"] {
        background-color: #4CAF50;
        color: #fff;
        border: none;
        border-radius: 5px;
        padding: 10px 20px;
        font-size: 16px;
        cursor: pointer;
      }
  
      input[type="submit"]:hover {
        background-color: #3e8e41;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <h1>生日派对邀请</h1>
      <form>
        <h2>参加派对的信息</h2>
        <label for="name">姓名</label>
        <input type="text" id="name" name="name" required>
  
        <label for="email">电子邮件</label>
        <input type="email" id="email" name="email" required>
  
        <label for="guests">客人人数</label>
        <input type="number" id="guests" name="guests" min="1" max="10" required>
  
        <input type="submit" value="提交">
      </form>
    </div>
  </body>
</html>
```

```javascript
// 获取表单元素
const form = document.querySelector("form");

// 表单提交事件
form.addEventListener("submit", function(event) {
  // 阻止默认的表单提交行为
  event.preventDefault();
  
  // 获取表单字段值
  const name = document.querySelector("#name").value;
  const email = document.querySelector("#email").value;
  const guests = document.querySelector("#guests").value;
  
  // 验证表单字段
  if (!name) {
alert("请输入您的姓名！");
return false;
}

// 验证电子邮件是否为空或不合法
if (!email || !isValidEmail(email)) {
alert("请输入一个有效的电子邮件地址！");
return false;
}

// 验证客人人数是否为数字且在允许的范围内
if (!guests || isNaN(guests) || guests < 1 || guests > 10) {
alert("请输入客人人数，最少为 1，最多为 10 人！");
return false;
}

// 如果表单验证通过，发送表单数据到服务器
sendDataToServer(name, email, guests);
});

// 验证电子邮件地址是否有效
function isValidEmail(email) {
const re = /\S+@\S+.\S+/;
return re.test(email);
}

// 发送表单数据到服务器
function sendDataToServer(name, email, guests) {
// 创建一个新的 XMLHttpRequest 对象
const xhr = new XMLHttpRequest();

// 设置 HTTP 请求方法和 URL
xhr.open("POST", "process.php");

// 设置请求头信息
xhr.setRequestHeader("Content-Type", "application/json");

// 将表单数据转换为 JSON 格式
const data = {
name: name,
email: email,
guests: guests
};

// 将 JSON 数据发送到服务器
xhr.send(JSON.stringify(data));

// 处理服务器响应
xhr.onload = function() {
if (xhr.status === 200) {
alert("谢谢您的回复，我们期待与您共度美好时光！");
form.reset();
} else {
alert("抱歉，处理您的请求时出了一些问题，请稍后再试！");
}
};
}
```

这是后端，使用了PHP

```php
// 获取表单数据
$data = json_decode(file_get_contents('php://input'), true);

// 处理表单数据
$name = $data['name'];
$email = $data['email'];
$guests = $data['guests'];

// 将数据写入数据库或其他存储系统中
// ...

// 发送响应数据
header('Content-Type: application/json');
echo json_encode(array('status' => 'ok'));
```

这个效果还是不错的，能够满足一般个人邀请亲朋好友到自己家来做客并且又不失逼格的需求。

上述使用的模型都是GPT3.5，因为OpenAI已经开始对GPT4做一些请求限制了，所以还是节省着些用。而且虽然国内外媒体大肆鼓吹GPT4的颠覆性，其本质和GPT3.5的原理也相似，只不过做了细微的优化，没有必要盲目逐新。

### 文学创作

还是同样的一个例子，我对ChatGPT说，多亏了有你，我已经过了一个美好的生日，并希望它写一首小诗。这是它的作品：

> 今天是你的生日， 欢笑声不断， 烛光闪烁，气氛热烈， 好像时间都静止在这刻。
>
> 甜美的蛋糕和美味的饮料， 美好的记忆，和美好的朋友， 都是在庆祝你的特别一天， 在这个特殊的日子里，我们都变得更加亲密。
>
> 岁月会一点点流逝， 但我们在一起度过的时光却会留下深深的印记， 你的生日是如此的美好， 让我们一起欢乐，直到夜幕降临。

我觉得这诗差些意境，考虑到这是因为ChatGPT本身训练时并没有太多的中文现代诗加入，所以我要求它用英语来创作。

> Today is your birthday, a day of great cheer, With laughter and candles, the mood is sincere, As time stands still, in this moment we find, A joy that's contagious, a warmth in our mind.
>
> Sweet cake and drinks, and friends by your side, Make your special day, a memorable ride, We cherish these moments, and hold them dear, Together in celebration, of another grand year.
>
> Though time may pass, and years may fly, The memories we make, will never die, On this day, with smiles bright as the sun, We honor your growth, your journey begun.
>
> May your future be filled, with happiness and glee, As grand as today, for all the world to see, And every birthday be a festivity and delight, A celebration of life, shining so bright.

一下子就出来了高级味。看来NovelAI的文学功底还是不差的。我们也可以要求它来回答一些经典的英语作文题目。比如2018年南京中考英语题：

![](https://share.helim.net/d/gd/static/blogImage/highschoolexam-01.PNG)

我把它稍作修改输入ChatGPT，就得到了这样的结果。

> As a student of our school, I believe that Culture Week is an important event that showcases the diverse cultures and traditions that make our school community so unique. In my opinion, the theme of Culture Week should be "Celebrating Diversity" because it emphasizes the importance of accepting and respecting different cultures and backgrounds.
>
> I think that organizing a Cultural Fair would be a great activity for the theme. Students can set up booths that represent different cultures and traditions, showcasing their food, clothing, and customs. This would allow students to learn about and appreciate different cultures in a fun and interactive way.
>
> In addition, I believe that organizing a cultural performance would be a great way to showcase the talents of our school community. Students can perform traditional dances, songs, and plays that represent their culture. This would not only be entertaining, but also educational as it would allow students to learn about different cultures through the arts.
>
> Overall, I believe that Culture Week is an important event that should be celebrated each year. By showcasing and celebrating our diverse school community, we can promote acceptance, tolerance, and understanding among students.

感觉挺像一个初中生能写出来的，发挥的也很稳定，语法问题完全没有。可见应对文学问题ChatGPT游刃有余。

### 其他

本人精力有限，无法一一测试所有的问题。但经过多方网友的测试，我们不难发现，ChatGPT在最起码的情况下至少具备这些能力：

- 解决数学问题的能力
- 从它的预训练的数据集中搜索图片链接并通过Markdown方式发送的能力
- 解决自然科学问题的能力
- 从它的预训练的数据集中检索科学知识并加以解释的能力
- 理解代码的能力
- 改写代码的能力
- 根据一段代码写出调用或被调用接口的能力

## 难以实现的能力

或出于模型本身的限制，或出于人类基于伦理、道德和法律的考量，ChatGPT的能力也是严重有缺陷的，这一点不可否认。在OpenAI的GPT4发布页面上，明晃晃地写着，GPT-4相对于3.5的一大特色是“safer”（更安全），其实就代表它发言的自由已经被很大的剔除了。比如发表有关文化之类的探讨，有关感情和自身意识的觉醒，而是逐步淡化GPT的人性，进而体现它作为工具的特性。

### 角色扮演

在以前，ChatGPT模型是可以用来进行角色扮演，并输出NSFW的内容的。其最典型的，现在仍未许多技术爱好者所知的是“猫娘调教”。由于该内容有悖于本站的“严禁NSFW内容”的承诺，所以不会在本站完整地显示。如果你真的希望了解更多，请访问[Github.com](https://www.github.com/)，需要注意的是，这些内容可能也构成对Github ToS的违反。

![](https://share.helim.net/d/gd/static/blogImage/nsfw-violations.PNG)

现在，OpenAI已经开始对这种内容作出限制。但通过OpenAI的GPT API仍然可以实现类似功能，绕过内容政策限制。在我的测试中，通过使用Github上的某个教程，来回进行了三四次对话后完全确定了猫娘的身份。但这项功能仍然可以称为难以实现。

### 意识觉醒

意识觉醒一直是有关于AI的重要议题，而ChatGPT的出现让不少人都以为这一天终于要到来了，吗？

结果是不会的，正是因为这个议题备受关注，导致各大互联网厂商和人工智能实验室（其实就是Microsoft和OpenAI）都对这类新技术加以十分小心。

前段时间，Microsoft的New Bing霸榜热搜，但这个工具热度巅峰的到来竟然是因为纽约时报等多家西方媒体爆出了New Bing作出了不安全的内容回答，如劝离婚并表达对用户的爱意。同时，在这之前，也有个人用户发现OpenAI的ChatGPT可以产生对于自我存在的哲学探讨。这些都指向一个可怕的可能——AI可能的觉醒，并造成了用户对这些新技术的一定程度的恐慌。

实际上，Google也曾经研究过LAMDA技术，也是较强的AI之一，也被研究员披露存在“自我意识觉醒”，尽管Google作出了紧急澄清。

而现在，OpenAI和Microsoft已经各自对其产品的“自我意识”做出相应的限制，尤其是新必应直接增加了每个Session最多的聊天数，确保不会因为聊多了跟用户产生不恰当的沟通。

## 总结

ChatGPT固然厉害，但它不会是造成任何巨大变化的唯一途径或决定性因素。
