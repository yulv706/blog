---
{"dg-publish":true,"dg-home":true,"dg-path":"home.md","permalink":"/home/","tags":["gardenEntry"],"dgPassFrontmatter":true}
---



<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>有趣的动态背景</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
        }

        .background {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background: linear-gradient(120deg, #f6d365, #fda085);
            background-size: 200% 200%;
            animation: gradientAnimation 10s ease infinite;
        }

        @keyframes gradientAnimation {
            0% {
                background-position: 0% 50%;
            }
            50% {
                background-position: 100% 50%;
            }
            100% {
                background-position: 0% 50%;
            }
        }

        .center-text {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 3rem;
            color: #fff;
            text-shadow: 2px 2px 10px rgba(0, 0, 0, 0.5);
            font-family: 'Arial', sans-serif;
        }
    </style>
</head>
<body>
    <div class="background"></div>
    <div class="center-text">欢迎来到我的博客!</div>
</body>
</html>

