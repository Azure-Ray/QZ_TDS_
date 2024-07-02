import org.datavec.api.records.reader.RecordReader;
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.api.split.FileSplit;
import org.datavec.api.transform.schema.Schema;
import org.datavec.api.transform.TransformProcess;
import org.datavec.local.transforms.LocalTransformExecutor;
import org.datavec.api.writable.Writable;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.dataset.api.iterator.recordreader.RecordReaderDataSetIterator;
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerMinMaxScaler;
import org.datavec.api.transform.schema.Schema.Builder;

import java.io.File;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;
import java.util.Locale;

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
                .transform(new org.datavec.api.transform.transform.string.StringToTimeTransform("date", "yyyy-MM-dd", Locale.ENGLISH))
                .transform(new org.datavec.api.transform.transform.string.StringToTimeTransform("time", "HH:mm:ss", Locale.ENGLISH))
                .build();

        RecordReader recordReader = new CSVRecordReader(numLinesToSkip, delimiter);
        recordReader.initialize(new FileSplit(new File(filePath)));

        // Read the data into a List<List<Writable>>
        List<List<Writable>> originalData = recordReader.next();

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


import org.deeplearning4j.nn.conf.NeuralNetConfiguration;
import org.deeplearning4j.nn.conf.layers.LSTM;
import org.deeplearning4j.nn.conf.layers.RnnOutputLayer;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.optimize.listeners.ScoreIterationListener;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.lossfunctions.LossFunctions;
import org.nd4j.linalg.learning.config.Adam;
import org.deeplearning4j.util.ModelSerializer;

import java.io.File;

public class LSTMModel {
    public static void main(String[] args) throws Exception {
        int numInputs = 7; // 7个特征：date, time, current, turnover, high, low, change, percent
        int numOutputs = 1; // 预测一个值，即未来的指数值
        int numHiddenNodes = 50;
        int batchSize = 64;
        int numEpochs = 100;
        int seed = 123;

        // 加载数据集
        DataSetIterator trainData = DataPreparation.loadData("path/to/your/hsi-data.csv", batchSize, 0, 1);

        // 构建LSTM神经网络
        MultiLayerNetwork model = new MultiLayerNetwork(new NeuralNetConfiguration.Builder()
                .seed(seed)
                .updater(new Adam(0.01))
                .list()
                .layer(new LSTM.Builder().nIn(numInputs).nOut(numHiddenNodes)
                        .activation(Activation.TANH)
                        .build())
                .layer(new LSTM.Builder().nIn(numHiddenNodes).nOut(numHiddenNodes)
                        .activation(Activation.TANH)
                        .build())
                .layer(new RnnOutputLayer.Builder(LossFunctions.LossFunction.MSE)
                        .activation(Activation.IDENTITY)
                        .nIn(numHiddenNodes).nOut(numOutputs)
                        .build())
                .build()
        );

        model.init();
        model.setListeners(new ScoreIterationListener(10));

        // 训练模型
        for (int epoch = 0; epoch < numEpochs; epoch++) {
            model.fit(trainData);
            System.out.println("Epoch " + epoch + " complete. Loss: " + model.score());
        }

        // 保存模型
        File locationToSave = new File("lstm-model.zip");
        boolean saveUpdater = true;
        ModelSerializer.writeModel(model, locationToSave, saveUpdater);
    }
}


import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.deeplearning4j.util.ModelSerializer;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.dataset.DataSet;

import java.io.File;

public class PredictSellPoint {
    public static void main(String[] args) throws Exception {
        // 加载训练好的模型
        File modelFile = new File("lstm-model.zip");
        MultiLayerNetwork model = ModelSerializer.restoreMultiLayerNetwork(modelFile);

        // 加载第三个月的数据
        DataSetIterator testData = DataPreparation.loadData("path/to/your/test-data.csv", 1, 0, 1);

        double maxPrice = Double.MIN_VALUE;
        while (testData.hasNext()) {
            DataSet dataSet = testData.next();
            INDArray input = dataSet.getFeatures();
            INDArray output = model.output(input);
            double currentPrice = output.getDouble(0);
            if (currentPrice > maxPrice) {
                maxPrice = currentPrice;
            }
        }
        System.out.println("最高卖出点: " + maxPrice);
    }
}
