import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDateTime;
import java.util.Iterator;
import java.util.List;

@Service
public class OperationLogService {

    @Autowired
    private OperationLogRepository operationLogRepository;

    @Transactional
    public void processIncidents(List<Incident> incidents, String operationType) {
        Iterator<Incident> iterator = incidents.iterator();
        
        while (iterator.hasNext()) {
            Incident incident = iterator.next();
            // 检查该IncidentId是否已经记录过指定的操作类型
            List<OperationLog> existingLogs = operationLogRepository.findByEntityIDAndOperationType(incident.getIncidentId(), operationType);
            
            if (existingLogs.isEmpty()) {
                // 如果不存在，插入新记录
                OperationLog newLog = new OperationLog();
                newLog.setEntityType("Incident");
                newLog.setEntityID(incident.getIncidentId());
                newLog.setOperationType(operationType);
                newLog.setStatus("Success"); // 或根据实际情况设置
                newLog.setOperationDetails(""); // 根据需要设置详细信息
                newLog.setCreatedAt(LocalDateTime.now());
                newLog.setUpdatedAt(LocalDateTime.now());
                operationLogRepository.save(newLog);
            } else {
                // 如果存在，从列表中移除
                iterator.remove();
            }
        }
    }

    // Incident类定义省略，假定其有getIncidentId方法
}
