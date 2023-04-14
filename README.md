# LEDMaster

孔乙己说回字有四种写法，而使用 STM32 点灯可不止有四种方式。本文利用 STM32F429IGT6 + STM32CubeMX + Keil5，通过各种外设，实现多种点灯方式。

> 本系列代码均已开源至 Github，欢迎 Fork&Star。

### 例程 1：使用 GPIO 点灯

**要求**：利用 STM32CubeMX 和 Keil5 MDK 进行 STM32 应用开发，完成以下功能。

- 实现 LED0、LED1 闪烁。

**步骤**：

1. 打开 STM32CubeMX 软件，按照 `STM32CubeMX 通用配置` <sup>[1]</sup>创建工程、后。在可视化界面的搜索框中输入 PB1，即可快速定位到引脚位置。

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230404223950257.png" alt="image-20230404223950257" style="zoom:57%;" />

<center>图 1.1 引脚位置</center>

2. 点击该引脚，可以看到多个选项，选择 `GPIO_Output`。PB0 与 PB 1 两个引脚都选上，使用中的引脚会变成绿色。

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230404224314725.png" alt="image-20230404224314725" style="zoom:72%;" />

<center>图 1.2 LED 灯引脚配置</center>

3. **生成代码**。

4. 按路径打开工程文件。按照 `Keil5 MDK 通用配置` <sup>[1]</sup>完成后开始编写代码。

   打开 `main.c` 文件。

   ```c
   /* Infinite loop */
   /* USER CODE BEGIN WHILE */
   while (1)
   {
       /* LED1 配置 */
       /* 输入低电平，点亮 LED1 */
       HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET);
       HAL_Delay(200);
       /* 输入高电平，熄灭 LED1 */
       HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET);
       HAL_Delay(200);
   
       /* LED0 配置 */
       /* 翻转电平，使 LED0 闪烁*/
       HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_1);
       HAL_Delay(200);
         
       /* USER CODE END WHILE */
   
       /* USER CODE BEGIN 3 */
   }
   /* USER CODE END 3 */
   ```

5. **下载程序**。

### 例程 2：使用按键点灯

**要求**：利用 STM32CubeMX 和 Keil5 MDK 进行 STM32 应用开发，完成以下功能：

1. 按下 KEY0（PH3）按键，松开后，切换 LED0（PB1）的开关状态。
2. 按下 KEY1（PH2）按键，切换 LED1（PB0）的开关状态。
3. 按下 KEY2（PC13）按键，把点亮的 LED 灯全部关闭。

**步骤**：

1. 打开 STM32CubeMX 软件，按照 `STM32CubeMX 通用配置`<sup>[1]</sup>完成后。将 PB0、PB1 引脚设置为 `GPIO_Output`，将 PH3、PH2、PC13 引脚设置为 `GPIO_Input`。

| ![image-20230406124013141](https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406124013141.png) | ![image-20230406124026659](https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406124026659.png) | ![image-20230406124041053](https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406124041053.png) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |

<center>图 2.1 LED 灯与按键引脚配置</center>

2. 将 PH3、PH2、PC13 的 GPIO 都改为上拉（Pull-up）。（**注**：将GPIO口设置为上拉输入模式，意思是当GPIO口没有外部信号输入时，将其上拉到高电平（即VCC电源电压）状态，从而避免GPIO口悬浮，导致输入不稳定或产生噪声。）

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406130602990.png" alt="image-20230406130602990" style="zoom:80%;" />

<center>图 2.2 GPIO 引脚配置</center>

3. **生成代码**。

