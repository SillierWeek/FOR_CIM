#### VSCode下的Verilog环境

<https://zhuanlan.zhihu.com/p/593091162?utm_id=0>

#### 值得注意的Verilog约定
##### 嵌套
模块定义不允许嵌套
多行注释不允许嵌套
##### 数字标注
不指明位数的数字默认为计算机字长的位数，比如'hAF是一个64位的十六进制数（也即在二进制下占64位；本机下）
**负数的标注**：符号在位数之前，比如-6'b1000
**带符号数的表示**：如-6'sd3，用于带符号的算术运算
##### 下划线
只提高可读性，可在**数字**中的任意位置出现
##### 字符串
不能包含回车符；保存在reg型变量中，一个字符占8位，所以reg变量长度通常得是8的倍数
##### 竞争
逻辑值相同而强度不同的多个信号竞争，结果值的强度和逻辑值服从高强度信号的值
逻辑值不同但强度相同的多个信号竞争，结果值为x，即不确定
##### 向量（数组）
最高有效位总是它的左边一位
```verilog
reg[15:0] data;
data='hFDB0;//非法
data[15:0]='hFDB0;//合法
reg[63:0] array_3d[3:0][1:0][7:0];
arrsy_3d[2][4]=0;//非法，企图写某一维度
```

##### wire和reg
见后

#### 基础概念
##### 数据类型
寄存器变量reg、线网wire、向量（用[]和:索引）、整数integer、实数real、时间time、（多维）数组
如：
```verilog
reg[63:0] array_3d[3:0][1:0][7:0];//三维64位寄存器数组，64是每个元素的位长
```
时间的其他类型：<https://blog.csdn.net/wuzhikaidetb/article/details/125992226>
##### 特殊字符
类似于C语言
\\n（换行）
\\t（tab）
\%%（%）
\\\（反斜杠）
\\"（双引号）
\\ooo（1~3个8位二进制数）
##### 系统任务
**\$display显示格式的控制符**
![[Pasted image 20231101232620.png]]
**$monitor变量监视**
\$monitor(...)，只需调用一次即一直生效，\$monitor存在多句时后面的会覆盖前面的；
\$monitoron、\$monitoroff相当于开关
比如：
```verilog
module test_reg();
    reg[15:0] data;
    reg[7:0] byte;
    integer i;

    initial begin
        $monitor($time," i=%1d,byte=%h",i,byte);
        data[15:0]='hFDB0;//赋值须放在初始化内？
        for(i=0;i<2;i=i+1)
        begin
            byte=data[(i*8)+:8];
            $display("%h",byte);
            #100;
        end
    end
endmodule
```
输出结果：
```
b0
                   0 i=0,byte=b0
fd
                 100 i=1,byte=fd
                 200 i=2,byte=fd
```
又如：
```verilog
module test_reg();
    reg[15:0] data;
    reg[7:0] byte;
    integer i;

    initial begin
        $monitor($time," i=%1d,byte=%h,data=%h",i,byte,data);
        data[15:0]='hFDB0;//赋值须放在初始化内？
        i=0;
        $monitoron;
        byte=data[(i*8)+:8];
        $display("%h",byte);
        #100;
        $monitoroff;
        i=i+1;
        byte=data[(i*8)+:8];
        $display("%h",byte);
        #100;
    end
endmodule
```
输出结果：
```
b0

                   0 i=0,byte=b0,data=fdb0

fd
```
**暂停与终止**
\$stop、\$finish
##### 阻塞/非阻塞赋值
区别是是否影响后面语句的执行
https://blog.csdn.net/atlll1/article/details/104636214
#### 模块定义
![[Pasted image 20231102223224.png]]
实现a(b+c)的模块定义如下
```verilog
module AND_OR(a,b,c,y);//声明了具有a,b,c,y四个引脚的AND_OR模块
  input a,b,c;
  output y;
  assign y=a&(b|c);//assign 表示是组合逻辑函数
 endmodule
```
也可写作
```verilog
module AND_OR(input a,b, c,output y);
//ANSI C风格，声明了具有a,b,c,y四个引脚的AND_OR模块，a,b,c默认类型为wire
  assign y=a&(b|c);//assign 表示是组合逻辑函数
 endmodule
```
#### 端口
参见：wire和reg
##### 位宽不同的连接
允许端口的内、外两部分位宽不同（给予警告）
##### 不进行连接的端口
```verilog
module A(input a,output s,reg[3:0] d);
	//...
endmodule

module Top;
reg p;
reg[3:0]r;
	//..
	A tmp_a(p,,r);//合法
endmodule
```
##### 顺序端口连接
参见上例
##### 命名端口连接
```verilog
module A(input a,output s,reg[3:0] d);
	//...
endmodule

module Top;
reg p,q;
reg[3:0]r;
	//..
	A tmp_a_sub(.a(p), .s(q));//悬空的忽略即可
	A tmp_a_all(.a(p), .d(r), .s(q));
endmodule
```
#### 层次
![[Pasted image 20231102230212.png]]
![[Pasted image 20231102230228.png]]
#### 测试平台
一个简单的testbench写法如下
```verilog
`timescale 1ns/1ps//这个“`”是键盘左上角Esc下方的，英文模式输入
`include "AND_OR.v"
module test_AND_OR;
  reg[2:0] cnt;//用于发生三位二进制输入
  wire y_out;
//实例化门
  AND_OR a(cnt[2],cnt[1],cnt[0],y_out);
//initial begin后的代码只会执行一次
  initial begin
    //Verilog以缩进取代大括号
    //=====生成波形=====
    $dumpfile("wave.vcd"); //生成的 vcd 文件名称 
    $dumpvars(0, test_AND_OR); //第二个参数为 tb 模块名称
    //=================
    cnt=3'b000;
    repeat (8) begin
      #100
      $display("IN=%b,OUT=%b",cnt,y_out);//%b是二进制输出格式符
      cnt+=1;
    end
  end
endmodule

```
运行后输出
```
[Running] AND_OR_tb.v
IN=000,OUT=0
IN=001,OUT=0
IN=010,OUT=0
IN=011,OUT=0
IN=100,OUT=0
IN=101,OUT=1
IN=110,OUT=1
IN=111,OUT=1
[Done] exit with code=0 in 0.106 seconds
```

