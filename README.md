CREATE TABLE OperationLogs (
    ID SERIAL PRIMARY KEY,
    EntityType VARCHAR(255),
    EntityID VARCHAR(255),
    OperationType VARCHAR(255),
    Status VARCHAR(100),
    OperationDetails TEXT,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
package com.example.demo.entity;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "OperationLogs")
public class OperationLog {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String entityType;

    @Column(nullable = false)
    private String entityID;

    @Column(nullable = false)
    private String operationType;

    @Column(nullable = true)
    private String status;

    @Column(nullable = true, length = 10485760) // for large text
    private String operationDetails;

    @Column(nullable = false)
    private LocalDateTime createdAt = LocalDateTime.now();

    @Column(nullable = false)
    private LocalDateTime updatedAt = LocalDateTime.now();

    // Constructors, Getters and Setters
}
package com.example.demo.repository;

import com.example.demo.entity.OperationLog;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface OperationLogRepository extends JpaRepository<OperationLog, Long> {
}
