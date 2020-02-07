# CAMx来源解析模块(SA)


光化学模式通常用来制定减排策略来达到空气质量的目标。传统的方法涉及到运行大量的减排策略或**零排放(Zero-Out)**模拟，从而确定特定污染物，污染类别和污染区域的来源。导致此过程有些不切实际，但是如果缺乏这些信息，可能导致来源的贡献被低估或高估，无法准确的识别贡献源。


CAMx中提供了源分配SA或源贡献功能，在模式单次运行过程中，可以评估多个源区域，类别和污染物类型对臭氧和颗粒物的时空贡献。应用此方法的难点在于：追踪前向排放和后续非线性变化的目标污染物之间的关系，包括：

* 不仅要考虑在**特定溯源点**来自**指定源区域**的**前体物**的情况，而且还要准确的评估前体物经过溯源点(receptor，即污染物的受体)时，对目标污染物的累积贡献
* 确保空气质量模型计算的**一致性**，因此得到的**源-受体**(source-receptor)之间的关系要和模式结果的总体浓度要一致
* 在实际情况限制下，要有足够的时空分辨率，而且计算资源要能够保证源分配工具的运行



SA利用一系列**示踪剂**追踪**前向排放的变化**以及由这些排放所形成的**臭氧和颗粒物组分**。这些示踪剂在CAMx的计算过程中仅作为"旁观者"，不参与到CAMx的计算过程，因此对总排放和浓度间的关系不会产生干扰。SA示踪剂并不是被动参与到CAMx计算过程中，而是会追踪在CAMx模拟过程中化学反应(chemical reaction)，传输(transport)，扩散(diffusion)，排放(emissions)和沉降(deposition)的影响，因此可以视为**"反应示踪剂"(reaction tracer)**。根据**地理区域**(geographical area(or region))或**排放类别**(emission category(or group))定义**源**(source)。

下图展示了CAMx模拟区域可以划分为多个源区域(source areas)，示例图中显示划分为40个源区域。而且，排放清单也可以划分为多个源类别，比如，每个源区域中，排放清单划分为三个类别(移动源，工业源和生物源)，这将产生120个示踪集合。