注意vcd要点击Add Signals里面，选择了变量才看得见波形

![](image/image_7l0U6_2xwj.png)

#### 描述组合逻辑
![](image/image_3juqeOLdPY.png)

#### 利用综合器实例化逻辑门
![](image/image_t70fcayaPA.png)

![](image/image_LOiVNRbDRv.png)

综合工具通过约束文件对设计进行再优化时倾向于提高电路的速度而非面积

编写代码时优先选用行为描述方式可使综合器尽可能利用无关项来简化逻辑

#### 有一定位长的变量声明
```verilog
input[3:0] in;
reg[0:3] tmp;//以tmp[0]为最高有效位
```

#### 一个典型的Verilog输入/输出模块
```verilog
module <模块名>(<参数名列表>);
  <输入/输出参数声明>;
  <内部信号（wire,reg）声明>;
  <模块体>;
endmodule
```

#### wire和reg
wire是输出信号默认声明类型，用于模块之间连接或assign语句赋值。比如以下语句等价
```verilog
wire a;
assign a=(b&c)|(~b&d);
//===========================
wire a=(b&c)|(~b&d);
```
线网的默认值为z，trireg型默认为x
reg用于case和casex语句中被赋值
与wire不同，reg类型的变量仅用于存储数据，无需驱动源和时钟信号（相比之下硬件寄存器则需时钟信号），其值可在仿真过程中的任意时刻通过赋值改变
```verilog
reg signed d[3:0];//寄存器型变量也可声明为带符号类型以参与带符号的算术运算
```
所有端口隐式定义为wire类型；若输出类型的端口需存储数值（即值不是随便变的），则须定义为reg；输出可以定义为reg型，但不能连接到reg型
input、inout（作双向数据传输的输入/输出端口）类型的端口，不能被声明为reg
#### case和casex
case——必须是强的枚举，即最小项
casex——允许枚举中出现蕴含项（无关状态），比如
```verilog
//..
  input[2:0] in;
  output mod4;
  always @(in) begin
    casex(in)
      4'bx00:mod4=0;
      default:mod4=1;
    endcase//不是endcasex
  end
//..
```
x为不定态，z为高阻抗态（也可以用?表示，除UDP外），在**二进制的某一位**上均合法
#### 描述组合逻辑的方法

**行为描述**
①直接的行为描述，如10以内素数电路输入2,3,5,7输出真，其余输出假；
②最简与或式行为描述：基于casex语句；
③真值表式行为描述：基于assign语句

**结构描述**
实例化逻辑门，不同的是可以避免实例化XOR12这样具体的门，而使用Verilog封装的门函数
```verilog
//比如wire a=(b&c)|(~b&d);用结构描述的方法为
//f.v——模块名与文件名一致
module f(a,b,c,d);
  input b,c,d;
  output a;
  wire b_and_c,nb_and_d;
  
  and and1(b_and_c,b,c);
  and and2(nb_and_d,~b,d);
  or or1(a,b_and_c,nb_and_d);
endmodule
```

