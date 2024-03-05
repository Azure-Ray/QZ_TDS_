@GetMapping("/check-token")
public ResponseEntity<String> checkToken(@CookieValue(name = "auth_token", required = false) String authToken) {
    if (authToken != null && !authToken.isEmpty()) {
        return ResponseEntity.ok("token_exists");
    } else {
        return ResponseEntity.ok("no_token");
    }
}
<script>
    function checkToken() {
        fetch("/check-token")
            .then(response => response.text())
            .then(data => {
                if (data === "token_exists") {
                    window.location.href = "/your-final-destination.html"; // Token存在，重定向到最终目标
                } else {
                    setTimeout(checkToken, 1000); // Token不存在，等待1秒后再次检查
                }
            });
    }

    // 页面加载完毕时开始检查
    window.onload = checkToken;
</script>
