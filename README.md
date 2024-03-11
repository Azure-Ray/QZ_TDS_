public static boolean containsSubstring(List<String> list, String target) {
        // 移除目标字符串中的 "ff"（如果存在）
        String normalizedTarget = target.replace(" ff", "").replace("ff", "");
        
        // 检查列表中是否有字符串包含处理过的目标字符串
        return list.stream().anyMatch(item -> 
                item.replace(" ff", "").replace("ff", "").contains(normalizedTarget));
    }
