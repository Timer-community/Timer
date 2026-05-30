Аренда сайтов. Внимание чтобы арендовать пишите сюда-> timerkhan.akhmetzyanov@mail.ru.А ещё на почту можно оправить просьбу на заказ курсов по програмированию.
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Рекламный блок</title>
    <style>
        .ad-container {
            position: relative;
            width: 600px;
            height: 400px;
            border: 2px dashed #999;
            margin: 50px auto;
            background-color: #f9f9f9;
        }

        .corner-top-left {
            position: absolute;
            top: 0;
            left: 0;
            width: 150px;
            height: 100px;
            border-top: 3px dashed #000;
            border-left: 3px dashed #000;
        }

        .corner-bottom-right {
            position: absolute;
            bottom: 0;
            right: 0;
            width: 150px;
            height: 100px;
            border-bottom: 3px dashed #000;
            border-right: 3px dashed #000;
        }

        .ad-text {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-family: Arial, sans-serif;
            font-size: 18px;
            color: #333;
            text-align: center;
            white-space: nowrap;
        }
    </style>
</head>
<body>
    <div class="ad-container">
        <div class="corner-top-left"></div>
        <div class="corner-bottom-right"></div>
        <div class="ad-text">Здесь могла быть ваша реклама или сайт</div>
    </div>
</body>
</html>