![](https://raw.githubusercontent.com/bugsuse/blogpic/master/2019/05/15/ch7_fig1.png)

<center><font size=2 color='gray'>图1 CAMx区域划分来源区域</font></center>

因为要考虑前体物(precursors)，臭氧(ozone)和颗粒物(PM)的所有来源，因此CAMx的**初始和边界条件**也应该作为一个单独的**源组(source groups)**。此方法可以计算贯穿所有网格和时刻，在所选源区域/组中，前体物，臭氧和颗粒物的来源贡献，也可以评估在VOC和NOx限制条件下，臭氧的形成，从而验证在特定时刻和位置，VOC或NOx排放的减少，臭氧是否会产生响应。



反应示踪剂的一个重要特征是：并不会干扰到CAMx的计算过程。因此SA所评估的总的臭氧，颗粒物和前体物浓度与CAMx的模式结果是相同的。进一步来说，因为使用了相同的气象和排放等输入数据，以及相同的数值计算方法，因此SA得到的源和受体的关系与CAMx所产生的结果具有高度一致性。

这种源分配方法最大的局限性在于：不同源排放之间的非线性化学反应，这意味着任何对排饭清单的扰动都会以非线性的方式改变源-受体间的关系和贡献。因此，对于臭氧和颗粒物来说，SA的结果仅能应用于特定的排放情景中，无法用来推测在源区域/组排放的变化所造成的影响(就是说，无法根据特定的排放得到的结果来推测其它排放策略可能造成的影响)。


## PSAT

PSAT使用了多个示踪剂族来追踪主要和次要的PM(颗粒物)的变化。PSAT主要可以对下列类别的CAMx物种进行分配：

* Sulfate (PSO4)
* Particulate nitrate (PNO3)
* Ammonium (PNH4)
* Secondary organic aerosol (SOA)
* Particulate mercury (HgP)
* Six categories of primary PM:
  * Elemental carbon (PEC)  
  * Primary organic aerosol (POA)
  * Crustal fine (FCRS)
  * Other fine (FPRM)
  * Crustal coarse (CCRS)
  * Other coarse (CPRM)



一个示踪剂族可以分配主要的颗粒物类别，然而次要的颗粒物类别需要几种示踪剂族追踪气体前体物和PM之间的关系。 PNO3和SOA是要分配的最复杂的PM类别，因为释放的前体物气体(NO，VOC)通过几步从PM种类(PNO3，SOA)中移除。



源区域/组i对应的每种类型的PM的PSAT反应示踪剂如下，特定类别对应的PSAT示踪剂名称以字母'P'开头：

```
Sulfur

		SO2i Primary SO2 emissions
		PS4i Particulate sulfate from primary emissions plus secondarily formed sulfate  

Nitrogen
		NITi Nitric oxide (NO) and nitrous acid (HONO)
		RGNi Nitrogen dioxide (NO2), nitrate radical (NO3), and dinitrogen pentoxide (N2O5)
		TPNi Peroxyl acetyl nitrate (PAN), analogues of PAN and peroxy nitric acid (PNA)
		NTRi Organic nitrates (RNO3)
		HN3i Nitric acid (HNO3)
		PN3i Particulate nitrate from primary emissions plus secondarily formed nitrate
		NH3i Ammonia (NH3)
		PN4i Particulate ammonium (NH4)
		
Secondary Organics
		AROi Aromatic (benzene, toluene and xylene) secondary organic aerosol precursors
		ISPi Isoprene secondary organic aerosol precursors
		TRPi Terpene secondary organic aerosol precursors
		SQT Sesquiterpene secondary organic aerosol precursors
		CG1i Condensable gases from aromatics (low volatility products)
		CG2i Condensable gases from aromatics (high volatility products)
		CG3i Condensable gases from isoprene (low volatility products)
		CG4i Condensable gases from isoprene (high volatility products)
		CG5i Condensable gases from terpenes (low volatility products)
		CG6i Condensable gases from terpenes (high volatility products)
		CG7i Condensable gases from sesqiterpenes
		PO1i Particulate organic aerosol associated with CG1
		PO2i Particulate organic aerosol associated with CG2		
		PO3i Particulate organic aerosol associated with CG3
		PO4i Particulate organic aerosol associated with CG4
		PO5i Particulate organic aerosol associated with CG5
		PO6i Particulate organic aerosol associated with CG6
		PO7i Particulate organic aerosol associated with CG7
		POHi Particulate non‐volatile organic aerosol from aromatic precursors
		PPAi Anthropogenic organic aerosol polymers (SOPA)
		PPBi Biogenic organic aerosol polymers (SOPB)
		
Mercury
		HG0i Elemental Mercury vapor
		HG2i Reactive gaseous Mercury vapor
		PHGi Particulate Mercury  

Primary Particulates
		PECi Primary Elemental Carbon
		POAi Primary Organic Aerosol
		PFCi Fine Crustal PM
		PFNi Other Fine Particulate
		PCCi Coarse Crustal PM
		PCSi Other Coarse Particulate
```

臭氧和PNO3都和NOx的排放有关。OSAT3和PSAT中氮氧化物的示踪剂族是一致的，唯一的差异是PSAT中还需要示踪剂追踪颗粒物类别。



如果应用所有PM类型的话，对于每个源区域/组，PSAT总共包括40个示踪剂。因为SA过程并不总是需要所有类别，PSAT是非常灵活的，可以选择所有/部分化学类别进行源解析。比如，硫酸盐，硝酸盐和铵盐的来源解析，每个源区域/组仅需要10个示踪剂。



PSAT中的基本假设是：对于每一种PM类型，PM应该分配给其主要前体物。比如，PSO4应该分配给SOx排放，PNO3应该分配给NOx排放，PNH4应该分配给NH3排放等等，作为一种来源解析(source apportionment)方法，PSAT必须要能够解释PM类别的所有模式来源。假设有模式类别A和B，那么将分别分配反应示踪剂ai和bi。A和B的所有源(排放，初始和边界条件)应该都包括反应示踪剂，以此获取所有的来源解析。


## 运行SA

### CAMx控制文件

SA的运行类似CAMx控制文件中其它工具。如果要运行OSAT，APCA和PSAT，在配置文件中`&CAMx_Control`部分，变量`Probing_Tool`必须要设置为"SA"。而且还要额外配置`&SA_Control`参数，配置模式SA部分的参数。详细配置参数见说明文档。


Each partial source area map to be used in the run must be listed by source group and grid: e.g.,
Partial_Source_Area_Map(3,2) refers to SA emissions group 3 and grid 2.  These map files must
be listed in the same order as the group emission input files (i.e., the map assigned to category
1 must be consistent with the emissions assigned to category 1).

### 明确排放组

SA可以在一些排放类别/组中分配臭氧，PM和前体物。为了实现此功能，每个组的排放都必须提供单独的排放文件，包括主网格和嵌套网格的低层网格排放以及点源排放。额外的排放文件必须以CAMx网格和点源排放的文件形式提供，具体描述见Section 3。如果一个类别(category)没有提供点源，那么组(group)的点源文件名留空。如果一个类别没有格点排放，那么组的格点排放文件名留空。

> APCA至少需要两个排放组，第一个排放组必须是生物排放。

例如，在通过三个组追踪排放时，应该提供三个排放文件，并且其和应该等于提供给核心模型的常规CAMx文件的总排放。CAMx提供了一个可选项，即可仅提供两个排放文件，给定三个组，然后利用常规CAMx排放与两个排放组的差计算第三个组的排放。需要指定`Use_Leftover_Group`选项，当指定了`leftover`选项时，模式会进行验证，防止leftover组太小，导致在模式计算精度范围内无法计算；如果没有提供`leftover`选项，模式会验证所提供的组的总排放要等于常规模式排放，即不需要leftover组。在上述两种情况下，如果无法满足条件，模式会停止并输出错误信息。

### 源区域地图(Source Area mapping)

在模式区域内，SA可以在一些地图区域内分配臭氧，PM和前体物，如图1所示。SA需要一个模式区域的数字地图文件，定义了示踪剂如何进行空间分配。源区域地图文件将每一个网格分配给一个或多个源区域(geographic source regions)。**主网格(master grid)**必须定义源区域地图，**嵌套网格(nested grid)**可根据需要进行定义。源区域地图的格式对于所有网格而言都是相同的，但是对于嵌套层的地图而言，必须要包括边界的行和列(boundary("buffer") rows and cols)。**每个嵌套层网格所定义的源区域具有比主网格定义的源区域更高的优先级。**如果没有为嵌套网格定义源区域地图，那么将使用其父网格(parent grid)的源区域地图。


有两种方式可以定义源区域地图：

* **原始方法**：将整个格点划分给一个地理区域，然后分配此网格中所有的源类。
* **分割方法**：将每个网格分配给多个区域，例如，一些地理政治边界在一个网格中存在交叉。此外，可能会定义单独的细分区域地图，唯一的定义了SA所追踪的每一个排放类别的源区域的分布。

运行SA时，需要提供原始源区域地图，但是也可以通过细分源区域地图控制。如果没有为一个/多个源类别(source categories)提供细分源区域地图文件时，原始区域地图会为SA提供默认的区域定义。在CAMx输出诊断文件中包括了一个报告，允许用户查看SA的区域配置。


原始SA地图格式非常简单：就是特定CAMx网格的3位整数的数组。下图展示了和图1相对应的单个网格的源区域地图文件。因为在图1中，CAMx区域有63行64列，下图所展示的文件也有63行64列。左上角的第一个数对应的是模式区域西北角。可以利用GIS软件将模式网格覆盖到地理政治边界地图上，然后利用每个网格的覆盖区域作为源区域。

![](https://raw.githubusercontent.com/bugsuse/blogpic/master/2019/05/15/ch7_fig8.png)


如果每个网格中存在多个交叉区域，单个网格的细分区域地图可能包括多个面板(panel)，面板的总数由所有网格中区域重叠的最大数确定。如果一个特定网格单元包含了4个最大重叠区域，那么细分地图就包含4个面板，每个面板为其中一个区域，其表示在该网格单元中的覆盖范围。


细分SA地图文件具有如下格式：

```
Loop over number of panels
    /SRCMAPnn-mm/ 	                        	Header keyword, where nn is source
    Loop from ny grid rows to 1
	(regn(i,j),frc(i,j),i=1,nx) 			Loop over nx grid columns,
																							500(i3,1x,f5.1)
    End loop over rows
End loop over panels
/END/            					End of file keyword
```


整数变量数组`regn`是单元`cell(i,j)`中的区域索引，实数变量数组`frc`是区域在单元`cell(i,j)`所覆盖的分数(percent)。对于非0单元分数，`regn`和`frc`都必须要列出，否则`regn`为0，`frc`为空。当计算所有面板之和时，每个单元所有区域的覆盖面积要等于100%。下图给出了10x10的网格单元示例。

![](https://raw.githubusercontent.com/bugsuse/blogpic/master/2019/05/15/ch7_fig9.png)


无论是原始源区域地图还是细分源区域地图，可能都无法充分解析应该分配特定点源的区域。为了更好的控制对地理区域点源的分配，可以使用点源文件中的`kcell`变量数组为任意点源指定区域索引(文件描述见Section 3)。此功能称为"**点源覆盖**"(point source override)。

#### 由SMOKE报告生成细分区域地图

Fortran工具REGNMAP可以利用SMOKE处理系统得到的信息生成细分区域地图文件。详细信息见说明文档。

### 受体定义(Receptor Definition)

在所选的受体位置(溯源点)可以将示踪剂浓度以模式输出频率(通常为1h)输出到文本文件。每个模式运行所需要的受体都会定义在输入文件中。目前支持三种类型受体：

* `POINT`：以CAMx投影坐标系统指定的点，点的浓度通过周围的四个网格的值双线性插值得到
* `SINGLE CELL`：由网格索引标示的单个网格
* `CELL AVERAGE`：由一系列网格索引标示的网格组，通过平均计算多个单元的示踪剂平均浓度
* `WALL OF CELLS`：由一系列格点索引标示的组单元和层(layer)索引定义的wall


对于由网格单元所定义的受体类型，有必要在受体定义记录中指定包含受体的网格，网格编号通常使用CAMx内部定义的网格顺序定义。CAMx中所定义的网格号在 `.diag` 文件中。每个受体可以通过10个字符表示。每个受体类型的格式见下表：

![](https://raw.githubusercontent.com/bugsuse/blogpic/master/2019/05/15/ch7_tab2.png)

示例如下：

```
POINT                City 1      1024.0     -272.0
SINGLE CELL          Cell 1      1              45        18
CELL AVERAGE         Region 10   2               8
          31 19
          32 19
          33 19
          34 19
          31 18
          32 18
          33 18
          34 18
WALL OF CELLS         Boundary1        2          10       20
          18 18
           1 5 
```

### 输出文件格式

SA会输出一些CAMx Fortran二进制格式文件(描述见Section 3)，包括主网格和嵌套网格示踪剂瞬时浓度文件(`.sa.inst`和`.sa.finst`)以及特定网格面的示踪剂平均浓度文件(`.sa.grdnn`)，指定网格面沉积质量文件(`.sa.depn.grdnn`)。此外，SA还会输出所选的受体位置(溯源点)的示踪剂浓度到文本文件。示踪剂种类的命名规则和受体浓度文件的格式如下：

#### 示踪剂种类名称

示踪剂种类名称通常由每个种类的唯一信息和SA的配置决定。种类名称要少于10个字符，和CAMx的规范一致。命名规范如下：

```
Emission Sources SSSeeerrr
where:
    SSS Species type, e.g., NOX, VOC, O3V, O3N, PSO4, etc.
    eee Emissions group:
     Single group, always 000
     Multiple groups, 001, 002, etc.
    rrr Region tracer released from, 001, 002, 003, etc.
   
Initial/Boundary   SSSeeerrr
where:
    SSS Species type, e.g., NOX, VOC, O3V, O3N, PSO4, etc.
    eee Initial Concentrations: always 000  
    		Boundary Concentrations not stratified by boundary: always 000
    		Boundary Concentrations stratified by boundary: WST, EST, STH, NTH,
				TOP indicating boundary of origin
    rrr IC for Initial Concentrations, BC for Boundary Concentrations
Examples:   NOX000015, VOC002015, O3V000IC, O3NTOPBC
```

#### 受体浓度文件

用户自定义的受体位置的示踪剂浓度会输出到受体浓度文件。文件格式为逗号分隔的文本文件。文件头的前两行为模式版本和运行日期，紧接着SA的配置，示踪剂的信息等。

### 后处理

网格浓度文件中的示踪剂浓度后处理，可以使用任意的后处理工具展示CAMx的浓度信息。

受体浓度文件中包含了模式运行的所有运行时间的受体信息，需要用户开发后处理工具分析。

## 准备输入和SA运行

下面是设置和运行SA的简单步骤，OSAT/APCA和PSAT，DDM等工具的流程是类似的：

* 定义要追踪的源组和区域。**注意：内存资源会随着示踪剂数目的增加迅速增减。**使用了大量示踪剂，示踪类别和嵌套网格的工具可能会超过内存限制。
* 构建源区域地图定义示踪剂排放的空间分配。对于小的模拟域(domain)或者很少的区域，可以手动完成。针对大网格建议使用GIS软件生成复杂的源区域地图。
* 处理排放清单到想要追踪的源组文件(source group files)(即，移动，面，点，生物源等)。
  * 处理排放之前要考虑潜在的源分配或敏感性应用，这是非常有利的
  * 高程点源会自动分配到它们所在的源区域。然而，你也可以覆盖这些所分配的单独点源的区域(见Section 3 kcell的定义)。不需要在源区域地图文件中定义点源区域。
* 编辑CAMx控制文件(见Section2)
  * 设置 `Probing_Tool` 变量为 "SA"，这将激活namelist中的 `&SA_Control ` 模块
  * 编辑添加 `&SA_Control`  模块，包含如下信息：
    * 输出路径
    * 是否分层边界条件
    * 打开特定的臭氧或PM类别
    * 源区域数
    * 源组数
    * 是否使用leftover组选项
    * 受体定义
    * 组的输入排放文件
  * 注意APCA需要需要每个网格的生物源排放列在第一个
* 配置CAMx源码定义示踪剂的数目，然后编译，这将确保有充足的内存进行计算
  * 编辑`Includes/camx.prm`
  * 改变`MXTRSP`，CAMx发布时默认`MXTRSP=1`，这对于标准的模式应用而言所需内存最小。如果你使用了一个不恰当的值运行SA，然后模式会终止，并告诉你可能要设置的`MXTRSP`的值
  * 执行CAMx中的Makefile文件构建可执行程序
* 运行CAMx并检查诊断输出文件确保模式正确运行，并且使用了你的配置运行了诊断工具。
* 使用合适的绘图工具检查网格示踪剂场
* 分析SA受体文件
* 使用后处理工具分析网格示踪剂输出文件


## 更新记录    

2019.05.15 更新基本指南

