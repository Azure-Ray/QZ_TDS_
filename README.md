private boolean updateUrlIfNeeded(boolean aaa, String currentUrl, String newUrl) {
    if (aaa || currentUrl == null || currentUrl.isEmpty()) {
        if (newUrl != null && !newUrl.trim().isEmpty()) {
            return true;
        }
    }
    return false;
}

// Usage
if (updateUrlIfNeeded(, pipelineMapEntity.aaa(), extraCrMapEntity.aaa())) {
    
}

