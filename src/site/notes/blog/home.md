---
{"dg-publish":true,"dg-home":true,"dg-path":"home.md","permalink":"/home/","tags":["gardenEntry"],"dgPassFrontmatter":true}
---


这是主页，好吧，现在还是空的

目前还不知道这里放些什么好

这个花园初步计划打算用来上传我学习过程中整理的笔记。
但是现在由于各种原因，感觉弄出来的效果没有达到我的预期
等我在学习学习如何整理笔记，等我有一套成熟的笔记系统的时候再进行更新吧

所以现在这个主页先让他闲置吧

<?php
// 名人名言数组
$quotes = [
    "“The only way to do great work is to love what you do.” – Steve Jobs",
    "“Life is what happens when you're busy making other plans.” – John Lennon",
    "“Get busy living or get busy dying.” – Stephen King",
    "“You have within you right now, everything you need to deal with whatever the world can throw at you.” – Brian Tracy",
    "“The purpose of our lives is to be happy.” – Dalai Lama",
    "“Be yourself; everyone else is already taken.” – Oscar Wilde",
    "“In three words I can sum up everything I've learned about life: it goes on.” – Robert Frost",
    "“To be yourself in a world that is constantly trying to make you something else is the greatest accomplishment.” – Ralph Waldo Emerson",
    "“You must be the change you wish to see in the world.” – Mahatma Gandhi",
    "“The best way to predict the future is to create it.” – Peter Drucker"
];

// 随机选择一句名言
$quote_of_the_day = $quotes[array_rand($quotes)];

// 输出到网页
echo "<div style='font-size: 1.5em; color: #555; margin: 20px; padding: 10px; border-left: 5px solid #eee;'>";
echo "<p>Today's Quote:</p>";
echo "<p><em>\"$quote_of_the_day\"</em></p>";
echo "</div>";
?>




