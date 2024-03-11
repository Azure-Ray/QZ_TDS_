import java.util.*;

public class ApprovalCheckerTest {

    public static void main(String[] args) {
        // 模拟的changesByGroup数据
        Map<String, List<ApprovaCrInfo>> changesByGroup = new HashMap<>();
        changesByGroup.put("Group1", new ArrayList<>(List.of(new ApprovaCrInfo("CR1"), new ApprovaCrInfo("CR2"), new ApprovaCrInfo("CR3"))));
        changesByGroup.put("Group2", new ArrayList<>(List.of(new ApprovaCrInfo("CR4"), new ApprovaCrInfo("CR5"))));
        
        // 模拟的approvedCrMap数据，假设"CR2"和"CR4"被批准
        Map<String, List<String>> approvedCrMap = new HashMap<>();
        approvedCrMap.put("Group1", new ArrayList<>(List.of("CR2")));
        approvedCrMap.put("Group2", new ArrayList<>(List.of("CR4")));

        // 移除被批准的CRs
        approvedCrMap.forEach((group, approvedCrs) -> {
            List<ApprovaCrInfo> crInfos = changesByGroup.get(group);
            if (crInfos != null) {
                crInfos.removeIf(crInfo -> approvedCrs.contains(crInfo.getSnCrNumber()));
                if (crInfos.isEmpty()) {
                    changesByGroup.remove(group);
                }
            }
        });

        // 打印最终的changesByGroup映射以查看结果
        changesByGroup.forEach((group, crInfos) -> {
            System.out.println("Group: " + group);
            crInfos.forEach(crInfo -> System.out.println(" - CR: " + crInfo.getSnCrNumber()));
        });
    }

    // 内部类来模拟ApprovaCrInfo
    public static class ApprovaCrInfo {
        private String snCrNumber;

        public ApprovaCrInfo(String snCrNumber) {
            this.snCrNumber = snCrNumber;
        }

        public String getSnCrNumber() {
            return snCrNumber;
        }
    }
}
