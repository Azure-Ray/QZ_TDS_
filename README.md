import java.sql.*;
import java.time.LocalDateTime;
import java.util.Collections;
import java.util.Comparator;
import java.util.Map;

public class DataSyncService {

    public void syncDataBasedOnUpdateTime(Connection fromConn, Connection toConn, String fromTable, String toTable, Map<String, ColumnItem> meta) throws SQLException {
        toConn.setAutoCommit(false);

        var getLastUpdateTime = "SELECT max(update_time) FROM %s".formatted(toTable);
        var getNewRecords = "SELECT * FROM %s WHERE update_time > ?".formatted(fromTable);

        LocalDateTime lastUpdateTime = getLastUpdateTime(toConn, getLastUpdateTime);

        try (var stmtFrom = fromConn.prepareStatement(getNewRecords)) {
            stmtFrom.setTimestamp(1, Timestamp.valueOf(lastUpdateTime));
            var rs = stmtFrom.executeQuery();

            var columns = meta.entrySet().stream()
                              .sorted(Comparator.comparingInt(entry -> entry.getValue().getIndex()))
                              .map(Map.Entry::getKey)
                              .toList();
            var fields = String.join(",", columns);
            var questionMarks = String.join(",", Collections.nCopies(columns.size(), "?"));
            var insertQuery = "INSERT INTO %s (%s) VALUES (%s) ON CONFLICT (id) DO UPDATE SET %s"
                    .formatted(toTable, fields, questionMarks, getUpdateSetClause(columns));

            try (var writeStmt = toConn.prepareStatement(insertQuery)) {
                int count = 0;
                while (rs.next()) {
                    for (int i = 0; i < columns.size(); i++) {
                        var key = columns.get(i);
                        var item = meta.get(key);
                        getSetItem(i + 1, item.getType(), rs, writeStmt);
                    }
                    writeStmt.addBatch();
                    if (++count % 500 == 0) {
                        writeStmt.executeBatch();
                        toConn.commit();
                    }
                }
                if (count % 500 != 0) {
                    writeStmt.executeBatch();
                    toConn.commit();
                }
            }
        }
    }

    private LocalDateTime getLastUpdateTime(Connection toConn, String query) throws SQLException {
        try (var stmt = toConn.prepareStatement(query)) {
            var rs = stmt.executeQuery();
            if (rs.next()) {
                Timestamp timestamp = rs.getTimestamp(1);
                return timestamp != null ? timestamp.toLocalDateTime() : LocalDateTime.MIN;
            }
        }
        return LocalDateTime.MIN;
    }

    private String getUpdateSetClause(java.util.List<String> columns) {
        return columns.stream()
                      .map(column -> "%s = EXCLUDED.%s".formatted(column, column))
                      .collect(java.util.stream.Collectors.joining(", "));
    }

    private void getSetItem(int parameterIndex, Type type, ResultSet rs, PreparedStatement writeStmt) throws SQLException {
        // Implement based on ColumnItem's type to set the value on writeStmt
    }

    // Assume ColumnItem class exists and has methods like getType() and getIndex()
    class ColumnItem {
        // Implementation
    }
}
