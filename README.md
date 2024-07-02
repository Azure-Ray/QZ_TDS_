import org.datavec.api.records.reader.RecordReader;
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.api.split.FileSplit;
import org.datavec.api.writable.Writable;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.dataset.api.iterator.recordreader.RecordReaderDataSetIterator;
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerMinMaxScaler;
import org.datavec.api.records.reader.impl.collection.CollectionRecordReader;
import org.datavec.api.writable.DoubleWritable;
import org.datavec.api.writable.Text;

import java.io.File;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class DataPreparation {
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
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        for (List<Writable> record : originalData) {
            String status = record.get(2).toString();
            if ("T".equals(status)) {
                String dateStr = record.get(0).toString();
                String timeStr = record.get(1).toString();
                String datetimeStr = dateStr + " " + timeStr;
                Date date = dateFormat.parse(datetimeStr);
                double timestamp = date.getTime();

                List<Writable> newRecord = new ArrayList<>();
                newRecord.add(new DoubleWritable(timestamp)); // Add the timestamp as the first feature
                for (int i = 3; i < record.size(); i++) { // Skip the first three columns (date, time, status)
                    Writable value = record.get(i);
                    if (value.toString().isEmpty()) {
                        newRecord.add(new DoubleWritable(0)); // Replace empty values with 0
                    } else {
                        newRecord.add(value);
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
