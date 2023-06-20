---
title: 'Finding hidden coding problems'
categories: Diaries
date: 2023-03-21 22:15:00
tags:
- 技术
---
A badly coded document in a computer can lead to serious problem. It can turn a well-edited document file into a completely unreadable mess. It can also cause some coding sensitive script stop working.

## Type I: Starting with out-of-range

```python
fileRes=open("test.txt","r")
fileContent=fileRes.read()
```

This can may cause an exception detected by Python syntax checker, which is called "Parse Error". The excpetion is described as a "out-of-range" charset error detected in the file object. The "range" here it's what we called a codeset(charset). Basically, if a python program shutdown for a "range" exception, the range stands for ASCII. Which mostly contains latin letters and simple emojis only.

We can use a one-line code to solve the problem.

```python
fileRes=open("test.txt","r",decoding="utf-8")
```

In most of the conditions, forcely changing coding method can solve ASCII's out-of-range problem. On the other hand, if a problem cannot be solved via this, there will be more to consider of.

## Type II: Non-widely used CJK characeters appear in the output

Sometimes by forcely changing charset to utf-8 as default encoding&decoding will work and solve everything. But if it doesn't help, the output may turn out to be a complete mess with a lot of non-widely used CJK characeters (like Hanzi). This is especially common when dealing with data from computers in China(C) Japan(J) and Korea(K), because in these countries, even if they all using "Hanzi", they have small differences between each other's character systems. To fit all those differences and build a universal usable coding system, those countries' governments come up with different charsets and standards. Each one of those charsets can be found using in the Internet. This itself has already became a mess. What makes it even worse is the Unicode, the well-known organization in making universal avaliable charsets, decided to make all the CJK characeters' Hanzi look completely same, which cause the content provider have to use other ways to seperate different countries' special Hanzi systems, which leads to a stronger mess.

UTF-8 or similar large set of coding are really big, and they may contain bunches of characters that are really strange. That make it possible for a syntax checker to ignore some of the coding errors, because even if these characters' binary code doesn't match what it is supposed to look like, it is still in the range of UTF-8 or other large charset. When the output is printed, the computer will find some rarely used charaters and print them without asking or showing any warnings.

The only way to completely solve this coding problem, just like solving Time Zone conflicts by changing all the different Time Zone's times to a unique time, the UTC, is to make sure that the whole progress of data processing is under UTF-8 or UTF-16. From the creation of the document, to the print of ouput, all the coding method should be manually set to "UTF-8" or "UTF-16".

But if machines make mistakes, humans will make just more. To prevent future mistakes in cross-device coding, a few lines of code is required to detect if there is any strange character and throw out an exception to abort the program if it is necessary.

## Type III: "Block" character displayed in output

A "Block" character can also stand for another out-of-range problem. But this time the "range" represents Font (or `font-family`in CSS3). A font is an impletion of a charset, it should contain what every character in a charset look like. But the problem is, sometime, even if the charset do contain a character, the Font's designer hasn't design a look for that character. While using a font with less than enough characters, the computer will try to fill the blank of the charset with another pre-selected font, which is called "fallback" or "safe-font". And if the computer cannot find a suitable font at all, the blank will be filled with blocks.

This problem is bascially impossible for a program to discover, because only humans can see if the character is properly printed onto screen or paper. And also, the problem is widely seen when opening a program from CJK countries on western countries' computers, because they may not have fonts that contain CJK characters installed. As we all know, CJK charset is really large, and deleting them will create a lot of free disk space.(20MB)

If a program with block displaying issues runs in CLI environment, the issue will probably never be solved. Till now, most of the common Terminals won't give permission to a program to change the font in use. So it is recommend for CLI program to use only ASCII characters.

If a program runs in GUI or web environment, it is possible to take a font file with the program. Make sure the font match the chosen charset.

Here coms an example in HTML.

```html
<meta charset="utf-8">
<!--Setting the charset-->
<link href="${Link to the webfont}" rel="stylesheet">
<style>
body{
font-family: "${font-name}",${safe font for fallback};
}
</style>
```

The code above will only work properly with modern web browsers if a webfont of WOFF(2) is used, because only these browsers contains modern webfont supports. On browsers from ancient times, like Internet Explorer, the issue may still be presented if a OTF or TTF font is not prepared for a backup. TTF fonts have been supported since IE8 by most browsers. But a TTF font contains CJK charset is often large enough to slow the whole page's load latency.

## Conclusion

Encoding & decoding issues have been widely seen in the world of computer since charesets that contain more stuff than ASCII released, and both personeels and major IT or IIT companies are suffering from them. Discussions on whether to destroy all the charsets and make a completely world-wide compatable charset and font standard everywhere never stop. But what we can make sure is, that the difficulties in communications between different languages and cultures will never disappear, and they are not simple technical issues that can be solved easily. And the small or huge differences between different cultures are just something that make the world a more colorful place.
