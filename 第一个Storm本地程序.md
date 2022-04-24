# 第一个Storm本地程序

## 一、程序简介

```
1、通过模拟用户访问网页产生的浏览日志信息，访问信息，截取用户的会话id并输出。
2、此过程中，首先创建一个数据保存为日志信息文件，通过storm程序的处理完成目标
```

## 二、类的简介

```
1、创建一个类 GenerateData实现日记数据的创建
2、创建一个拓扑项目（包括组件Spout、Bolt）
	Spout类：实现接收数据并将数据源发送出去，为拓扑的数据发送来源
	Bolt类：为数据处理类，接受Spout传输的数据，实现对数据的细节操作处理，同时可以定义多个Bolt，实现对数据的多层处理。
3、创建Main运行主程序：依据Topology类创建一个拓扑包含并设置Spout和Bolt，设置拓扑细节，实现在本地运行和集群运行操作的设定。
```

## 三、实现步骤（此程序基于idea开发）

### 3.1 导入Storm包到idea中

```
1、实现上述代码需要借助storm提供的工具包和类，依据外部包完成拓扑的创建
2、将storm安装文件下lib文件夹导入idea中作为外部可用包。
	File-->project structure(ctrl+shitf+alt+s) -->libraries-->点击+ 即可添加lib包
```

### 3.2创建数据（GenerateData类）

通过String数组的拼接和字节输出流将数据保存在Website.log中。

```java
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Random;

public class GenerateData {
	public static void main(String[] args) {
		
		//1 创建一个文件路径
		File file = new File("F:Storm_Work/Website.log");
		//2 准备数据
//		2.1网站名称
		String[] hosts = { "www.uu_fly_pig.com" };
		
		//2.2 会话id
		String[] session_id = {"ABYH6Y4V4SCVXTG6DPB4VH9U123", "XXYH6YCGFJYERTT834R52FDXV9U34",
				"BBYH61456FGHHJ7JL89RG5VW9UYU7", "CYYH6Y2345GHI8990FG4V9U567", "WWYH6Y4V4SFXZ56JIPDPB4V678" };
		
		// 2.3访问网站时间
		String[] time = {"2017-08-07 08:40:50", "2017-08-07 08:40:51", "2017-08-07 08:40:52", "2017-08-07 08:40:53","2017-08-07 08:40:53",
						"2017-08-07 09:40:49", "2017-08-07 10:40:49", "2017-08-07 11:40:49", "2017-08-07 12:40:49" };

		//3 拼接数据
		Random random = new Random();
		StringBuffer sb = new StringBuffer();
		
		for (int i = 0; i< 30; i++) {
			sb.append(hosts[0] + "\t" + session_id[random.nextInt(5)] + "\t"  + time[random.nextInt(8)] + "\n");
			
		}
		
	
		//4写数据到文件
		FileOutputStream outputstream = null;
		
		try {
			outputstream = new FileOutputStream(file);
		
			outputstream.write(sb.toString().getBytes());
		}catch (Exception e) {
			e.printStackTrace();
			
		}finally {
			//5 关闭资源
			try {
				outputstream.close();
			}catch (IOException e) {
				e.printStackTrace();
			}	
		}
	}
}

```

### 3.3 创建Spout组件（WebLogSpout类）

Spout组件的创建可以继承类BaseRichSpout，实现的方法较少，本程序以实现接口IRichSpout的方式同样可以。

方法nextTuple主要用于发送数据给Bolt组件

```java
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Map;

import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichSpout;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Values;

public class WebLogSpout implements IRichSpout{
	private BufferedReader reader; 
	private SpoutOutputCollector collector;
	private String str;
	
	@Override
	public void nextTuple() {
		//发送数据
		try {
			while((str = reader.readLine()) != null) {
				collector.emit(new Values(str));
				Thread.sleep(500);
			}
		} catch (Exception e) {

		}
		
	}

	@Override
	public void open(Map<String, Object> arg0, TopologyContext arg1, SpoutOutputCollector collector) {
		
		this.collector = collector;
		
		
		//打开文件
		try {
			reader = new BufferedReader(new InputStreamReader(new FileInputStream("F:/Storm_Work/Data/Website.log")));

		} catch (IOException e) {

			e.printStackTrace();
		};
	}

	@Override
	public void declareOutputFields(OutputFieldsDeclarer declear) {
		declear.declare(new Fields("logs"));

	}

	@Override
	public Map<String, Object> getComponentConfiguration() {
		// TODO 自动生成的方法存根
		return null;
	}
	@Override
	public void ack(Object arg0) {
		// TODO 自动生成的方法存根
		
	}

	@Override
	public void activate() {
		// TODO 自动生成的方法存根
		
	}

	@Override
	public void close() {
		// TODO 自动生成的方法存根
		
	}

	@Override
	public void deactivate() {
		// TODO 自动生成的方法存根
		
	}

	@Override
	public void fail(Object arg0) {
		// TODO 自动生成的方法存根
		
	}

	
	
}

```

### 3.4创建Bolt组件

主要通过execute方法，执行器进行对数据的细节操作。

```java
import java.util.Map;

import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichBolt;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Tuple;

public class WebLogBolt implements IRichBolt{
	private int line_num;
	@Override
	public void cleanup() {
		// 清除数据要关闭时处理的
		
	}

	@Override
	public void execute(Tuple input) {
		// 执行
		
		
		//1 获取数据
//		String stringByField = input.getStringByField("logs");
		String line = input.getString(0);
		
		// 2 切割数据
		String[] split = line.split("\t");
		String session_id = split[1];
		
		//3 统计行数
		line_num++;
		
	
		//4 打印
		System.out.println(Thread.currentThread().getId() + " session_id: " + session_id + " line_num:" + line_num);
	}

	@Override
	public void prepare(Map<String, Object> arg0, TopologyContext arg1, OutputCollector collector) {
		// 准备
		
	}

	@Override
	public void declareOutputFields(OutputFieldsDeclarer arg0) {
		// 声明
		
	}

	@Override
	public Map<String, Object> getComponentConfiguration() {
		// 获取配置信息
		return null;
	}

}
```

### 3.5 主程序创建拓扑（WebLogMain）

主程序中设定了两个Spout和两个Bolt，两个数据发送源和两个线程处理数据。

```java
import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.StormSubmitter;

import org.apache.storm.generated.AlreadyAliveException;
import org.apache.storm.generated.AuthorizationException;
import org.apache.storm.generated.InvalidTopologyException;
import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.thrift.TException;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.topology.base.BaseRichSpout;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;
import org.apache.storm.utils.Utils;


public class WebLogMain{
	public static void main(String[] args) throws Exception {
		//1 创建拓扑
		//定义输入源
		
		TopologyBuilder builder = new TopologyBuilder();
		
		//2 设置Spout和Bolt
		
		builder.setSpout("WebLogSpout", new WebLogSpout(), 2);
		builder.setBolt("WebLogBolt", new WebLogBolt(), 2). shuffleGrouping("WebLogSpout");
		
		//3 配置Worker开启个数
		
		Config conf = new Config();
		conf.setNumWorkers(2);
		
		//4 提交程序
		
		if(args.length > 0) {
			//集群提交
			try  {
				StormSubmitter. submitTopology(args[0], conf, builder.createTopology());
			}
			catch (Exception e) {
				e.printStackTrace();
			}
		}else{
			//本地提交 

				LocalCluster localCluster = new LocalCluster();
				localCluster.submitTopology("webtopology", conf, builder.createTopology());
		}
	}
}

```

## 四、输出结果

![image-20220414225259585](C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20220414225259585.png)