#### 编写测试平台（testbench）
测试平台本质上仍是.v文件，但不被综合，不占用芯片面积。注意，测试平台中的initial、repeat、#10（延时）在综合时都是不允许或不建议的
```verilog
//f_tb.v
`include "f.v"//须包含源模块文件——默认在同一目录下
module f_tb;
  reg[2:0] in;
  wire a;
  
  f test_f(a,in[2],in[1],in[0]);//参数顺序须与模块定义的相同
  
  initial begin
    in=0;//给输入赋初值
    repeat(8) begin
      #10
      $display("in = %3b ; OUTPUT = %1b",in,a);
      in=in+1;
    end
  end
endmodule
```
可以将两个同一功能不同实现模块用testbench对照以验证功能的正确性
![](image/image_qH4nYk0xjb.png)

#### 定义常量

```verilog
//比如定义一个常量SI=4330
`define SI 4'd4330
//使用时
//..
  case(x)
    `SI:out=1;//注意短反撇
//..

```

#### 同时赋值
![](image/image_bUGFsKUSe3.png)
#### 默认参数

```verilog
//独热码发生器
module Dec(a,b);
//定义默认参数n=2,m=4
  parameter n=2;
  parameter m=4;
  
  input[n-1:0] a;
  output[m-1:0] b;
  
  assign b=1<<a;
endmodule

//实例化时
Dec dec_2_4(a1,b1);
Dec #(3,8) dec_3_7(a2,b2);//使用非默认的参数定义3位输入8位输出的独热码发生器
```

#### 局部参数
![[Pasted image 20231101222439.png]]

#### 信号复制扩长
k{y}是将y复制k次成一个四位的二进制数

#### 缩减运算符

![](image/61a0fb9241ac4a3ea19947fc132d8578_7SpAmvD1VN.png)
比如用缩减运算符比较两个二进制数是否相等：
```verilog
wire eq=&(a~^b)//~^是同或，也就是异或^的反
```

#### 输出时序时间
```verilog
$display($time,"res=%d",result);
```
则开始会输出从复位时计所经过的时间及result的值

#### 可变向量域选择
```verilog
for(i=0;i<8;i=i+1)
	tmpByte=data[(i*8)+:8];//即8位8位地选，顺序是[7:0],[15:8],...[63,56]
```
比如
```verilog
module test_reg();
    reg[15:0] data;
    reg[7:0] byte;
    integer i;
    initial begin
        data[15:0]='hFDB0;//赋值须放在初始化内？
        for(i=0;i<2;i=i+1)
        begin
            byte=data[(i*8)+:8];
            $display("%h",byte);
        end
    end
endmodule
```
输出结果
```
b0
fd
```

#### 存储器建模
![[Pasted image 20231101222226.png]]


#### 门级建模
就是连接逻辑图（门级电路图）的连线过程
具体步骤：画逻辑图→门级原语转述→编写激励→测试观察
基本的门都会把高阻态变为不定态
引用格式中，参数统一为输出在前
##### 与门和或门
这一组允许多于2个的输入
```verilog
wire[7:0]out;
wire in1,in2;
wire in3;

and a1(out[7],in1,in2);
nand na1(out[6],in1,in2);
or or1(out[5],in1,in2);
nor nor1(out[4],in1,in2);
xor x1(out[3],in1,in2);
xnor xn1(out[2],in1,in2);
and a_3(out[1],in1,in2,in3);
and (out[0],in1,in2);//不给实例命名的引用

wire[7:0] outArray,in4,in5;

and andArray[7:0](outArray,in4,in5);//相当于8个与门一一对应，构成阵列
```
##### 缓冲器和非门
这一组允许多于1个的输出
```verilog
wire[3:0]out;
wire in1;
buf b1(out[3],in1);
not n1(out[2],in1);
buf b_2(out[1],out[0],in1);
```
##### 带控制端的门
```verilog
wire[3:0]out;
wire in1,ctrl;
bufif1 bf1(out[3],in1,ctrl);//ctrl为1时用作缓冲器，否则高阻态/不定态/质量不好的高低电平
bufif0 bf2(out[2],in1,ctrl);
notif1 nt1(out[1],in1,ctrl);
notif0 nt2(out[0],in1,ctrl);
```
##### 门延迟
**上升延迟、下降延迟**都是由0变到1（不是10%~90%）
**关断延迟**：变到高阻态
![[Pasted image 20231103200148.png]]
![[Pasted image 20231103200258.png]]

