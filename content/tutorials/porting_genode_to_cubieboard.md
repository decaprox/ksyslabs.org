--- 
title: Портирование Genode OS Framework на новую аппаратную платформу
---
### Портирование Genode OS Framework на новую аппаратную платформу

Данная статья рассказывает о [Genode OS Framework](http://genode.org/) и процессе портирования его на новую аппаратную платформу на базе процессора архитектуры ARM. В качестве ядра для Genode используется [Fiasco.OC](http://os.inf.tu-dresden.de/fiasco/).

![text](/upload/cubieboard.png)

И так, что такое Genode OS Framework? Это фреймворк для построения операционной системы на базе микроядра. Genode предоставляет единый API, позволяющий использовать различные микроядра для построения ОС. На данный момент поддерживаются следующие микроядра: Codezero, Fiasco, Fiasco.OC, Nova, OKL4, Pistachio. Также поддерживается работа на ядре Linux. Кроме этого, возможен запуск без использования сторонних ядер (base-hw) на некоторых платформах на базе архитектуры ARM.<br />
Эти ядра поддерживают разные архитектуры процессоров и могут использовать некоторое аппаратные возможности. Например, ядро Fiasco.OC может работать на большом количестве архитектур, таких как: x86, amd64, ARM (доступны и другие архитектуры, но они не поддерживаются в Genode на данный момент), а ядро Nova является микрогипервизором для архитектуры x86 и позволяет использовать аппаратную виртуализацию. Благодаря использованию фреймворка Genode мы можем без изменений исходного кода компилировать приложение для использования на любом из поддерживаемых ядер.
Более подробную информацию о Genode можно получить из документации, размещенной на сайте проекта, а также из материалов, прочитаных Norman Feske для “[Летней школы системного программирования Ksys labs](http://ksyslabs.ru/index.php?nn=36)” 

В настоящее время Genode+FIasco.OC для архитектуры ARM поддерживает следующие платформы: Realview PBXA9, Versatile Express A9X4, Pandaboard (TI OMAP4), Freescale iMX53, Arndale (Samsung Exynos 5).<br />
В качестве платформы для портирования был использован мини-ПК [Cubieboard](http://cubieboard.org/) на базе SoC [Allwinner A10](http://linux-sunxi.org/A10).
Эта платформа интересна по ряду причин:<br />
- не слишком устаревшая архитектура Cortex-A8;<br />
- наличие исходного кода загрузчика U-Boot, ядра Linux и ”утекшей” в сеть документации в виде руководства пользователя;<br />
- большой набор переферии (SATA, HDMI, и др.);<br />
- наличие большого количества недорогих “hackable” устройств на этом чипе (Cubieboard, Mele A1000/A2000, и другие).

Рассмотрим этот SoC более подробно.

![text](/upload/a10_diagram.png)<br />
Рисунок 1 - Функциональная диаграмма


CPU: ARM Cortex-A8 с частотой до 1Ghz с поддержкой NEON, VFPv3, Trustzone<br />
GPU: Mali 400 MP с поддержкой Open GL ES 2.0<br />
VPU: CedarX с поддержкой FullHD видео.<br />
Переферия: 4xUSB Host, USB OTG, 4xSD/MMC, 8xUART, 4xSPI, 3xI2C, GPIO, SATA, HDMI, LCD-интерфейс и другие<br />

Ядро Fiasco.OC поддерживает архитектуру процессоров Cortex-A8. Это значит, что для портирования его на новую платформу нужно только добавить поддержку платформы, так называемый Board Support PAckage ([BSP] (http://ru.wikipedia.org/wiki/Board_Support_Package)). Исходный код BSP расположен в директории kernel/fiasco/src/kern/arm/bsp.<br />
В состав BSP для архитектуры ARM в Fiasco.OC  входит: <br />
- драйвер контроллера прерываний;<br />
- драйвер таймера;<br />
- драйвер UART;<br />
- реализация сброса.<br />
Кроме этого, в состав BSP входит конфигурация распределения памяти.<br />
Память в A10, согласно руководству пользователя, распределяется следующим образом:

![text](/upload/a10_mem.png)<br />

Для того, чтобы ОС могла использовать память, она должна знать физические адреса и размер необходимых регионов памяти. Эти параметры задаются в классе Mem_layout, файл mem_layout-arm-sun4i.cpp:

~~~
EXTENSION class Mem_layout
{
public:
  enum Virt_layout_sun4i : Address {
    Timer_map_base       = Devices1_map_base + 0x020C00,
    Intc_map_base        = Devices1_map_base + 0x020400,
    Uart_base            = Devices1_map_base + 0x028000,

    Watchdog_map_base    = Timer_map_base    + 0x90,
  };

  enum Phys_layout_sun4i : Address {
    Devices1_phys_base   = 0x01c00000,

    Sdram_phys_base      = 0x40000000,
    Flush_area_phys_base = 0xe0000000,
  };
};
~~~
Контроллер прерываний необходим для обработки событий от различных источников прерываний. Драйвер контроллера прерываний должен реализовать такие операции как: конфигурирование контроллера, маскирование прерываний, функцию обработки. Код драйвера в файле pic-arm-sun4i.cpp.

Таймер необходим для генерации событий,таких как: конец тайм-слота или таймаут IPC.
В A10 имеется 6 таймеров. В качестве таймера для переодических прерываний используется таймер Timer0. Кроме таймеров общего назначения в состав SoC также входят Watchdog и RTC с будильником.<br />
Для использования таймера в режиме системного он должен генерировать прерывание каждую 1 мс. Инициализация таймера и необходимые функции реализованы в файле timer-arm-sun4i.cpp.

Драйвер UART используется ядром для вывода отладочных сообщений и доступа к отладчику ядра JDB. В Fiasco.OC инициализация модуля UART отсутсвует, считается, что его уже настроил загрузчик, в нашем случае - U-Boot. Код драйвера UART находится в файлах uart-arm-sun4i.cpp и kernel/fiasco/src/lib/uart/uart_sun4i.cc.

Для выполнения полного сброса процессора применяется Watchdog таймера, назначением которого является сброс системы при зацикливании кода. Реализация находится в файле reset-arm-sun4i.cpp.

После выполнения этих действий мы получаем BSP для выбраного процессора.

Рассмотрим процесс загрузки Genode+Fiasco.OC на архитектуре ARM:

1. При старте ROM-boot ищет загрузчик на SD-карте или загружается с NAND при отсутсвии SD-карты.
2. U-Boot, выполняя стартовые скрипты, загружает образ Genode в виде ELF или u-boot-image.
3. Загруженный модуль содержит в себе ядро Fiasco.OC и все остальные исполняемые файлы, такие как: sigma0, root task (в Genode это модуль core) и пользовательские программы.
4. Первым стартует bootstrap, который выполняет часть операций неоходимых для старта ядра, такие как:
— сканирование доступной памяти (выполняется не для всех архитектур, на ARM значение доступной памяти задается в конфигурации на платформу);
— перемещение модулей в памяти (sigma0 и root task должны быть расположены в определенных регионах, чтобы ядро при старте могло их запустить);
— и непосредственно запуск ядра.
7. Ядро выполняет необходимую инициализацию системы, запускает sigma0 и root task.
8. Начинает выполняться модуль core (являющийся root task), который инициализирует сервисы Genode и позволяет запускать необходимые нам приложения, используя конфигурационный файл.

Соответсвенно для запуска bootstrap на Cubieboard он должен знать об этой платформе, поэтому добавляем нужную конфигурацию l4/mk/platforms/cubieboard.conf и l4/pkg/bootstrap/server/src/platform/sun4i.cc. Кроме этого необходимо реализовать драйвер UART для вывода информации о загрузке в консоль.

Заключительный этап портирования — добавление необходимых файлов конфигурации в сборочную систему Genode. Изменения можно посмотреть в соответвующем коммите в репозитории. Хорошее описание системы сборки было рассмотрено в лекции "Genode OS Framework Programming Environment".

Исходные коды находятся на Github [https://github.com/Ksys-labs/genode](https://github.com/Ksys-labs/genode) в ветке **tutorials**. Модифицированная версия Fiasco.OC в репозитории [https://github.com/Ksys-labs/foc](https://github.com/Ksys-labs/foc) в ветке **r47-sun4i**, эти исходные коды уже содержат необходимые патчи и скачиваются с помощью системы сборки Genode.

Для сборки необходимо клонировать исходный код:

~~~
git clone git://github.com/Ksys-labs/genode.git
git checkout tutorials
cd genode
~~~

Собрать тулчейн для ARM:

~~~
./tools/tool_chain arm
~~~

И скачать ядро Fiasco.OC:

~~~
make prepare -C base-foc
~~~

Теперь все готово для запуска.

1. Создаем директорию для сборки, используя команду:

~~~
./tools/create_builddir foc_cubieboard BUILD_DIR=_build.foc_cubieboard
cd _build.foc_cubieboard
~~~

2. Добавляем репозиторий hello_tutorial для сбоки простейшего сценария, которому не требуются отсутсвующие пока драйверы.

~~~
echo 'REPOSITORIES += $(GENODE_DIR)/hello_tutorial' >> etc/build.conf
~~~

3. Включаем генерацию файла u-boot-image

~~~
echo 'SPECS += uboot' >> etc/spec.conf
~~~

4. Cобираем образ

~~~
make run/hello
~~~

После непродолжительной сборки получаем образ в виде: ELF (hello.elf) и u-boot-image (uImage) в директории var/run/hello.
После запуска на устройстве, в подключенной консоле, можно увидеть процесс загрузки и выполняющиеся приложения из hello tutorial:

~~~
L4 Bootstrapper
  Build: #4 Чт. апр. 18 22:48:37 MSK 2013, 4.7.2
  Scanning up to 1024 MB RAM
  Memory size is 1024MB (40000000 - 80000000)
  RAM: 0000000040000000 - 000000007fffffff: 1048576kB
  Total RAM: 1024MB
  mod07: 4117e000-411b8e3c: genode/timer
  mod06: 41148000-4117ddc0: genode/hello_server
  mod05: 4111c000-41147c28: genode/hello_client
  mod04: 410d6000-4111b738: genode/init
  mod03: 410d5000-410d51a4: genode/config
  mod02: 4106e000-410d431c: genode/core
  mod01: 41064000-4106d374: sigma0
  mod00: 41015000-41063d20: /home/vanner/projects/genode-iloskutov/_build.foc_cubieboard/kernel/fiasco.oc/fiasco
  Moving up to 8 modules behind 41100000
  moving module 00 { 41015000-41063d1f } -> { 412a4000-412f2d1f } [322848]
  moving module 01 { 41064000-4106d373 } -> { 412f3000-412fc373 } [37748]
  moving module 02 { 4106e000-410d431b } -> { 412fd000-4136331b } [418588]
  moving module 03 { 410d5000-410d51a3 } -> { 411b9000-411b91a3 } [420]
  moving module 04 { 410d6000-4111b737 } -> { 411ba000-411ff737 } [284472]
  moving module 05 { 4111c000-41147c27 } -> { 41100000-4112bc27 } [179240]
  moving module 06 { 41148000-4117ddbf } -> { 4112c000-41161dbf } [220608]
  moving module 07 { 4117e000-411b8e3b } -> { 41162000-4119ce3b } [241212]
  moving module 03 { 411b9000-411b91a3 } -> { 4119d000-4119d1a3 } [420]
  moving module 04 { 411ba000-411ff737 } -> { 4119e000-411e3737 } [284472]
  Scanning /home/vanner/projects/genode-iloskutov/_build.foc_cubieboard/kernel/fiasco.oc/fiasco -serial_esc 
  Scanning sigma0
  Scanning genode/core
  Relocated mbi to [0x4100e000-0x4100e19c]
  Loading -iloskutov/_build.foc_cubieboard/kernel/fiasco.oc/fiasco
  Loading sigma0
  Loading genode/core
  find kernel info page...
  found kernel info page at 0x40002000
Regions of list 'regions'
    [ 40001000,  400019ff] {      a00} Kern   -iloskutov/_build.foc_cubieboard/kernel/fiasco.oc/fiasco
    [ 40002000,  40060fff] {    5f000} Kern   -iloskutov/_build.foc_cubieboard/kernel/fiasco.oc/fiasco
    [ 40090000,  4009673b] {     673c} Sigma0 sigma0
    [ 40098000,  4009e17b] {     617c} Sigma0 sigma0
    [ 40100000,  4024743f] {   147440} Root   genode/core
    [ 41000000,  410143f3] {    143f4} Boot   bootstrap
    [ 4100e000,  4100e299] {      29a} Root   Multiboot info
    [ 41100000,  411e3737] {    e3738} Root   Module
  API Version: (87) experimental
  Sigma0 config    ip:40090100 sp:41013d24
  Roottask config  ip:4014af84 sp:00000000
  Starting kernel -iloskutov/_build.foc_cubieboard/kernel/fiasco.oc/fiasco at 40001198
Hello from Startup::stage2
Boot_alloc: size=0x180
Boot_alloc: allocated extra memory block @0xf13e1000 (size=400)
Boot_alloc: @ 0xf13e1000
Boot_alloc: remaining free block @ 0xf13e1180 (size=280)
Cache config: ON
ID_PFR[01]:  00001131 00000011 ID_[DA]FR0: 00000400 00000000
ID_MMFR[04]: 01100003 20000000 01202000 00000211
FPU0: Arch: VFPv3(3), Part: VFPv3(30), r: 3, v: c, i: 41, t: hard, p: dbl/sngl
Startup::stage2 finished
SERIAL ESC: allocated IRQ 1 for serial uart
Not using serial hack in slow timer handler.
Welcome to Fiasco.OC (arm)!
L4/Fiasco.OC arm microkernel (C) 1998-2013 TU Dresden
Rev: 8991035 compiled with gcc 4.7.2 for sun4i    []
Build: #3 Чт. апр. 18 22:48:33 MSK 2013

Calibrating timer loop... done.
SIGMA0: Hello!
  KIP @ 40002000
  allocated 4KB for maintenance structures
SIGMA0: Dump of all resource maps
RAM:------------------------
[0:40000000;40000fff]
[0:40061000;4008ffff]
[0:40097000;40097fff]
[0:4009f000;400fffff]
[4:40100000;40247fff]
[0:40248000;4100dfff]
[4:4100e000;4100efff]
[0:4100f000;410fffff]
[4:41100000;411e3fff]
[0:411e4000;7effffff]
IOMEM:----------------------
[0:0;3fffffff]
[0:80000000;ffffffff]

KIP @ 40002000
    magic: 4be6344c
  version: 87014444
         sigma0  esp: 41013d24  eip: 40090100
         sigma1  esp: 00000000  eip: 00000000
           root  esp: 00000000  eip: 4014af84
MBI @ 4100e000
 mod[3] [4119d000,4119d1a4) config
 mod[4] [4119e000,411e3738) init
 mod[5] [41100000,4112bc28) hello_client
 mod[6] [4112c000,41161dc0) hello_server
 mod[7] [41162000,4119ce3c) timer
:ram_alloc: Allocator 40230784 dump:
 Block: [50000000,5000001c) size=0000001c avail=00000000 max_avail=00000000
 Block: [5000001c,50000038) size=0000001c avail=00000000 max_avail=00000000
 Block: [50000038,50000054) size=0000001c avail=00000000 max_avail=00000000
 Block: [50000054,50000070) size=0000001c avail=00000000 max_avail=2effff58
 Block: [50000070,5000008c) size=0000001c avail=00000000 max_avail=00000000
 Block: [5000008c,500000a8) size=0000001c avail=00000000 max_avail=2effff58
 Block: [500000a8,7f000000) size=2effff58 avail=2effff58 max_avail=2effff58
 => mem_size=788529152 (752 MB) / mem_avail=788528984 (751 MB)
:region_alloc: Allocator 402318f4 dump:
 Block: [00001000,40000000) size=3ffff000 avail=3ffff000 max_avail=3ffff000
 Block: [7f000000,bfff0000) size=40ff0000 avail=40ff0000 max_avail=40ff0000
 Block: [bfff1000,c0000000) size=0000f000 avail=0000f000 max_avail=0000f000
 => mem_size=2164252672 (2063 MB) / mem_avail=2164252672 (2063 MB)
:io_mem: Allocator 40230be0 dump:
 Block: [00000000,40000000) size=40000000 avail=40000000 max_avail=40000000
 Block: [40001000,40002000) size=00001000 avail=00001000 max_avail=40000000
 Block: [40003000,40061000) size=0005e000 avail=0005e000 max_avail=0005e000
 Block: [40090000,40097000) size=00007000 avail=00007000 max_avail=0005e000
 Block: [40098000,4009f000) size=00007000 avail=00007000 max_avail=80ffffff
 Block: [7f000000,ffffffff) size=80ffffff avail=80ffffff max_avail=80ffffff
 => mem_size=3238449151 (3088 MB) / mem_avail=3238449151 (3088 MB)
:io_port: Allocator 4023103c dump:
:irq: Allocator 40231498 dump:
 Block: [00000000,00000100) size=00000100 avail=00000100 max_avail=00000100
 => mem_size=256 (0 MB) / mem_avail=256 (0 MB)
:rom_fs: Rom_fs 402321a8 dump:
 Rom: [4119e000,411e3738) init
 Rom: [41100000,4112bc28) hello_client
 Rom: [4119d000,4119d1a4) config
 Rom: [4112c000,41161dc0) hello_server
 Rom: [40002000,40003000) l4v2_kip
 Rom: [40002000,40003000) kip
 Rom: [41162000,4119ce3c) timer
:core ranges: Allocator 40233f08 dump:
 Block: [40100000,40248000) size=00148000 avail=00148000 max_avail=00148000
 Block: [41100000,411e4000) size=000e4000 avail=000e4000 max_avail=2f000000
 Block: [50000000,7f000000) size=2f000000 avail=2f000000 max_avail=2f000000
 => mem_size=790806528 (754 MB) / mem_avail=790806528 (754 MB)
int main(): --- create local services ---
int main(): --- start init ---
int main(): transferred 751 MB to init
int main(): --- init created, waiting for exit condition ---
[init] Could not open file "ld.lib.so"
[init -> hello_server] Hello::Root_component::Root_component(Genode::Rpc_entrypoint*, Genode::Allocator*): Creating root component.
[init -> hello_server] virtual Hello::Session_component* Hello::Root_component::_create_session(const char*): creating hello session.
[init -> hello_client] virtual void Hello::Session_client::say_hello(): Saying Hello.
[init -> hello_server] virtual void Hello::Session_component::say_hello(): I am here... Hello.
[init -> hello_client] int main(): Added 2 + 5 = 7
[init -> hello_client] virtual void Hello::Session_client::say_hello(): Saying Hello.
[init -> hello_server] virtual void Hello::Session_component::say_hello(): I am here... Hello.
[init -> hello_client] int main(): Added 2 + 5 = 7
...
~~~
