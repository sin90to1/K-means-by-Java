# K-Means

姓名：石博佳

学号：2019170901023

## 软件说明

本文件为基于jdk的自动化K- Means算法演示器。

软件界面👇

![image-20220711071010150](README.assets/image-20220711071010150.png)

本软件左侧为用户交互区，右侧为可视化区。其中，用户交互区的各个按钮和文本框分别为：自定义K值，自定义遍历次数，自定义数据数量，随机生成数据，将数据可视化结果保存到文件，清空绘图区，可视化，聚类。

## 文件路径

> project
>
> > KMeans.java
> >
> > Image.png：生成的图片缓存
> >
> > Clustered.png：聚类后的可视化结果
> >
> > Unclustered.png：未聚类的可视化结果

## 软件演示及使用说明

首先需要自定义K值、循环次数、数据集大小，以防报错。此处将文本框限定为仅能输入数字不能输入其他内容。

![image-20220711072449626](README.assets/image-20220711072449626.png)

在输入超参数后，进行数据随机生成，生成完毕后会有对话框提示。请注意，**K不能大于10（可视化颜色限制）**。

![image-20220711072500520](README.assets/image-20220711072500520.png)

点击Save Result to Image，先保存可视化结果。此时可以看到脚本根目录中出现了Unclustered.png文件。此时点击Visualize可以可视化。

![image-20220711072533096](README.assets/image-20220711072533096.png)

之后便是生成聚类后的结果。首先点击Clear按钮清空画布，再点击Cluster进行聚类，单击Save保存，再单击Visualize进行可视化生成。

![image-20220711072621717](README.assets/image-20220711072621717.png)

## 代码实现