4. 打开工程文件。按照 `Keil5 MDK 通用配置` 完成后开始编写代码。

   ① 打开 `main.c` 文件。

   ```c
   /* USER CODE BEGIN 0 */
   
   /* HAL_Delay() 经常会导致程序卡死，自己写一个延时函数 */
   void Delay(unsigned int t)
   {
   	while(t--);
   }
   
   /* 功能函数：扫描按键 */
   void Scan_Keys()
   {
       /* 配置 KEY0--LED0*/
       /* 原理：由于将按键引脚全部配置为上拉，所以按下时会产生低电平(RESET)，检测到低电平执行下面程序 */
       if(HAL_GPIO_ReadPin(GPIOH,GPIO_PIN_3) == GPIO_PIN_RESET)
       {
           Delay(1000); //去抖动，防止误触
           if(HAL_GPIO_ReadPin(GPIOH,GPIO_PIN_3) == GPIO_PIN_RESET)
           {
               /* while() 是为了保证按下后进入循环，直到松开，程序才会向下执行，即松开后切换 LED0 的状态 */
               while(HAL_GPIO_ReadPin(GPIOH,GPIO_PIN_3) == GPIO_PIN_RESET);
               HAL_GPIO_TogglePin(GPIOB,GPIO_PIN_1);
           }
       }
   }
   
   /* USER CODE END 0 */
   ```

   ② 在 `main` 函数中执行 `Scan_Keys` 功能。

   ```c
   /* Infinite loop */
   /* USER CODE BEGIN WHILE */
   while (1)
   {
       Scan_Keys();
       /* USER CODE END WHILE */
   
       /* USER CODE BEGIN 3 */
   }
   /* USER CODE END 3 */
   ```

   > （补充知识）**宏定义**：当执行到 KEY0 时，就会去找到 KEY0 的定义并执行。
   >
   > ```c
   > # define KEY0 HAL_GPIO_ReadPin(GPIOH,GPIO_PIN_3)
   > ```
   >
   > ```c
   > /* GPIO_PIN_RESET 已经被定义好了为 0，相应的 GPIO_PIN_SET = 1。 */
   > /* 右键点击 GPIO_PIN_RESET，选择 Go To Definition of 'GPIO_PIN_RESET' 可以看到源码如下: */
   > 
   > typedef enum
   > {
   > GPIO_PIN_RESET = 0,
   > GPIO_PIN_SET
   > }GPIO_PinState;
   > ```
   >
   > （补充知识）可以在 `gpio.c` 文件中通过结构体来配置引脚。

   ① 完整代码。延时程序和扫描按键功能：

   ```c
   /* USER CODE BEGIN 0 */
   # define KEY0 HAL_GPIO_ReadPin(GPIOH,GPIO_PIN_3)
   # define KEY1 HAL_GPIO_ReadPin(GPIOH,GPIO_PIN_2)
   # define KEY2 HAL_GPIO_ReadPin(GPIOC,GPIO_PIN_13)
   
   /* HAL_Delay() 经常会导致程序卡死，自己写一个延时函数 */
   void Delay(unsigned int t)
   {
   	while(t--);
   }
   
   /* 功能函数：扫描按键 */
   void Scan_Keys()
   {
       /* 原理：由于将按键引脚全部配置为上拉，所以按下时会产生低电平(RESET)，检测到低电平执行下面程序 */
   
       /* 配置 KEY0--LED0*/
       if(KEY0 == 0)
       {
           Delay(1000); //去抖动，防止误触
           if(KEY0 == 0)
           {
               /* while() 是为了保证按下并抬起只执行一次程序，如果一直按着，那么就一直都是低电平，程序就不会向下执行 */
               while(KEY0 == 0);
               HAL_GPIO_TogglePin(GPIOB,GPIO_PIN_1);
           }
       }
   
       /* 配置 KEY1--LED1*/
       if(KEY1 == 0)
       {
           Delay(1000); //去抖动，防止误触
           if(KEY1 == 0)
           {
               HAL_GPIO_TogglePin(GPIOB,GPIO_PIN_0);
               while(KEY1 == 0);
           }
       }
   
       /* 配置 KEY3--LED0|LED1 */
       if(KEY2 == 0)
       {
           Delay(1000); //去抖动，防止误触
           if(KEY2 == 0)
           {
               HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0|GPIO_PIN_1, 1);
               while(KEY2 == 0);
           }
       }
   }
   /* USER CODE END 0 */
   ```

   ② 在 `main` 函数的 while 中实现扫描按键功能。

   ```c
   /* Infinite loop */
   /* USER CODE BEGIN WHILE */
   while (1)
   {
       Scan_Keys();
       /* USER CODE END WHILE */
   
       /* USER CODE BEGIN 3 */
   }
   /* USER CODE END 3 */
   ```

5. **下载程序**。

### 例程 3：使用外部中断点灯

**要求**：利用 STM32CubeMX 和 Keil5 进行 STM32 应用开发，完成以下功能：

