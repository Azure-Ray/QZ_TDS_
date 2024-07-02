import org.datavec.api.records.reader.RecordReader;
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.api.split.FileSplit;
import org.datavec.api.transform.schema.Schema;
import org.datavec.api.transform.TransformProcess;
import org.datavec.api.writable.Writable;
import org.datavec.local.transforms.LocalTransformExecutor;
import org.nd4j.common.io.ClassPathResource;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.dataset.api.iterator.recordreader.RecordReaderDataSetIterator;
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerMinMaxScaler;

import java.io.File;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class DataPreparation {
    public static DataSetIterator loadData(String filePath, int batchSize, int labelIndex, int numClasses) throws Exception {
        int numLinesToSkip = 1;
        char delimiter = ',';

        // Define the schema of the input data
        Schema inputDataSchema = new Schema.Builder()
                .addColumnString("date")
                .addColumnString("time")
                .addColumnString("status")
                .addColumnDouble("current")
                .addColumnDouble("turnover")
                .addColumnDouble("high")
                .addColumnDouble("low")
                .addColumnDouble("change")
                .addColumnDouble("percent")
                .build();

        // Define the transformation process
        TransformProcess tp = new TransformProcess.Builder(inputDataSchema)
                .filter(new org.datavec.api.transform.filter.ConditionFilter(
                        new org.datavec.api.transform.condition.column.StringColumnCondition("status", org.datavec.api.transform.condition.ConditionOp.Equal, "T")))
                .removeColumns("status") // 移除不需要的列
                .transform(new org.datavec.api.transform.transform.string.StringToTimeTransform("date", "yyyy-MM-dd"))
                .transform(new org.datavec.api.transform.transform.string.StringToTimeTransform("time", "HH:mm:ss"))
                .transform(new org.datavec.api.transform.transform.doubletransform.TimeMathOpTransform("timestamp", "date", "time", TimeMathOpTransform.MathOp.ADD)) // 将日期和时间列合并为时间戳
                .removeColumns("date", "time") // 移除原始日期和时间列
                .build();

        RecordReader recordReader = new CSVRecordReader(numLinesToSkip, delimiter);
        recordReader.initialize(new FileSplit(new File(filePath)));

        // Read the data into a List<List<Writable>>
        List<List<Writable>> originalData = new ArrayList<>();
        while (recordReader.hasNext()) {
            originalData.add(recordReader.next());
        }

        // Apply the transformation to the dataset
        List<List<Writable>> transformedData = LocalTransformExecutor.execute(originalData, tp);

        // Convert transformed data to DataSetIterator
        RecordReader transformedReader = new CollectionRecordReader(transformedData);

        DataSetIterator iterator = new RecordReaderDataSetIterator(transformedReader, batchSize, labelIndex, numClasses);

        // Normalize data
        NormalizerMinMaxScaler scaler = new NormalizerMinMaxScaler(0, 1);
        scaler.fit(iterator);
        iterator.setPreProcessor(scaler);

        return iterator;
    }
}
