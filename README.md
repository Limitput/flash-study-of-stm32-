# flash-study-of-stm32，此为学习stm32 的 bootloader的前置学习，先掌握flash的应用（读写擦）
## 首先试一下内部flash
先查找最大存储地址，例如0x0800FFE0
        #define FLASH_SAVE_ADDRESS 0x0800FF00
        static FLASH_EraseInitTypeDef EraseInitStruct = {
            .TypeErase /*擦除类型*/= FLASH_TYPERESS_PAGES,//页擦除
            .PageAddress = FLASH_SAVE_ADDRESS,//擦除地址
            .NbPages = 1// 擦除页数
        }

接下来是具体的写入规范
    1、先清除要写入东西的部分，不能够直接覆盖
    2、擦除之前要先解锁flash
    3、擦除前要关闭中断，不然会出意外
        对于第3点，gpt是这样形容的
        在擦除Flash之前要关闭中断，这是为了保障数据完整性和系统稳定性。具体原因如下：

        防止代码重入和冲突
        Flash擦除操作是一种比较复杂且有严格时间要求的过程。在擦除期间，如果中断发生（比如某个外部事件或定时器），可能会导致CPU去执行其他代码，尤其是如果中断服务程序也涉及到对Flash的读写，会和正在进行的擦除操作产生冲突，可能导致擦除出错，数据损坏，甚至程序跑飞（死机）。

        保证Flash操作的原子性
        擦除Flash时，Flash控制器需要占用总线，并且不会响应其它的读写请求。如果发生中断，某些中断服务程序可能会尝试访问Flash，会引起总线争用或Flash控制器的异常工作。关闭中断可以保证整个Flash擦除过程不会被打断，确保原子性和可靠性。

        安全性考虑
        有些嵌入式系统或MCU在Flash擦除过程中会暂时无法执行从Flash读出来的代码（这时程序一般运行在RAM）。如果发生中断，而中断服务程序在Flash中，反而会因为代码无法正常取指，直接导致系统异常。

    HAL_FLASH_Unlock();//解锁flash

    uint32_t PageError = 0;
    __disable_irq();//关闭中断

    if(HAL_FLASHEx_Erase(&EraseInitStruct,&PageError) == HAL_OK)
    {
        printf("erase ok");
    }

    __enable_irq();//打开中断

    uint32_t data = 0x55556666;
    uint32_t addr = FLASH_SAVE_ADDR;
    HAL_FLASH_Program(FLASH_TYPEPROGRAM_WORD/*此应该是写入的数据类型*/,addr,data);//值得注意的是，单片机通常小端存储，即按序存入后的数据为0x66665555
    
    HAL_FLASH_Lock();//flash上锁
# 待测试的一点是再读出来后是66665555还是55556666




## 接下来是外部flash