1. 将 KEY0（PH3） 设置为外部中断输入，下降沿触发。在中断服务函数中，切换 LED0（PB1）的开关状态。
2. 将 KEY1（PH2） 设置为外部中断输入，上升沿触发。在中断服务函数中，切换 LED1（PB0） 的开关状态。

**步骤**：

1. 打开 STM32CubeMX 软件，按照 `STM32CubeMX 通用配置` 完成后。将 PB0、PB1 引脚设置为 `GPIO_Output`，将 PH3、PH2 引脚设置为 `GPIO_EXTI3`。

| <img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406213714766.png" alt="image-20230406213714766"  /> | ![image-20230406213957774](https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406213957774.png) |
| :----------------------------------------------------------: | :----------------------------------------------------------: |

<center>图 3.1 LED 灯与按键引脚配置</center>

2. 在 `GPIO` 中将 PH2、PH3 的配置都改为上拉（Pull-up）。同时，PH3 的 `GPIO mode` 改为外部中断下降沿触发，PH2 的 `GPIO mode` 改为外部中断上升沿触发。（**注**：上升沿触发指的是当外部中断引脚输入的信号由低电平变为高电平时，触发中断事件；而下降沿触发则指的是当外部中断引脚输入的信号由高电平变为低电平时，触发中断事件。）

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406214723072.png" alt="image-20230406214723072" style="zoom:60%;" />

<center>图 3.2 按键引脚中断配置</center>

3. 使能中断。

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406215450172.png" alt="image-20230406215450172" style="zoom:61%;" />

<center>图 3.3 使能中断配置</center>

4. **生成代码**。

5. 打开工程文件。按照 `Keil5 MDK 通用配置` 完成后开始编写代码。

   > *了解代码结构*。
   >
   > 1. 打开 `main.c` 文件，其【main()】函数中，有【MX_GPIO_Init()】语句，表示 GPIO 的初始化，右键点进它的定义。来到 `gpio.c` 文件，在【MX_GPIO_Init()】函数中可以看到 *EXTI interrupt init*。
   >
   >    ```c
   >    HAL_NVIC_SetPriority(EXTI2_IRQn, 0, 0); //第一行：设置优先级
   >    HAL_NVIC_EnableIRQ(EXTI2_IRQn); //第二行：使能
   >    ```
   >
   > 2. 打开 `stm32f4xx_it.c` 文件，拉到最下面，可以看到 PH2、PH3 的中断服务函数【HAL_GPIO_EXTI_IRQHandler()】，右键点进它的定义，来到 `stm32f4xx_hal_gpio.c` 文件，可以看到在中断服务函数里面，有一个【HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)】函数，右键点进它的定义，来到下面的虚函数【__weak void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)】，可以看到里面什么都没有做，需要重写它。一般将其拷到 `main.c` 文件中重写。

   打开 `main.c` 文件。

   ```c
   /* USER CODE BEGIN 0 */
   void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
   {
   	/* KEY0--LED0 配置 */
   	/* 判断传进来的引脚是否为 PH3 */
   	if(GPIO_Pin == GPIO_PIN_3)
   	{
   		HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_1);
   	}
       
       /* KEY1--LED1 配置 */
   	/* 判断传进来的引脚是否为 PH2 */
   	if(GPIO_Pin == GPIO_PIN_2)
   	{
   		HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_0);
   	}
   }
   /* USER CODE END 0 */
   ```

   #### 掌握调试技巧

   > 1. 加断点。在 `stm32f4xx_it.c` 文件中的【HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_3)】前面加上断点。因为我们重写了虚函数，中断响应后，会首先来到这个函数。
   > 2. 进入调试模式。①全速运行。②按下 KEY0 按键，可以看到代码运行到了函数【HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_3)】这里。
   >
   > <img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406224730587.png" alt="image-20230406224730587" style="zoom:80%;" />
   >
   > <center>图 3.4 调试步骤-1</center>
   >
   > 3. 选择单步走，可以看到现在进入了 `stm32f4xx_hal_gpio.c` 文件中的【HAL_GPIO_EXTI_IRQHandler()】函数中的 if 语句。
   >
   > <img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406225129219.png" alt="image-20230406225129219" style="zoom:70%;" />
   >
   > <center>图 3.5 调试步骤-2</center>
   >
   > 4. 继续单步走。当执行【HAL_GPIO_EXTI_Callback()】回调函数时，进入到了 `main.c` 文件中。然后继续执行，发现 LED0 发生翻转。
   >
   > <img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406225710807.png" alt="image-20230406225710807" style="zoom:90%;" />
   >
   > <center>图 3.6 调试步骤-3</center>