```java
/*
 * @Author: 石博佳
 * @Date: 2022-06-30 08:56:52
 * @LastEditTime: 2022-07-11 07:45:44
 * @LastEditors: 石博佳
 * @Description: This is the K-means implementation based on the JDK 
 * @FilePath: /Proj/KMeans.java
 */
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.io.File;
import java.io.IOException;
import java.awt.Dimension;
import java.awt.Color;
import java.awt.Font;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.BorderLayout;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.image.BufferedImage;
import javax.imageio.ImageIO;
import javax.swing.ImageIcon;
import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JLabel;
import javax.swing.JTextField;
import javax.swing.text.AttributeSet;
import javax.swing.text.PlainDocument;
import javax.swing.JOptionPane;

class NumberTextField extends PlainDocument {
    /**
     * @description: 限制文本框内容，使非数字文本无法键入。
     */    
    public NumberTextField() {
        super();
    }
    public void insertString(int offset, String str, AttributeSet attr)
            throws javax.swing.text.BadLocationException {
        if (str == null) {
            return;
        }
        char[] s = str.toCharArray();
        int length = 0;
        for (int i = 0; i < s.length; i++) {
            if ((s[i] >= '0') && (s[i] <= '9')) {
                s[length++] = s[i];
            }
        super.insertString(offset, new String(s, 0, length), attr);
        }
    }
}

class DataSet<T> {
    /**
     * @description: 数据集类，包含存储、访问、修改数据的一系列方法。
     */    
    int dataSize;
    List<int[]> dataList = new ArrayList<>();

    public void generateList(int size){
        /**
         * @description: 生成数据
         * @param {size}: 数据集大小 
         */        
        this.dataSize = size;
        for(int i=0;i<size;i++){
            this.dataList.add(randPoint());
        }
    }

    public int[] get(int ind){
        /**
         * @description: 访问数据集
         * @param {ind}: 索引
         * @return {dataList.get(ind)}: 返回的被访问数据
         */        
        return dataList.get(ind);
    }

    public void setClass(int ind, int label){
        /**
         * @description: 为特定的点设置其属于的聚类
         * @param {ind}: 索引
         * @param {label}: 欲设置的聚类/标签
         */        
        int[] point = get(ind);
        point[2] = label;
        dataList.set(ind, point);
    }

    public void setCent(int ind, boolean clearState){
        /**
         * @description: 设置点是否为质心
         * @param {ind}: 索引
         * @param {clearState}: 是否为质心，true时为质心；false时将质心变为普通点
         */        
        int[] point = get(ind);
        if(clearState) point[3] = 1;
        else point[3] = 0;
        dataList.set(ind, point);
    }

    private static int[] randPoint(){
        /**
         * @description: 随机生成数据点
         * @return {point}: 生成的单个数据点
         */        
        int[] point = new int[4];
        for(int i=0;i<2;i++){
            point[i] = (int)Math.round(Math.random()*600);
        }
        point[2] = 0;
        point[3] = 0;
        return point;
    }
}

class HyperParameters{
    /**
     * @description: 超参数类，记录了算法需要的各种超参数；
     * @param {K}: K值
     * @param {dataSize}: 数据集大小
     * @param {numIter}: 迭代次数
     * @param {clusterStat}: 是否聚类，可视化中使用
     */    
    int K;
    int dataSize;
    int numIter;
    boolean clusterStat;
    public HyperParameters(){
        this.K = 0;
        this.dataSize = 0;
        this.numIter = 0;
        this.clusterStat = false;
    }
}

public class KMeans{
    public static void main(String[] args) {
        /**
         * 主函数，主要负责GUI界面框架设定
         */        
        DataSet myDataSet = new DataSet();
        
        JFrame frame = new JFrame("KMeans Algorithm");
        frame.setSize(800, 600);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JPanel drawPanel = new JPanel();
        drawPanel.setPreferredSize(new Dimension(600,600));
        frame.add(drawPanel, BorderLayout.EAST);

        JPanel toolPanel = new JPanel();
        toolPanel.setPreferredSize(new Dimension(200,600));
        frame.add(toolPanel, BorderLayout.WEST);
        placeComponents(frame, toolPanel, myDataSet, drawPanel);

        frame.setVisible(true);
    }

    private static void placeComponents(JFrame frame, JPanel panel, DataSet dataSet, JPanel drawPanel){
        /**
         * 各个组件添加
         */        
        HyperParameters parameters = new HyperParameters();
        panel.setLayout(null);

        JLabel paraBoundLabel = new JLabel("---Hyper Parameters---",JLabel.CENTER);
        paraBoundLabel.setFont(new Font("Helvetica", Font.BOLD, 14));
        paraBoundLabel.setBounds(10,10,180,20);
        panel.add(paraBoundLabel);

        JLabel clusterLabel = new JLabel("K Value:");
        clusterLabel.setBounds(10,40,80,30);
        panel.add(clusterLabel);

        JTextField clusterText = new JTextField(20); // K值输入框
        clusterText.setBounds(90,40,80,30);
        clusterText.setDocument(new NumberTextField());
        panel.add(clusterText);

        JLabel irLabel = new JLabel("Iretation:");
        irLabel.setBounds(10,80,80,30);
        panel.add(irLabel);

        JTextField irText = new JTextField(20); // 迭代次数输入框
        irText.setBounds(90,80,80,30);
        irText.setDocument(new NumberTextField());
        panel.add(irText);

        JLabel generateLabel = new JLabel("---Auto Gene Set---",JLabel.CENTER);
        generateLabel.setFont(new Font("Helvetica", Font.BOLD, 14));
        generateLabel.setBounds(10,120,180,20);
        panel.add(generateLabel);

        JLabel sizeLabel = new JLabel("No.Points:");
        sizeLabel.setBounds(10,150,80,30);
        panel.add(sizeLabel);

        JTextField sizeText = new JTextField(20); // 数据集大小输入框
        sizeText.setBounds(90,150,80,30);
        sizeText.setDocument(new NumberTextField());
        panel.add(sizeText);

        JButton geneButton = new JButton("Generate");
        geneButton.setBounds(10,190,180,50);
        geneButton.addActionListener(new ActionListener(){ // 数据自动生成
            @Override
            public void actionPerformed(ActionEvent event){
                try{
                    parameters.dataSize = Integer.parseInt(sizeText.getText());
                    dataSet.generateList(parameters.dataSize); 
                    JOptionPane.showMessageDialog(frame, "Data Generated");
                } catch (NumberFormatException e){
                    JOptionPane.showMessageDialog(frame, "You Havn't Typed in Dataset Size", "Warning", JOptionPane.WARNING_MESSAGE);
                }
            }
        });
        panel.add(geneButton);

        JButton clrButton = new JButton("Clear Drawing Panel");
        clrButton.setBounds(10,400,180,50);
        clrButton.addActionListener(new ActionListener(){ // 清除画板（drawPanel）
            @Override
            public void actionPerformed(ActionEvent event) {
                drawPanel.removeAll();
                drawPanel.repaint();
            }
        });
        panel.add(clrButton);

        JButton toImage = new JButton("Save Result to Image");
        toImage.setBounds(10,350,180,50);
        toImage.addActionListener(new ActionListener(){ // 保存可视化为图片
            @Override
            public void actionPerformed(ActionEvent event) {
                BufferedImage bi = new BufferedImage(600, 600, BufferedImage.TYPE_INT_BGR);
                File clusterFile = new File("./Clustered.png");
                File unclusteredFile = new File("./Unclustered.png");
                try{
                    if(clusterFile.exists()||unclusteredFile.exists()){
                        clusterFile.delete(); unclusteredFile.delete();
                        clusterFile.createNewFile(); unclusteredFile.createNewFile();
                    }
                }catch(IOException e){
                    e.printStackTrace();
                }
                Graphics g = bi.getGraphics();
                Graphics2D g2d = (Graphics2D) g;
                g2d.setColor(Color.WHITE);
                g2d.fillRect(0, 0, 600, 600);
                g2d.setColor(Color.RED);
                for(int i=0;i<dataSet.dataSize;i++){
                    int x = dataSet.get(i)[0];
                    int y = dataSet.get(i)[1];
                    int lab = dataSet.get(i)[2];
                    switch (lab) {      // 颜色选择，基于jdk的预设颜色
                        case 0:
                            g2d.setColor(Color.BLACK);
                            break;
                        case 1:
                            g2d.setColor(Color.RED);
                            break;
                        case 2:
                            g2d.setColor(Color.BLUE);
                            break;
                        case 3:
                            g2d.setColor(Color.GREEN);
                            break;
                        case 4:
                            g2d.setColor(Color.CYAN);
                            break;
                        case 5:
                            g2d.setColor(Color.MAGENTA);
                            break;
                        case 6:
                            g2d.setColor(Color.ORANGE);
                            break;
                        case 7:
                            g2d.setColor(Color.PINK);
                            break;
                        case 8:
                            g2d.setColor(Color.GRAY);
                            break;
                        case 9:
                            g2d.setColor(Color.YELLOW);
                            break;
                        case 10:
                            g2d.setColor(Color.LIGHT_GRAY);
                            break;
                        }
                    g2d.drawOval(x, y, 3, 3);
                }
                try{
                    if(parameters.clusterStat)
                    ImageIO.write(bi, "png", clusterFile);
                    else
                    ImageIO.write(bi, "png", unclusteredFile);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
        panel.add(toImage);

        JButton visButton = new JButton("Visualize");
        visButton.setBounds(10,450,180,50);
        visButton.addActionListener(new ActionListener(){   // 可视化，基与clusterStat变量。true时将clustered.png加载到程序界面；反之加载unclustered.png
            @Override
            public void actionPerformed(ActionEvent event){
                JLabel imLabel = new JLabel();
                if(parameters.clusterStat){
                    imLabel.setIcon(new ImageIcon("./Clustered.png"));
                    System.out.println("Show this");
                }else{
                    imLabel.setIcon(new ImageIcon("./Unclustered.png"));
                    System.out.println("Show Unclustered");
                }
                drawPanel.add(imLabel);
                imLabel.setBounds(0, 0, 600, 600);
                // TODO: Message Box of K and Size
            }
        });
        panel.add(visButton);
        
        JButton clusterButton = new JButton("Cluster");
        clusterButton.setBounds(10,500,180,50);
        clusterButton.addActionListener(new ActionListener(){   // K-Means主算法
            @Override
            public void actionPerformed(ActionEvent event) {
                parameters.clusterStat = true;
                try{
                    parameters.K = Integer.parseInt(clusterText.getText());

                    if(parameters.K > dataSet.dataSize){
                        JOptionPane.showMessageDialog(frame, "K Should Be Smaller Than Size");
                    }
                    parameters.numIter = Integer.parseInt(irText.getText());

                    int[] centroids = new int[parameters.K];
                    List centList = new ArrayList();
                    for (int k=0;k<parameters.K;k++){   // 随机选择质心
                        while(true){
                            int randInd = (int)Math.round(Math.random()*parameters.dataSize);
                            if(centList.contains(randInd)) continue;
                            else{
                            centList.add(randInd);
                            centroids[k] = randInd;
                            break;
                            }
                        }
                        centroids[k] = (int)Math.round(Math.random()*parameters.dataSize);
                        dataSet.setClass(centroids[k], k+1);
                        dataSet.setCent(centroids[k], true);
                    }

                    for(int n=0;n<parameters.numIter;n++){  
                        for(int i=0;i<parameters.dataSize;i++){ // 计算每一个点和质心点距离，开始聚类
                            int[] point = dataSet.get(i);
                            if(point[3]==1) continue;
                            int[] dist = new int[parameters.K];
                            for(int k=0;k<parameters.K;k++){
                                int[] cent_point = dataSet.get(centroids[k]);
                                dist[k] = (int)(Math.pow(cent_point[0]-point[0], 2)+Math.pow(cent_point[1]-point[1], 2));
                            }
                            int min = 0;
                            for(int k=0;k<parameters.K;k++){
                                if(dist[min]>dist[k]) min = k;
                            }
                            dataSet.setClass(i, min+1);
                        }
                        
                        for(int k=1;k<=parameters.K;k++){   // 重新计算质心
                            int sum_x = 0;
                            int sum_y = 0;
                            int count = 0;
                            for(int i=0;i<parameters.dataSize;i++){
                                int[] point = dataSet.get(i);
                                if(point[2]==k){
                                    sum_x = sum_x + point[0];
                                    sum_y = sum_y + point[1];
                                    count = count + 1;
                                }
                            }
                            int near_x = (int)Math.round(sum_x/count);
                            int near_y = (int)Math.round(sum_y/count);
                            HashMap<Integer, Integer> distances = new HashMap<Integer, Integer>();
                            for(int i=0;i<dataSet.dataSize;i++){
                                int[] point = dataSet.get(i);
                                if(point[2]==k){
                                    distances.put(i, (int)(Math.pow(point[0]-near_x,2)+Math.pow(point[1]-near_y,2)));
                                }
                            }
                            int min = distances.keySet().iterator().next();
                            for(Integer i : distances.keySet()){
                                if(distances.get(i)<distances.get(min)){
                                    min = i;
                                }
                            }
                            dataSet.setCent(centroids[k-1],false);
                            centroids[k-1] = min;
                            dataSet.setCent(centroids[k-1], true);
                        }
                    }
                    System.out.println("Clustered!");
                } catch (NumberFormatException e) {
                    JOptionPane.showMessageDialog(frame, "Please Type in K and No.Iteration!");
                }
            }
        });
        panel.add(clusterButton);
    }
}
```

