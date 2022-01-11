---
title: "Interrupt with User Push Button (PB) on STM32F3-Discovery"
categories:
  - Embedded
tags:
  - vscode
  - embedded
  - interrupt 
  - stm32
  - stm32f3 discovery
last_modified_at: 2022-01-02
toc: true
toc_label: "Menu"
toc_icon: "columns"
--- 

# PB on STM32F3-Discovery
The push button on the STM32F3-Discovery is connected to the GPIO-Port `GPIOA` and GPIO-Pin `GPIO_PIN_0`, or for short pin `A0`. In the setting of this pin in CubeMX, i cannot set the GPIO_Mode option to external interrupt. So i took a look to the example code and demo code of this board. 

![cubemx screenshot](/assets/pinA0.png)

# Initialize GPIO
## My code with HAL library

`gpio.c -> MX_GPIO_Init()`
```c
  /*Configure GPIO pin : PtPin */ /*User Button pin*/
  GPIO_InitStruct.Pin = B1_Pin;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  HAL_GPIO_Init(B1_GPIO_Port, &GPIO_InitStruct);
```

`main.c`
```c
static void EXTI0_Config(void)
{
  /* Initialize EXTI_LINE */
  EXTI_ConfigTypeDef pb_exti;
  pb_exti.Line = EXTI_LINE_0;
  pb_exti.Mode = EXTI_MODE_INTERRUPT;
  pb_exti.Trigger = EXTI_TRIGGER_RISING;
  pb_exti.GPIOSel = EXTI_GPIOA;
  HAL_EXTI_SetConfigLine(&hexti0, &pb_exti);
  
  /* Enable and set EXTI0 Interrupt to the lowest priority */
  HAL_NVIC_SetPriority(EXTI0_IRQn, 15, 15);
  HAL_NVIC_EnableIRQ(EXTI0_IRQn);
}
```

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
  if (GPIO_Pin == B1_Pin)
  {
      printf("Push Button (Blue) pushed. External Interupt activate\n");
      UserPressButton++;
      if (UserPressButton > 0x02)
      {
        UserPressButton = 0;
      }
      Set_All_LEDs(GPIO_PIN_RESET);
      HAL_GPIO_TogglePin(LD3_GPIO_Port, LD3_Pin);
      HAL_GPIO_TogglePin(LD10_GPIO_Port, LD10_Pin);
    

    // change mode in state machine
  }
}
```


## Demo Code
`stm32f3-discovery.c`
```c
void STM_EVAL_PBInit(Button_TypeDef Button, ButtonMode_TypeDef Button_Mode)
{
  GPIO_InitTypeDef GPIO_InitStructure;
  EXTI_InitTypeDef EXTI_InitStructure;
  NVIC_InitTypeDef NVIC_InitStructure;

  /* Enable the BUTTON Clock */
  RCC_AHBPeriphClockCmd(BUTTON_CLK[Button], ENABLE);
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_SYSCFG, ENABLE);

  /* Configure Button pin as input */
  GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN;
  GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_NOPULL;
  GPIO_InitStructure.GPIO_Pin = BUTTON_PIN[Button];
  GPIO_Init(BUTTON_PORT[Button], &GPIO_InitStructure);

  if (Button_Mode == BUTTON_MODE_EXTI)
  {
    /* Connect Button EXTI Line to Button GPIO Pin */
    SYSCFG_EXTILineConfig(BUTTON_PORT_SOURCE[Button], BUTTON_PIN_SOURCE[Button]);

    /* Configure Button EXTI line */
    EXTI_InitStructure.EXTI_Line = BUTTON_EXTI_LINE[Button];
    EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt;
    EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Rising;  
    EXTI_InitStructure.EXTI_LineCmd = ENABLE;
    EXTI_Init(&EXTI_InitStructure);

    /* Enable and set Button EXTI Interrupt to the lowest priority */
    NVIC_InitStructure.NVIC_IRQChannel = BUTTON_IRQn[Button];
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0x0F;
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 0x0F;
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;

    NVIC_Init(&NVIC_InitStructure); 
  }
}
```