6. **下载程序**。

### 例程 4：使用定时器中断点灯

**要求**：在 STM32F429 进行 STM32 应用开发，完成以下功能：

1. 利用 TIM2 实现间隔定时，每隔 0.2s 将 LED0（PB1）的开关状态翻转。

2. 利用 TIM3 实现间隔定时，每隔 1s 将 LED1（PB0）的开关状态翻转。

> *计算公式*：定时器发生中断的时间
>
> 定时时间 = （PSC + 1）*（ARR + 1）/ 定时器时钟频率
>
> - PSC：Prescaler，预分频器
> - ARR：重装载值，写在 Counter Period 中。
>
> *练习题*：时钟信号频率 1KHz，PSC = 9，ARR = 999，求定时时间。
>
> - `9` 的意思是预分频器接收 10 个脉冲就会溢出。如下图，一个脉冲 1ms，PSC 每接收到 10 个脉冲，也就是 10ms，传给主计数器一次。
>
> - `999` 的意思是主计数器的最大计数值为 1000。如下图，PSC 传给主机数器一个脉冲是 10ms，主计数器一共能接收 1000 个，也就是 10000ms = 10s，然后传给内核。
> - 综上所述，此处是每 10s 产生一次中断。
>
> <img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20221024105011992.png" alt="img" style="zoom:80%;" />
>
> <center>图 4.1 计算过程</center>

**步骤**：

1. 打开 STM32CubeMX 软件，按照 `STM32CubeMX 通用配置` 完成后，操作通用计时器 TIM2、TIM3。

   > 计算：时钟频率为 84MHz，
   >
   > - TIM2：$ \frac{8400\times2\times10^3}{84\times10^6}=0.2s=200ms $
   > - TIM3：$ \frac{8400\times1\times10^4}{84\times10^6}=1s $

| ![image-20230406234448736](https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406234448736.png) | ![image-20230406234557793](https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230406234557793.png) |
| :----------------------------------------------------------: | :----------------------------------------------------------: |

<center>图 4.2 TIM2&3 配置</center>

2. 使能 NVIC 中断。

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230407104212315.png" alt="image-20230407104212315" style="zoom:120%;" />

<center>图 4.3 NVIC 使能</center>

3. 配置 GPIO 向 LED 灯输出高低电平。（同 `例程 1：使用 GPIO 点灯` 第 2 步）

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230404224314725.png" alt="image-20230404224314725" style="zoom: 66%;" />

<center>图 4.4 LED 灯引脚配置</center>

3. **生成代码**。

4. 打开工程文件。按照 `Keil5 MDK 通用配置` 完成后开始编写代码。

   > 写中断服务函数，需要先找到对应的虚函数。
   >
   > - 打开 `stm32f4xx_it.c` 文件，拉到最下面，可以看到 TIM2、TIM3 的中断服务函数【HAL_TIM_IRQHandler(&htim2)、HAL_TIM_IRQHandler(&htim3)】。随便右键点一个进入定义，来到 `stm32f4xx_hal_tim.c` 文件中，在【HAL_TIM_IRQHandler()】函数中，有很多复杂的功能，找到定时器中断回调函数【HAL_TIM_PeriodElapsedCallback(htim)】，右键点进定义，将原型拷贝到 `main.c` 文件中。
   >
   > 开启定时器中断。
   >
   > - 打开 `stm32f4xxhal_hal_tim.c` 文件，（在 400~500 行之间）找到【HAL_TIM_Base_Start_IT(TIM_HandleTypeDef *htim)】函数，将其拷贝到 `main.c` 文件中，在【main】函数中初始化。

   打开 `main.c` 文件。

   ① 在【main】函数外重写中断服务函数：

   ```c
   /* USER CODE BEGIN 0 */
   void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
   {
   	/* TIM2--LED0 */
   	if(htim->Instance == TIM2)
   	{
   		HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_1);
   	}
       
       /* TIM3--LED1 */
       if(htim->Instance == TIM3)
       {
           HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_0);
       }
   }
   /* USER CODE END 0 */
   ```

   ② 在【main】函数中开启定时器中断。

   ```c
   /* USER CODE BEGIN 2 */
   HAL_TIM_Base_Start_IT(&htim2);
   HAL_TIM_Base_Start_IT(&htim3);
   /* USER CODE END 2 */
   ```

