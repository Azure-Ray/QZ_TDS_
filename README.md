import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.IntStream;


public class ApprovalChecker {

    private static final int PAGE_SIZE = 20;

    private static void checkApprovalStatusesByGroup(String group, List<String> crNumbers, Map<String, List<String>> approvedCrMap) {
        IntStream.range(0, (crNumbers.size() + PAGE_SIZE - 1) / PAGE_SIZE)
                .forEach(i -> {
                    int start = i * PAGE_SIZE;
                    int end = Math.min((i + 1) * PAGE_SIZE, crNumbers.size());
                    List<String> page = crNumbers.subList(start, end);
                    
                    String crNumbersParam = String.join(",", page);
                    
                    Map<String, String> params = new HashMap<>();
                    params.put("crNumbers", crNumbersParam); // 假设API接受"crNumbers"参数
                    String url = HttpUtil.buildGetUrl(snowApiHost, "/snow/approvals", params);
                    String respBody = HttpUtil.simpleGet(url);
                    List<ChangeApprovalsReadResponseModel> approvals = JacksonUtils.toObj(respBody, new TypeReference<>() {});
                    
                    approvals.stream()
                            .filter(approval -> "approved".equals(approval.getState()))
                            .forEach(approval -> approvedCrMap.computeIfAbsent(group, k -> new ArrayList<>()).add(approval.getGroup()));
                });
    }

    public static void filterUnapproved(Map<String, List<ApprovaCrInfo>> changesByGroup) {
        Map<String, List<String>> approvedCrMap = new HashMap<>();
        
        changesByGroup.forEach((group, crInfos) -> {
            List<String> crNumbers = crInfos.stream()
                                            .map(ApprovaCrInfo::getSnCrNumber)
                                            .collect(Collectors.toList());
            checkApprovalStatusesByGroup(group, crNumbers, approvedCrMap);
        });

        approvedCrMap.forEach((group, approvedCrs) -> {
            List<ApprovaCrInfo> crInfos = changesByGroup.get(group);
            if (crInfos != null) {
                crInfos.removeIf(crInfo -> approvedCrs.contains(crInfo.getSnCrNumber()));
                if (crInfos.isEmpty()) {
                    changesByGroup.remove(group);
                }
            }
        });
    }

}
