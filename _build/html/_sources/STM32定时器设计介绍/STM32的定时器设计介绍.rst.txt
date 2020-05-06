STM32的定时器设计介绍
===============================

以 `STM32F103RC`_ 为例,ST定义了三种定时器类型。

.. _STM32F103RC: https://www.st.com/resource/en/reference_manual/cd00171190-stm32f101xx-stm32f102xx-stm32f103xx-stm32f105xx-and-stm32f107xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

- Basic timers(基础定时器)
- General-purpose timers(通用目的定时器)
- Advanced-control timers(增强型定时器)


Basic timers(基础定时器)
---------------------------------------

1. Feature
>>>>>>>>>>>>>>>>>>>
- 16-bit auto-reload upcounter
- 16-bit 分频器
- 与DAC共享部分电路以驱动DAC
- 计数超时会产生DMA Request/Interrupt

2. Block Diagram
>>>>>>>>>>>>>>>>>>>
|Block Diagram| 

.. |Block Diagram| image:: BasicTimer.PNG

基础定时器提供了基本的功能，外部时钟在分频之后进入到基础定时器中，当基础定时器的CNT计数达到Auto-reload Register数值的时候
产生中断/DMA请求。基础定时器仅支持上升计数模式，当达到ARR的时候自动清零。

3. Register Design
>>>>>>>>>>>>>>>>>>>
- TIMx_CR1
    + ARPE: 选择当溢出事件产生是否重新装载ARR的值
    + OPM:  是否开启one-pulse模式，决定溢出事件产生后是否停止计数
    + URS:  选择触发interrupt/DMA request的事件，是仅overflow/underflow事件还是包含其他
    + UDIS: interrupt/DMA request事件的使能位
    + CEN:  计数使能
- TIMx_CR2
    + MMS:  当基础定时器作为主定时器为从模块提供时钟的时候，选择时钟事件，是Reset事件或者是CNT_EN事件或者是分频器update事件
- TIMx_DIER
    + UDE/UIE: interrupt以及DMA的使能
- TIMx_SR
    + UIF: 中断标志位
- TIMx_EGR
    + UG:清理CNT的数据，重新计数
- TIMx_CNT
    + CNT:计数值
- TIMx_PSC
    + PSC[15:0]: 分频值
- TIMx_ARR
    + ARR: 预加载值

4. Application
>>>>>>>>>>>>>>>>>>>

- 提供固定事件的中断信号
- 为DAC/ADC等其他模块提供工作时钟

5. 优势与我们可借鉴的方面
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

- 此模块比较简单，仅支持上升计数模式，仅有一个比较值，产生固定时间的计数信号，可以把这种简单的基础计数器模块当作一个其他设备的时钟分频器。
- 我们可以借鉴的点是，是否可以设计这种基础计数器为其他的外设提供更加灵活的时钟？
- 之前ADC的模块存在一个问题，ADC的输入时钟在BL602设计里面只支持1 4 8 16 256 等分频选项，导致了如果输入的时钟不是由专门的PLL时钟电路产生的2.048M时钟，那么mic应用会拿不到标准的8K采样率，因为ADC会做一个64，128，或256次平均，相当于分频，使得最终的ADC时钟取不到整数，如果TIMER模块支持Basic Timer 并且这个Basic Timer可以成为ADC的输入时钟，那么就可以解决这个问题。
- 如果Basic Timer可以为DAC提供时钟，那么DAC就可以灵活的解调各种采样率的音频。

General-purpose timers(通用目的定时器)
-----------------------------------------

1. Feature
>>>>>>>>>>>>>>>>>>>
2. Block Diagram
>>>>>>>>>>>>>>>>>>>
3. Application
>>>>>>>>>>>>>>>>>>>
4. Register Design
>>>>>>>>>>>>>>>>>>>

Advanced-control timers(增强型定时器)
-----------------------------------------

1. Feature
>>>>>>>>>>>>>>>>>>>
2. Block Diagram
>>>>>>>>>>>>>>>>>>>
3. Application
>>>>>>>>>>>>>>>>>>>
4. Register Design
>>>>>>>>>>>>>>>>>>>
