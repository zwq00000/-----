无线测量数据终端（HMI）设计方案
===============================
##一、概述 ##
数据终端是无线测量整体解决方案的一部分，解决作业现场的测量数据的显示和报警。
数据终端完成一些简单的功能，和数据接收管理程序不同，终端不负责编制测量方案和测量任务。
一个数据终端(HMI)只负责一个工位的测量任务，一个测量工位假设只有一位测量员，同时只能进行一个测量任务，允许多个无线量具发射端。

###1.2 测量任务###
 a. 测量方案
 测量方案是预定义的对单一测量作业内容的描述，一个测量方案通常只对应一种测量目标对象（被测量的零件）。
 一个测量方案中包括多个测量规格，每个测量规格包括 顺序号、测量点名称、 测量范围（上下公差）和 量具信息。
 *量具信息允许在制定测量任务时指定，对于数据终端来说，是必须配置的数据项*
 
 b. 测量任务
 测量任务由 **人、机、料、法、环** 五部分构成。
 
  - **人** 测量人员（工号、部门、班组、姓名等）
  - **机** 测量工具（量具）
  - **料** 被测工件信息(零件号、品名、图号、批号）
  - **法** 根据生产工艺要求制定的测量标准（测量位置、采样数量、公差范围、量具类型）
  - **环** 应用环境（时间、工序、工位等）
  
 其中 **测量方案** 规范了 **法**的部分。
 测量任务是测量方案和测量数据的集合。
 一个测量任务中仅包含一个测量方案。

##二、终端功能描述##

###2.1 下载/接收测量任务###
终端负责下载测量任务，并更新显示功能。
接收任务信息允许手动接收和自动接收两种方式。（第一期先开发手动接收）
接收方式采取从U盘读取和直接接收网络数据。（第一期采取从U盘读取指定的配置文件）

###2.2 无线数据接收###
通过串口接收无线测量的数据帧，实现帧校验。
测量数据帧通过**量具ID**匹配，决定是否接收。需要按照接收顺序记录序号、接收时间、测量值和发射端电量。
接收的数据根据按键特征进行处理。
通常单次按键表示接收到一个正常的测量数据。
长按键表示取消上一条的测量记录。

###2.3 测量数据显示###
根据测量任务规格匹配当前测量位置，测量位置信息由**测量规格**内容决定。
根据测量顺序逐个显示测量值。

- 根据测量规格明细的分组信息
    - 测量值根据测量规格明细进行分组。

需要显示的项目:
 - 计数值(接收组数)
 - 当前位置
 - 测量点名称 测量值
 
根据当前测量位置显示当前测量值。

*第二期会增加简单的统计分析功能*

###2.4 报警功能###
在数据显示界面上有明显的报警信息。
报警信息需要明确,鲜明。
蜂鸣器报警安排在第二版实现。

###2.5 测量数据存储###
接收到的测量数据根据测量任务Id存储到规定目录下，按照接收顺序依次排列。
存储格式为 **测量时间\t测量读数**
*注:测量读数是指接收的原始值*

###2.6 历史数据查询/测量数据表格###
第一期需要完成当前测量任务的数据表格。
由于测量规格项目数量不确定，所以表格列数量也是动态的。

##三、数据结构##

###3.1 BeeFrame 接收数据帧###
- 帧结构
    -  帧头( len = 1byte; val = 0xFE )
    -  +长度( len = 1byte; val &lt; 98 )
    -  +有效数据( len &lt; 98bytes; val = any )
        -  ++ 桢类型
        -  ++ 载荷数据
    -  +校验( len = 1byes; val = 所有有效数据异或结果 )

- 测量数据接收数据帧
    - 功 能: 远端数据接收
    - 命 令 值: 0x25
    - 数据格式:
        - 帧头( len = 1byte; val = 0xFE )
            -  长度( len = 1bytes; 6 <= val <= 98 )
            - cmd/帧类型 ( len = 1bytes; val = 0x25|0x80 )
            - 数据( len <= 97bytes;
                - val = 数据编号( len = 1byte; val = any 本字段用于分别短时间内远端节点重复发送的同一个数据 )
                - 其它 ( len = 4byte; val = any )
                - 数据 "ID:%04X %f \r\n")
        -  校验( len = 1bytes; val = 所有有效数据异或结果 )

- 按键类型
    - 按键类型包含在 接收数据帧载荷数据的第一个字节
    - 长按 1
    - 短按 0
    
###3.2 测量任务 ###

 1. 测量任务定义
 测量任务按照 **人、机、料、法、环**五部分组成
 任务Id 格式为 **yyyyMMdd{4位序号}**
 
 2. 测量任务数据交换格式（XML）
 
```XML
<!-- 测量任务 -->
<TaskStore TaskId="{任务Id}" EmployeeId="{工号}" EmployeeName="操作工姓名" DepName="{部门名称}" 
            PartId="{工件Id}" PartNo="{工件号}" PartName="{工件名称}" SchemaId="{测量方案Id}" 
            WorkDate="{作业日期}" ProcessId="{工序}" Station="{测量工位}" Batch="{批次号}"  CreatedBy="{创建人}" Status="{任务状态}">
    <!-- 测量方案 -->
    <MeasuringSchema Name="{测量方案名称}" SchemaId="{测量方案Id}" Summary="{方案摘要/说明}" FeatureCount="{特征点数量}" 
                    SamplesCount="{测量点数量}" Units="{测量单位:英寸/厘米...}" CreateDate="{方案创建日期}">
        <!-- 测量规格 -->            
        <Tolerance FeatureName="{测量点名称}" OrderNumber="{序号}" RawLSL="{规格下限}" 
                    RawUSL="{规格上限}" GageId="{量具Id}" SamplesCount="{采样数:同一位置测量次数}" />
        <Tolerance FeatureName="{测量点名称}" OrderNumber="{序号}" RawLSL="{规格下限}" 
                    RawUSL="{规格上限}" GageId="{量具Id}" SamplesCount="{采样数:同一位置测量次数}" />
        <Tolerance FeatureName="{测量点名称}" OrderNumber="{序号}" RawLSL="{规格下限}" 
                    RawUSL="{规格上限}" GageId="{量具Id}" SamplesCount="{采样数:同一位置测量次数}" />
                    
    </MeasuringSchema>
</TaskStore>
```

样例数据
```xml
<TaskStore TaskId="201403080007" EmployeeId="21" EmployeeName="似露非露" DepName="asd" PartId="2" PartNo="12129" PartName="12912010" DrawingNo="19souf203" SchemaId="102CE1DA-A58B-46F1-97FB-3526DC2F4148" WorkDate="2014-03-08T18:45:47.9489964+08:00" ProcessId="机加工" CreatedBy="系统管理员" Status="Running">
    <MeasuringSchema Name="新建测量方案" SchemaId="102CE1DA-A58B-46F1-97FB-3526DC2F4148" Summary="" FeatureCount="3" SamplesCount="3" Units="Inch" GageCount="1" CreateDate="2014-02-16T21:22:52.2842224+08:00" PartId="2"  ProcessId="机加工">
        <Tolerance FeatureName="1#" OrderNumber="1" RawLSL="0" RawUSL="10" GageId="ACDA" SamplesCount="1" />
        <Tolerance FeatureName="2#" OrderNumber="2" RawLSL="0" RawUSL="10" GageId="ACDA" SamplesCount="1" />
        <Tolerance FeatureName="3#" OrderNumber="3" RawLSL="0" RawUSL="10" GageId="ACDA" SamplesCount="1" />
    </MeasuringSchema>
</TaskStore>
```

###3.3 数据交换文件规格 ###
 1. 测量任务文件
  - 任务文件通过U盘或者通过网络下发
  - 可以筛选当天的任务
  - 可以筛选本机任务文件（工序+工位）
  - 可以不指定工序和工位进行选择
  - 文件名格式为 **{任务Id}.task**.
  
 2. 任务原始记录
     - 任务原始记录存储格式采用文本格式，每条记录一行。文件名为 **{任务Id}.data**。
     - 文件格式采用文本格式，格式为**{操作时间}\t{原始读数}\n**
     - 任务数据格式样例 
     >"09:21:12\t12.23\n"

 3. 统计数据格式（**HMI暂不实现**）
  - 统计数据使用XML格式



###3.3 测量方案 ###
- 测量方案 **MeasuringSchema**
 - 方案ID **SchemaId**
 - 方案名称 **Name**
 - 方案说明 **Summary**
 - 测量单位 **Units** 厘米/英寸
 - 测量值总数 **FeatureCount**
 - 采样总量 **SamplesCount**
 - 创建人 **CreatedBy**
 - 创建日期 **CreateDate**
 - 零件信息 **零件号、零件名称、图纸编号**
 - 测量规格 **Tolerances** 测量规格集合

- 测量规格 **Tolerance**
 - 测量顺序 **OrderNumber**
 - 测量值名称 **FeatureName**
 - 规格下限 **LSL**
 - 规格上限 **USL**
 - 规格下限(文本) **RawLSL** (为了保留原始分辨率) 
 - 规格上限(文本) **RawUSL** ( 为了保留原始分辨率) 
 - 测量点数量 **SamplesCount**
 - 中心值 **U** 计算值
 - 公差范围 **R** 计算值
 
- 注册量具 **Gage**
 - 量具ID **GageId** 无线发送器编号
 - 量具编号 **GageSN** 用户自定义编号
 - 量具名称 **Name**
 - 量具类型 **GageType**
 - 供应商 **Vendor**
 - 测量精度 **Accuracy**
 - 量程下限 **LowerRang**
 - 量程上限 **UpperRange**
 - 使用者 **UserOf**
 - 量具电量 **BetteyValue**
 
- 测量记录 **IMeasureData**
 - 记录时间 **Time** (记录当时的时间)
 - 测量值 **Value** (数值类型)
 - 读数(原始值) **RawValue**
 
## 四、C#参考实现##

 1. 帧类型
    ```C#
    /// <summary>
    ///     桢命令类型
    /// </summary>
    [Flags]
    public enum FrameType : byte{
        /// <summary>
        /// 接收数据桢
        /// </summary>
        [Description("接收桢")]
        RxFrame = 0x80,
        
        /// <summary>
        ///     回测命令: 0x3F
        /// </summary>
        [Description("回测")]
        Echo = 0x3F,
    
        /// <summary>
        ///     取接收器信息命令:0x31
        /// </summary>
        [Description("读接收器信息")]
        ReadReciverInfo = 0x31,
    
        /// <summary>
        ///     恢复出厂设置:0x38
        /// </summary>
        [Description("恢复出厂设置")]
        FactoryReset = 0x38,
    
        /// <summary>
        ///     配置中心通信密钥:0x21
        /// </summary>
        [Description("配置中心通信密钥")]
        SetReciverKey = 0x21,
    
        /// <summary>
        ///     读取中心通信密钥: 0x22
        /// </summary>
        [Description("读取中心通信密钥")]
        ReadReciverKey = 0x22,
    
        /// <summary>
        ///     配置其它设备通信密钥:0x23
        /// </summary>
        [Description("配置其它设备通信密钥")]
        SetDeviceKey = 0x23,
    
        /// <summary>
        ///     读取其密设备通信密钥:0x24
        /// </summary>
        [Description("读取其密设备通信密钥")]
        ReadDeviceKey = 0x24,
    
        /// <summary>
        ///     接收 远端数据 0x25
        /// </summary>
        [Description("接收数据")]
        ReciveData = 0x25,
    }
    ```
 2. 帧结构
    ```C#
    /// <summary>
    ///     全局命令帧格式：
    ///     帧头( len = 1byte; val = 0xFE )
    ///     +长度( len = 1byte; val &lt; 98 )
    ///     +有效数据( len &lt; 98bytes; val = any )
    ///     ++ 桢类型
    ///     ++ 载荷数据
    ///     +校验( len = 1byes; val = 所有有效数据异或结果 )
    /// </summary>
    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Ansi)]
    public struct BeeFrame{
        /// <summary>
        ///     桢头标志
        /// </summary>
        public const byte FrameHead = 0xFE;
    
        /// <summary>
        ///     接收桢 掩码
        /// </summary>
        public const byte RxFrameMask = 0x80;
    
        /// <summary>
        ///     有效载荷数据长度( len = 1byte; val &lt; 98 )
        /// </summary>
        public byte DataLen;
    
        /// <summary>
        ///     帧类型
        /// </summary>
        public FrameType FrameType;
    
        /// <summary>
        ///     有效数据( len &lt; 98bytes; val = any )
        /// </summary>
        public byte[] Data;
    
        /// <summary>
        ///     校验( len = 1byes; val = 所有有效数据异或结果 )
        /// </summary>
        //public byte Checksum;
        public BeeFrame(FrameType frameType){
            DataLen = 1;
            FrameType = frameType;
            Data = new byte[0];
        }
    
        public BeeFrame(FrameType frameType, byte[] data){
            DataLen = (byte) (data.Length + 0x1);
            FrameType = frameType;
            Data = data;
        }
    
        /// <summary>
        ///     桢命令类型
        /// </summary>
        public FrameType BaseFrameType{
            get{
                return (FrameType & ~FrameType.RxFrame);
            }
        }
    
        /// <summary>
        ///     是否为接收桢
        /// </summary>
        public bool IsRxFrame{
            get { return (FrameType & FrameType.RxFrame) == FrameType.RxFrame; }
        }
    
        /// <summary>
        ///     <see cref="Data" />字段的按位异或校验
        /// </summary>
        public byte CheckSum{
            get { return (byte) (Data.CalculateChecksum() ^ ((byte) FrameType)); }
        }
    
        /// <summary>
        /// 写入到IO流
        /// </summary>
        /// <param name="outputStream"></param>
        public void WriteTo(Stream outputStream){
            outputStream.WriteByte(FrameHead);
            outputStream.WriteByte(DataLen);
            outputStream.WriteByte((byte) FrameType);
            outputStream.Write(Data, 0, Data.Length);
            outputStream.WriteByte(CheckSum);
        }
    }
    ```
 3. 数据接收方法
 
 ```C#
    /// <summary>
    /// 从串口读取数据
    /// </summary>
    /// <param name="port"></param>
    /// <returns></returns>
    public static BeeFrame ReadStream(SerialPort port){
        while (true){
            if (port.ReadByte() == BeeFrame.FrameHead){
                //读取到桢头
                var frame = new BeeFrame{DataLen = (byte) port.ReadByte(), FrameType = (FrameType) port.ReadByte()};
                if (frame.DataLen > 1){
                    //读取数据包
                    frame.Data = new byte[frame.DataLen - 1];
                    port.Read(frame.Data, 0, frame.Data.Length);
                }
                int checksum = port.ReadByte();
                if (frame.CheckSum != checksum){
                    throw new IOException("数据包校验错误");
                }
                return frame;
            }
        }
    }
 ```
 4. 测量数据解析

 ```C#
 /// <summary>
/// 预编译的消息解析正则表达式
/// </summary>
private static readonly Regex ParseReg = new Regex(@"(id?|Id?|iD?|ID?):(?<SN>\S+)\s+(?<Value>(-?\d*\.?\d*))",
                                                          RegexOptions.ExplicitCapture | RegexOptions.Compiled);
    /// <summary>
    /// 解析测量数据消息
    /// </summary>
    /// <param name="reciveStr">形如"ID:%04X %f \r\n"</param>
    /// <returns>测量记录</returns>
    public static MeasureRecord ParseMessage(string reciveStr)
    {
        var match = ParseReg.Match(reciveStr);
        if (match.Success){
            var sn = match.Result("${SN}");
            string rawValue = match.Result("${Value}");
            return new MeasureRecord(){GageId = sn,RawValue = rawValue};
        }
        return null;
    }

 ```



> Written with [StackEdit](https://stackedit.io/).