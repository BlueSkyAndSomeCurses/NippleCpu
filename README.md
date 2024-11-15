# CpuProject

**1. Архітектура**
* Інструкції зберігаються в ROM і займають 12 біт в кожній комірці: 4 біти - код операції, решта адреси даних. Всього комірок в ROM 256. Також він має 4 бітні шини даних та адреси.
* Процесор має всього 8 4-бітних регістрів загального використання та 1 регістр для зберігання прапорців. Також для зберігання даних використовується 4*16 RAM.
* Для виконання арифметичних та логічних операцій використовується 4-бітний ALU.
* Для послідовного виконання інструкцій використовується 8 бітний лічильник (2 4-бітні із серії 7400), операції умовних переходів виконуються, записуючи адресу потрібної іструкції ROM в лічильник.
Далі наведене схема процесора в Logisim:
![123](https://user-images.githubusercontent.com/47101236/71410781-50515200-264f-11ea-9a8f-272eefc34685.png)
**2. Логіка**
Процесор здійснює ту чи іншу операцію за допомогою контрольних бітів:
* Digit
* Jump (для операцій безумовного переходу)
* Alu (для операцій, які використовують Alu)
* AluLogic (для логічних операцій Alu)
* AluCarry (ставить cin Alu на одиничку, в даному процесорі використовується тільки для операції SUB)
* FWright (біт запису в регістр флагів прапорець, в даному процесорі використовується тільки для операції SUB)
* Adress (для операцій, які здійснюються з використанням даних з RAM)
* Write (прапорець запису в регістр чи RAM)
Ці біти визначаються за допомогою декодера коду операції:

![123](https://user-images.githubusercontent.com/47101236/71411098-a1157a80-2650-11ea-81af-d82ccdad5dee.png)

Далі табличка позначень різних операцій (r1, r2 - any register, immed - числове значення):


Operation              | Mnemonic                                                                                                                                                                                                                                                                                                                                       | Description
------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------
**0000**            | mvr r1, r2 | скопіювати вміст r2 в r1
**0001**            | mvd r1, immed | записати в регістр r1 immediate значення
**0010**            | mva r1, r2 | записати в комірку пам'яті RAM значення, що знаходиться в r2 (адреса(порядковий номер) комірки знаходиться в r1)
**0011**            | mvf r1, r2 | записати в регістр r1 значення, яке знаходиться в комірці RAM за адресою(порядковий номер), що знаходиться в регістрі r2
**0110**            | sub r1, r2 | записати в r1 різницю r1 - r2, також це використовується як порівняння r2 та r1 (записуються прапорці в регістр флагів)
**1001**            | add r1, r2 | записати в r1 суму r1 + r2
**1010**            | nor r1, r2 | записати в r1 - r1 nor r2
**1011**            | iop r1, r1 | r1 - регістр з адресою пристрою, r2 - регістр з даними
**1100**            | jeq immed | перейти на комірку пам'яті за адресою immed в ROM, якщо прапорець рівності дорівнює 1, immed - 8 бітне число
**1101**            | jls immed | перейти на комірку пам'яті за адресою immed в ROM, якщо прапорець знаку дорівнює 1, immed - 8 бітне число
**1110**            | jle immed | перейти на комірку пам'яті за адресою immed в ROM, якщо прапорець знаку або рівності дорівнює 1, immed - 8 бітне число
**1111**            | jne immed | перейти на комірку пам'яті за адресою immed в ROM, якщо прапорець рівності дорівнює 0, immed - 8 бітне число |



**Ремарки:**
* Кількість операцій може бути 16, тому, можливо, будуть реалізовані ще операції
* В переліку операцій немає безумовного переходу, адже це можна зробити таким чином sub r1, r1; je immed
* Процесор (поки що) має тільки 1 логічну операцію nor, адже вона робить систему повною, тобто будь-яку іншу операцію можна зробити використовуючи тільки nor
* Регістр прапорець 2-бітний, він зберігає біт знаку та біт рівності, тобто якщо ми зробимо sub r1, r2 і біт знаку буде дорівнювати 1, це означатимо що r1 < r2
* При зміні значення тактового генератора з 0 до 1 виконується операція, але нічого не записується (тобто біт Write = 0), а записується при переході з 1 на 0
* Джампи записуються в ROM наступним чином j<type> HEX

**3.PC**


![123](https://user-images.githubusercontent.com/47101236/71412603-1be19400-2657-11ea-98df-1887b4223035.png)

Схема, як реалізований перехід о однієї іструкції на іншу. Всі іструкції збережені в ROM. Clock - тактовий генератор. При зміні його значення з 0 на 1 виконується перехід на наступну іструкцію. A - адреса регістра, де потрібно буде зберегти значення, B - адреса регістра звідки брати дані або просто immed число. Command - код операції. Adress - це адреса комірки пам'яті в ROM, використовується для умовних переходів. Якщо перші два біти коду операції - це 1 та котрольний біт DoJump = 1 (тобто виконується умова переходу), то значення AB (a1a2a3a4b1b2b3b4 - 8-bit) записується в лічильник.


**4. Registers**

![123](https://user-images.githubusercontent.com/47101236/71412894-539d0b80-2658-11ea-818d-aab0a1c8a005.png)

Схема регістрів дуже проста: to - адреса регістру, в який потрібно записати дані data, w - контрольний біт, який визначає чи записувати чи не записувати, A - адреса регістру A, B - адреса регістру B, за допомогою цих адрес визначення з яких регістрів давати дані на вихід.

**5. Sorting**

Сортування даних за допомогою InsertionSort

![ezgif com-video-to-gif](https://user-images.githubusercontent.com/47101236/71413186-b5aa4080-2659-11ea-9951-ba7b5f5f1796.gif)

**6. Мікросхеми**

Для побудови цього процесору використовувались такі мікросхеми:

* RAM - 74F189 64-Bit Random Access Memory with 3-STATE Outputs
* ROM x2- AT28C256 256K (32K x 8) Paged CMOS E^2PROM
* ALU - SN74LS181 ARITHMETIC LOGIC UNITS/FUNCTION GENERATORS
* Counter - SN74HC193 4-bit Synchronous Up/Down Counter (Dual clock with Clear)
* D-Trigger х9 - SN74LS173A 4-BIT D-TYPE REGISTERSWITH 3-STATE OUTPUTS
* Багато логічних мікросхем та мультиплексорів із серії 7400

**7. Файли**

*  [CPU.circ](https://github.com/shakhovm/CpuProject/blob/master/CPU.circ) - головна схема процесора
*  [7400-lib.circ](https://github.com/shakhovm/CpuProject/blob/master/7400-lib.circ), [logi7400dip.circ](https://github.com/shakhovm/CpuProject/blob/master/logi7400dip.circ) - бібліотека Logisim елементів серії 7400
* [SortAlg](https://github.com/shakhovm/CpuProject/blob/master/SortAlg) - інструкції для алгоритму сортування (цей файл можна записати в ROM на схемі)
* [Sorting.txt](https://github.com/shakhovm/CpuProject/blob/master/Sorting.txt) - мнемонічний вигляд алгоритму сортування

**План роботи**

1. Проєктування логічної схеми в програмі “Logisim”.

* Розширення адресного простору RAM.
* Створення стекової пам’яті та відповідних інструкцій процесора для роботи із ним.
* Реалізація переривань процесора.
* Додати периферійні пристрої для візуалізації поточних даних роботи процесора.

2. Проєктування схеми у програмі Circuit Maker для створення друкованої плати в майбутньому.
* Перенесення логічної схеми у програму Circuit Maker.
* Розведення схеми.
3. Програмування логічної схеми процесора з використанням технології FPGA.
4. Синтез прототипу з використанням друкованої плати.