5. **下载代码**。

### 例程 5：使用定时器输出 PWM 实现呼吸灯

**要求**：在 STM32F429 进行 STM32 应用开发，完成以下功能：

- 利用 TIM3 输出 PWM 信号，使 LED0、LED1 变成呼吸灯效果。

**步骤**：

1. 打开 STM32CubeMX 软件，按照 `STM32CubeMX 通用配置` 完成后。将 PB0、PB1 引脚分别设置为 `TIM3_CH3`、`TIM3_CH4`。

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230404234417624.png" alt="image-20230404234417624" style="zoom:72%;" />

<center>图 5.1 LED 灯引脚配置</center>

2. 配置 TIM3 的 CH3、CH4，完成后可以看到右边 PB0、PB1 引脚变为绿色。

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230404234944897.png" alt="image-20230404234944897" style="zoom:58%;" />

<center>图 5.2 TIM3 Channel 配置</center>

3. **生成代码**。

4. 打开工程文件。按照 `Keil5 MDK 通用配置` 完成后开始编写代码。

   打开 `main.c` 文件。在 `main` 函数中开启 PWM。

   ```c
   /* USER CODE BEGIN 2 */
   HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_3);
   HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_4);
   /* USER CODE END 2 */
   ```

   输出 PWM。

   ```c
   /* Infinite loop */
   /* USER CODE BEGIN WHILE */
   while (1)
   {
       /** LED0 配置 **/
       /* LED0 逐渐熄灭 */
       for(uint16_t pwmVal=0; pwmVal<1000; pwmVal++)
       {
           __HAL_TIM_SetCompare(&htim3, TIM_CHANNEL_4, pwmVal);
           HAL_Delay(1);
       }
       /* LED0 逐渐点亮 */
       for(uint16_t pwmVal=1000; pwmVal>0; pwmVal--)
       {
           __HAL_TIM_SetCompare(&htim3, TIM_CHANNEL_4, pwmVal);
           HAL_Delay(1);
       }
   
       /** LED1 配置 **/
       /* LED1 逐渐熄灭 */
       for(uint16_t pwmVal=0; pwmVal<1000; pwmVal++)
       {
           __HAL_TIM_SetCompare(&htim3, TIM_CHANNEL_3, pwmVal);
           HAL_Delay(1);
       }
       /* LED1 逐渐点亮 */
       for(uint16_t pwmVal=1000; pwmVal>0; pwmVal--)
       {
           __HAL_TIM_SetCompare(&htim3, TIM_CHANNEL_3, pwmVal);
           HAL_Delay(1);
       }
       /* USER CODE END WHILE */
   
       /* USER CODE BEGIN 3 */
   }
   /* USER CODE END 3 */
   ```

5. **下载代码**。

### 例程 6：使用串口通信点灯

**要求**：在 STM32F429IGT6 中进行 STM32 应用开发，完成以下功能。

- 开机后，向串口 1 发送”Hello World!”。
- 串口 1 收到字节指令”0xA1”，打开 LED0（PB1），发送”LED0 Opened!”。
- 串口 1 收到字节指令”0xA2”，关闭 LED0（PB0），发送”LED0 Closed!”。
- 在串口发送过程中，打开 LED1 作为发送数据指示灯。

**步骤**：

1. 打开 STM32CubeMX 软件，按照 `STM32CubeMX 通用配置`配置完成后。将 PB0、PB1 引脚分别设置为 `GPIO_Out`。接着配置 USART1 的模式、参数及使能中断，中断的优先级设置为 1，如图所示：

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230410152306321.png" alt="image-20230410152306321" style="zoom:45%;" />

<center>图 6.1 串口 USART1 配置</center>

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230413114558206.png" alt="image-20230413114558206" style="zoom:87%;" />

<center>图 6.2 使能 USART1 中断</center>

2. **生成代码**。

