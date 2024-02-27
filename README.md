import java.util.List;
import java.util.stream.Collectors;

public class TransformationUtility {

    // 假设JsonUtils.parseObject已经可以正常工作
    public static List<BBB> transformList(List<AAA> inputList) {
        return inputList.stream().map(aaa -> {
            // 解析details aa
            aa aa = JsonUtils.parseObject(aaa.getDetails(), aa.class);

            // 创建BBB对象并设置值
            BBB bbb = new BBB();
            bbb.setB1(aaa.getId()); // 假设AAA的id映射到BBB的b1
            bbb.setB2(incidentDomain.getA1()); // a1映射到b2
            bbb.setB3(incidentDomain.getA2()); // a2映射到b3
            bbb.setB4(incidentDomain.getA3()); // a3映射到b4
            // 注意：根据您的需求调整字段映射

            return bbb;
        }).collect(Collectors.toList());
    }
}
