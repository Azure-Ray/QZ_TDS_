<!DOCTYPE html>
<html>
<head>
    <title>Waiting Page</title>
    <script>
        function checkCookie() {
            if (document.cookie.indexOf("auth_token=") >= 0) {
                window.location.href = "/bulk-approval-ss0"; // 重定向回原始检查页面或最终目标页面
            } else {
                setTimeout(checkCookie, 1000); // 每1秒检查一次
            }
        }

        // 页面加载完毕时开始检查
        window.onload = checkCookie;
    </script>
</head>
<body>
    <p>Checking your authentication status, please wait...</p>
</body>
</html>
