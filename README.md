import org.datavec.api.records.reader.RecordReader;
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.api.split.FileSplit;
import org.datavec.api.writable.Writable;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.dataset.api.iterator.recordreader.RecordReaderDataSetIterator;
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerMinMaxScaler;
import org.datavec.api.records.reader.impl.collection.CollectionRecordReader;
import org.datavec.api.writable.DoubleWritable;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class DataPreparation {
    private static final Logger log = LoggerFactory.getLogger(DataPreparation.class);

    public static DataSetIterator loadData(String filePath, int batchSize, int labelIndex, int numClasses) throws Exception {
        int numLinesToSkip = 1;
        char delimiter = ',';

        RecordReader recordReader = new CSVRecordReader(numLinesToSkip, delimiter);
        recordReader.initialize(new FileSplit(new File(filePath)));

        // Read the data into a List<List<Writable>>
        List<List<Writable>> originalData = new ArrayList<>();
        while (recordReader.hasNext()) {
            originalData.add(recordReader.next());
        }

        // Manually transform data: filter out non-T status and convert date/time to timestamp
        List<List<Writable>> transformedData = new ArrayList<>();
        SimpleDateFormat dateFormat = new SimpleDateFormat("M/d/yyyy HH:mm:ss");

        for (List<Writable> record : originalData) {
            String status = record.get(2).toString();
            if ("T".equals(status)) {
                String dateStr = record.get(0).toString().replace(" 0:00", ""); // Remove " 0:00"
                String timeStr = record.get(1).toString();
                String datetimeStr = dateStr + " " + timeStr;
                Date date = dateFormat.parse(datetimeStr);
                double timestamp = date.getTime();

                List<Writable> newRecord = new ArrayList<>();
                newRecord.add(new DoubleWritable(timestamp)); // Add the timestamp as the first feature
                for (int i = 3; i < record.size(); i++) { // Skip the first three columns (date, time, status)
                    Writable value = record.get(i);
                    if (value.toString().isEmpty() || value.toString().equals("null")) {
                        newRecord.add(new DoubleWritable(0)); // Replace empty or null values with 0
                    } else {
                        try {
                            double numericValue = Double.parseDouble(value.toString());
                            if (numericValue < 0 || numericValue > 1000000) { // Example threshold for abnormal values
                                log.warn("Abnormal value detected: {} at column {}", numericValue, i);
                                newRecord.add(new DoubleWritable(0)); // Replace abnormal values with 0
                            } else {
                                newRecord.add(new DoubleWritable(numericValue));
                            }
                        } catch (NumberFormatException e) {
                            log.warn("Non-numeric value detected: {} at column {}", value.toString(), i);
                            newRecord.add(new DoubleWritable(0)); // Replace non-numeric values with 0
                        }
                    }
                }
                transformedData.add(newRecord);
            }
        }

        // Create a RecordReader from the transformed data
        RecordReader finalReader = new CollectionRecordReader(transformedData);

        DataSetIterator iterator = new RecordReaderDataSetIterator(finalReader, batchSize, labelIndex, numClasses);

        // Normalize data
        NormalizerMinMaxScaler scaler = new NormalizerMinMaxScaler(0, 1);
        scaler.fit(iterator);
        iterator.setPreProcessor(scaler);

        return iterator;
    }
}
