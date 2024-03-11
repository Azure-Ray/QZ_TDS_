    public static boolean containsSubstring(List<String> list, String target) {
        // 针对没有"-ff"的情况，替换"ff"（如果存在），同时注意大小写
        String normalizedTarget = normalizeString(target);

        // 检查列表中是否有字符串在忽略"ff"和大小写的情况下包含处理过的目标字符串
        return list.stream().anyMatch(item ->
                normalizeString(item).contains(normalizedTarget));
    }

    private static String normalizeString(String input) {
        // 替换掉" ff"或"ff"，但不替换"-ff"
        String temp = input.replaceAll("(?i)(?<!-) ff", "").replaceAll("(?i)(?<!-)ff", "");
        // 返回小写版本以进行不区分大小写的匹配
        return temp.toLowerCase();
    }