3. 打开工程文件。按照 `Keil5 MDK 通用配置` 完成后开始编写代码。

   > 打开 `usart1.c` 文件，可以看到 huart1 的配置。
   >
   > <img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230410153414008.png" alt="image-20230410153414008" style="zoom:;" />
   >
   > <center>图 6.3 USART1 的代码初始化</center>

   打开 `main.c` 文件。

   ```c
   -----------------------
   ↓↓↓注：main() 函数外↓↓↓
   ----------------------- 
   /* USER CODE BEGIN 0 */
   
   /* 宏定义 LED0、LED1 亮灭 */
   #define LED0_ON() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_RESET);
   #define LED0_OFF() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);
   #define LED1_ON() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET);
   #define LED1_OFF() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET);
   
   /* 首先定义了无符号 8 位整数型（unsigned 8-bit integer）数组的语句，常用于表示一串 ASCII 字符。其中，Tx_str1 是数组的名称，方括号中没有指定数组长度，因此该数组长度将根据初始化时赋值的元素个数进行确定 */
   uint8_t Tx_str1[] = "Hello World!\r\n"; // \r：回车，\n：换行
   uint8_t Tx_str2[] = "LED1 Opened!\r\n";
   uint8_t Tx_str3[] = "LED1 Closed!\r\n";
   
   /* 定义了一个无符号8位整数型（unsigned 8-bit integer）变量,通常用于存储通过UART接收到的单个字节数据，每次接收完成后将数据存储到该变量中 */
   uint8_t Rx_dat = 0;
   /* 中断回调函数, 在这里面实现接收到指令后的功能*/
   void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
   {
   	if(huart->Instance == USART1)
   	{
   		if(Rx_dat == 0xa1)
   		{
   			LED0_ON();
   			
               /* LED1 作为指示灯，先设置高电平灭掉，然后发送数据，再点亮 */
   			LED1_OFF();
   			HAL_Delay(200);
   			HAL_UART_Transmit(&huart1, Tx_str2, sizeof(Tx_str2), 10000);
   			HAL_Delay(200);
   			LED1_ON();
   			
   			HAL_UART_Receive_IT(&huart1, &Rx_dat, 1);
   		}
   		else if(Rx_dat == 0xa2)
   		{
   			LED0_OFF();
   			
               /* LED1 作为指示灯，先设置高电平灭掉，然后发送数据，再点亮 */
   			LED1_OFF();
   			HAL_Delay(200);
   			HAL_UART_Transmit(&huart1, Tx_str3, sizeof(Tx_str3), 10000);
   			HAL_Delay(200);
   			LED1_ON();
   			
   			HAL_UART_Receive_IT(&huart1, &Rx_dat, 1);
   		}
   	}
   }
   
   /* USER CODE END 0 */
   
   -----------------------
   ↓↓↓注：main() 函数内↓↓↓
   ----------------------- 
   /* USER CODE BEGIN 2 */
   
   /* 功能 1：开机后向串口 1 发送 “Hello World!” */
   /* LED1 作为指示灯，先设置高电平灭掉，然后发送数据，再点亮 */
   LED1_OFF();
   HAL_Delay(200); //加入延时函数，以便肉眼观察得到
   HAL_UART_Transmit(&huart1, Tx_str1, sizeof(Tx_str1), 10000);
   HAL_Delay(200); //加入延时函数，以便肉眼观察得到
   LED1_ON();
   
   /* 功能 2、3：接收串口发来的数据” */
   /* &huart1 表示指向 UART1 外设的指针，&Rx_dat 表示指向存储接收数据的缓冲区的指针， 1 表示要接收的数据长度 */
   HAL_UART_Receive_IT(&huart1, &Rx_dat, 1);
   
   /* USER CODE END 2 */  
   ```

   > *串口调试助手操作要点*
   >
   > 1. MicroUSB 接口要接到板子的左下角第二个口，写着 USB_232。才会显示 `COM4：USB-SERIAL`。`COM3` 不能用于串口调试。
   > 2. 波特率要和 CubeMX 中设置的一致。
   > 3. 在发送框中输入程序中所写的字符串，然后选择 `16 进制发送`。

<img src="https://storybeginswhenicu.oss-cn-hangzhou.aliyuncs.com/img/image-20230410181007350.png" alt="image-20230410181007350" style="zoom:75%;" />

<center>图 6.4 功能实现</center>

### 参考

[1] [STM32CubeMX-Keil 通用配置 | Story Begins…… )](https://storybeginswhen.icu/2023/04/11/059-STM32-开发通用配置/)
