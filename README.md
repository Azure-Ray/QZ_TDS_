<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>deeplearning4j-example</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-core</artifactId>
            <version>1.0.0-M1.1</version>
        </dependency>
        <dependency>
            <groupId>org.nd4j</groupId>
            <artifactId>nd4j-native-platform</artifactId>
            <version>1.0.0-M1.1</version>
        </dependency>
        <dependency>
            <groupId>org.datavec</groupId>
            <artifactId>datavec-api</artifactId>
            <version>1.0.0-M1.1</version>
        </dependency>
        <dependency>
            <groupId>org.datavec</groupId>
            <artifactId>datavec-local</artifactId>
            <version>1.0.0-M1.1</version>
        </dependency>
    </dependencies>
</project>


import org.datavec.api.records.reader.RecordReader;
import org.datavec.api.records.reader.impl.csv.CSVRecordReader;
import org.datavec.api.split.FileSplit;
import org.nd4j.linalg.dataset.api.iterator.DataSetIterator;
import org.nd4j.linalg.dataset.api.iterator.recordreader.RecordReaderDataSetIterator;
import org.nd4j.linalg.dataset.DataSet;
import org.nd4j.linalg.dataset.api.preprocessor.NormalizerMinMaxScaler;

import java.io.File;

public class DataPreparation {
    public static DataSetIterator loadData(String filePath, int batchSize, int labelIndex, int numClasses) throws Exception {
        int numLinesToSkip = 1;
        char delimiter = ',';
        RecordReader recordReader = new CSVRecordReader(numLinesToSkip, delimiter);
        recordReader.initialize(new FileSplit(new File(filePath)));

        DataSetIterator iterator = new RecordReaderDataSetIterator(recordReader, batchSize, labelIndex, numClasses);
        
        // 归一化数据
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
        int numInputs = 1; // 只有一个特征，即当前指数值
        int numOutputs = 1; // 预测一个值，即未来的指数值
        int numHiddenNodes = 50;
        int batchSize = 64;
        int numEpochs = 100;
        int seed = 123;

        // 加载数据集
        DataSetIterator trainData = DataPreparation.loadData("path/to/your/hsi-data.csv", batchSize, 3, 1);

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
        DataSetIterator testData = DataPreparation.loadData("path/to/your/test-data.csv", 1, 3, 1);

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
