# WRF模式嵌套模拟


模式嵌套运行是为了得到更高分辨率的模拟。嵌套层完全包含在父层(Parent Domain, 即粗糙层, Corase Domain)中。嵌套层的边界条件来自父层(上一层)。使用嵌套层方式模拟可以无需在更大的模拟域中使用更高分辨率进行模拟，可以节省计算资源，只需要在更小的模拟域中执行高分辨率模拟，并且外层和内层的起始时间和空间边界条件不同。

模式嵌套模拟有几种不同的设置方式：

### Two-way Nesting

Two-way嵌套模拟可以同时运行具有不同网格分辨率的模拟域，并且各模拟域之间可以相互通信(模拟域之间存在反馈作用)。粗糙模拟域为嵌套层提供边界条件，而嵌套层计算后反馈到粗糙层。可以通过 namelist.input 中的 feedback 参数控制(设置 feedback = 1)：

#### 使用单个输入文件运行 two-way嵌套

  此情况的预处理步骤类似单模拟域设置。当运行多模拟域(至少2层)模拟时，其和单模拟域唯一的差别在执行wrf.exe时。

  嵌套层不需要输入文件。嵌套层所需要的静态数据和气象数据都由粗糙层的插值而来。

  **优势**：嵌套层可以从不同的时间启动。

  **缺点**：嵌套层无法使用高分辨率的静态场数据。

  **工作流程**：

  ![](https://github.com/bugsuse/blogpic/blob/master/2018/11/19/flowchart_2way_1input.png?raw=true)

  **注意：**步骤A之前的流程和单层模拟相同。

#### 使用两个输入文件运行two-way嵌套

  wrf运行之前，这种情况的预处理过程会创建嵌套层所需要的输入文件。在WRF模式步骤中，对于输入数据有以下两种方式选择：

  * **使用所有气象数据和静态数据作为嵌套层的输入**，或者
  * **仅使用静态数据作为嵌套层的输入**

  > ⚠️
  >
  >  这里的静态数据是插值到模拟域中的地理信息数据
  >
  > 气象数据是初始场数据，可以是FNL，或者GFS等初始场插值到模拟域

  由于模式在相同时刻启动，因此每一层都需要输入文件，每个模拟域都要有 wrf_input_d0x，x表示模拟域的id。

  以此方式运行WRF模式时，嵌套层的输入数据有两种选择：

  * 应用所有气象数据和静态数据到嵌套层

    此情况中嵌套层的所有静态数据和气象数据均来自嵌套层的输入文件

    **注意**：嵌套模拟时推荐使用此方式。

  * 仅使用静态数据作为嵌套层的输入

    此情况中只有静态数据来自嵌套层输入文件，而气象数据由粗糙层插值而来。

    **注意：**只有当嵌套层起始时间晚于父层时间时，才使用此方式。

  工作流程：

  ![](https://github.com/bugsuse/blogpic/blob/master/2018/11/19/flowchart_2way_2input.png?raw=true)

  注意：

  * <font color='blue'>步骤A</font>：WPS可以为嵌套层产生多个 met_em.d02.*，但是只需要初始时刻的文件。
  * <font color='blue'>步骤B</font>：此步之前的过程对于上述两个输入数据选择方式都需要。
  * <font color='blue'>步骤C</font>：嵌套层总是从粗糙层获取边界条件，因此不会创建 wrfbdy_d02文件

  **运行示例：**

  * ungrib.exe: 提取气象场
  * geogrid.exe: 插值地理数据到模拟域

  上述两步和正常的多层嵌套相同，主要的差别在于 metgrid 的运行：

  * metgrid.exe : 插值输入数据到模拟域

    * 设置 namelist.wps 参数

      **其余参数和多层嵌套类似，主要在于内层嵌套的结束时间。**

      ```bash
      start_date = '2016-06-22_00:00:00','2016-06-22_00:00:00',
      end_date = '2016-06-24_00:00:00','2016-06-24_00:00:00',
      ```

      **注意**: 由于内层嵌套只需要初始时刻的信息，所以这里 d02 的结束时间设置为起始时间。你也可以选择为 d02 创建所有时刻的 metgrid 文件，但是并不会使用这些数据。

    * 运行 metered.exe 插值输入数据到模拟域

  1) **同时使用静态数据和气象数据**

  * 编辑 namelist.input (确保处于WRFV3/run的运行目录)

    ```bash
    run_days = 0,
    run_hours = 12,
    run_minutes = 0,
    run_seconds = 0,
    start_year = 2016, 2016,
    start_month = 06, 06, 
    start_day = 22, 22, 
    start_hour = 00, 00, 
    end_year = 2016, 2016, 
    end_month = 06, 06, 
    end_day = 24, 24, 
    end_hour = 00, 00,
    interval_seconds = 21600
    input_from_file = .true., .true., 
    history_interval = 180, 60, 
    frames_per_outfile = 1000, 1000,
    restart = .false.,
    restart_interval = 5000,
    time_step = 180,
    max_dom = 2, 
    s_ we = 1, 1, 
    e_we = 98, 118, 
    s_sn = 1, 1,
    e_sn = 70, 103,
    s_vert = 1, 1, 
    e_vert = 30,30, 
    num_metgrid_levels = 27
    dx = 30000, 10000, 
    dy = 30000, 10000,
    i_parent_start = 1, 35,
    j_parent_start = 1, 23,
    parent_grid_ratio = 1, 3, 
    parent_time_step_ratio = 1, 3, 
    feedback = 1,
    ```

    **注意**：这种方式和只有一种输入数据的方式唯一的差别就在于d02的 input_from_file=.true.  参数。

  2) **嵌套层中仅使用静态数据**

  * 编辑 namelist.input 

    ```bash
    run_days = 2,
    run_hours = 0,
    run_minutes = 0,
    run_seconds = 0,
    start_year = 2016, 2016,
    start_month = 06, 06, 
    start_day = 22, 22, 
    start_hour = 00, 00, 
    end_year = 2016, 2016, 
    end_month = 06, 06, 
    end_day = 24, 24, 
    end_hour = 00, 00,
    interval_seconds = 21600
    input_from_file = .true., .true., 
    fine_input_stream = 0, 2, 
    history_interval = 180, 60, 
    frames_per_outfile = 1000, 1000,
    restart = .false.,
    restart_interval = 5000,
    time_step = 180,
    max_dom = 2, 
    s_ we = 1, 1, 
    e_we = 98, 118, 
    s_sn = 1, 1,
    e_sn = 70, 103,
    s_vert = 1, 1, 
    e_vert = 28, 28, 
    num_metgrid_levels = 27
    dx = 30000, 10000, 
    dy = 30000, 10000,
    i_parent_start = 1, 35,
    j_parent_start = 1, 23,
    parent_grid_ratio = 1, 3, 
    parent_time_step_ratio = 1, 3, 
    feedback = 1,
    ```

    **注意**: 这种方式和同时使用静态数据和气象数据的方式唯一的差别在于：d02的 fine_input_stream**=**2  参数。

### One-way Nesting

  One-way嵌套虽然同样可以让不同分辨率的模拟域同时运行，但是只有来自粗糙层的通信，嵌套层的计算结果并不会反馈到粗糙层。这种方法和two-way的运行方式相同，但是namelist.input中的 feedback=0。

#### 使用 ndown 运行 one-way 嵌套

    ndown可以在父层模拟域运行完成之后执行 one-way 嵌套。

  **注意:** 只有在你已经针对粗糙域之行了长时间的模拟，才推荐使用 ndown。然后再使用更高分辨率模拟来决定嵌套是否有利。如果你还没有运行粗糙域模拟，那么建议使用two-way或者上面提到的 one-way方法进行模拟。

### 参考链接：
1. http://www2.mmm.ucar.edu/wrf/OnLineTutorial/CASES/NestRuns/index.php
2. http://www2.mmm.ucar.edu/wrf/OnLineTutorial/CASES/NestRuns/2way1input.php
3. http://www2.mmm.ucar.edu/wrf/OnLineTutorial/CASES/NestRuns/wrf_2way2inputs_static.php




