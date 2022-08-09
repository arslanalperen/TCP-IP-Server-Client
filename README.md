<div id="top"></div>

<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->

[![Instagram][instagram-shield]][instagram-url]
[![LinkedIn][linkedin-shield]][linkedin-url]
[![Github][github-shield]][github-url]  

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <h3 align="center">TCP/IP Server & Client Example</h3>

  <p align="center">
    TCP/IP Server and Client Example without Memory Config on NUCLEO-F429ZI 
    <br />
    <a href="https://github.com/arslanalperen/TCP-IP-Server-Client"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/arslanalperen/TCP-IP-Server-Client">View Demo</a>
    ·
    <a href="https://github.com/arslanalperen/TCP-IP-Server-Client/issues">Report Bug</a>
    ·
    <a href="https://github.com/arslanalperen/TCP-IP-Server-Client/issues">Request Feature</a>
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#ethernet-setup">Ethernet Setup</a>
       <ul>
        <li><a href="#creating-a-project">Creating a Project</a></li>
        <li><a href="#clock-configuration">Clock Configuration</a></li>
        <li><a href="#timer-configuration">Timer Configuration</a></li>
        <li><a href="#ethernet-configuration">Ethernet Configuration</a></li>
        <li><a href="#lwip-configuration">LWIP Configuration</a></li>
      </ul>
    </li>
    <li>
      <a href="#software-setup">Software Setup</a>
      <ul>
        <li><a href="#tcp-server">TCP Server</a></li>
        <li><a href="#tcp-client">TCP Client</a></li>
      </ul>
    </li>
    <li>
      <a href="#ethernet-connection">Ethernet Connection</a>
      <ul>
        <li><a href="#tcp-server-connection">TCP Server Connection</a></li>
        <li><a href="#tcp-client-connection">TCP Client Connection</a></li>
      </ul>
    </li>
    <li><a href="#review">Review</a></li>
  </ol>
</details>

TCP/IP Server demo is installed on NUCLEO-F429ZI board. The purpose is to send dummy data to the Client at regular intervals over the TCP/IP Server device.

# Ethernet Setup

The mentioned configurations are also valid for TCP/IP Server and Client devices.

## Creating a Project

A new project should be created. `New` and `STM32 Project` options should be selected from the `File` section in the upper left corner. On the screen that appears, go to the `Board Selector` section and write `NUCLEO-F429ZI` in the `Commercial Part Number` section (Figure 1).

<div align="center"> <img src="Images/project-setup.png" alt="Figure 1"> </div>

A card is selected from the `Boards List` section, the project is named and created. Preferably the project should not be created with default settings. After the project is created, the ioc file of the project is opened. Again, preferably after the ioc file is opened, the `Clear Pinout` option should be selected from the `Pinout` section and the currently selected pinouts should be cleared.

## Clock Configuration

In order to set the clock of the card, the `RCC` screen is opened from the `System Core` section. `High Speed Clock (HSE)` option is selected as `Crystal/Ceramic Resonator`. The `Low Speed Clock (LSE)` option is kept off (Figure 2). While creating the demo, the clock settings were chosen in this way. The project can work with different clock settings.

<div align="center"> <img src="Images/clock-choice.png" alt="Figure 2"> </div>

Go to the `Clock Configuration` section. `HCLK` option is preferred to be operated at maximum. It is set to 180 MHz. After the `Clock Configuration` part is set without any problems, the clock settings are completed.

<div align="center"> <img src="Images/clock-config.png" alt="Figure 3"> </div>

## Timer Configuration

Move to `Timer` category and select `TIM1`. Preferably `TIM1` is used. If desired, a different timer can be used. What is important at this point is which APB line the selected timer is connected to. `TIM1` is connected to APB2 line. If a different timer is to be used, it can be found to which APB line the selected timer is connected by examining the STM32F429ZI Reference Manual. For this, go to the `2.3 Memory Map` section in the Reference Manual. The `Bus` option written at the point where TIM1 is seen in the `Peripheral` part of the table indicates which APB line it is connected to.

<div align="center"> <img src="Images/tim1-bus.png" alt="Figure 4"> </div>

After selecting the `TIM1` option, the `Clock Source` option is selected as `Internal Clock` from the `Mode` section.

<div align="center"> <img src="Images/tim1-mode.png" alt="Figure 5"> </div>

Timer is requested to run at 1 second intervals. `Prescaler` and `Counter Period` should be set for 1 second in `Parameter Settings` section. The time interval during which the timer will operate is divided by the corresponding APB timer clock value `Prescaler`. The result is also divided by the `Counter Period` value. If the result is 1, the timer duration will be 1 second. Therefore, `Prescaler` value is entered as `18000-1` and `Counter Period` value is entered as `10000-1`.

```sh
TIMupdateFreq(HZ) = Clock/((PSC-1) * (Period -1))
```

<div align="center"> <img src="Images/timer-param-conf.png" alt="Figure 6"> </div>

`TIM1 update interrupt and TIM10 global interrupt` option must be activated in `NVIC Settings` in `Configuration` section. After completing this setting, the timer configuration will be completed.

<div align="center"> <img src="Images/timer-nvic.png" alt="Figure 7"> </div>

## Ethernet Configuration

`ETH` under `Connectivity` must be activated. At this point, it should be known whether the card to be used supports MII or RMII. For example, the card used may support RMII in software but not in hardware. To check this information, the schematic file of the relevant card can be examined. The following steps can be followed to access the schematic file for the NUCLEO-F429ZI board. Search for `NUCLEO-F429ZI schematic` on the browser. Enter the search result named `NUCLEO-F429ZI – STMicroelectronics`. In the `CAD Resources` tab, under the `Schematic Pack` title, the file named `STM Nucleo (144 Pins) schematics` is can be downloaded.

<div align="center"> <img src="Images/sch-access.png" alt="Figure 8"> </div>

The downloaded zip contains both individual Altium files and a pdf file `MB1137` with the entire schematic. `LAN8742A-CZ-TR` chip and peripherals can be seen on page 5 of the pdf file.

<div align="center"> <img src="Images/lan8742-sch.png" alt="Figure 9"> </div>

In Figure 9, it can be seen that the connections of the chip is RMII. Therefore, the `Mode` option `RMII` can be selected on the `ETH` configuration page.

<div align="center"> <img src="Images/eth-param-conf.png" alt="Figure 10"> </div>

In the `Advanced Parameters` tab, make sure that the `PHY` setting is selected as `LAN8742A_PHY_ADDRESS`.

<div align="center"> <img src="Images/eth-phy-conf.png" alt="Figure 11"> </div>

In the `GPIO Settings` tab, it is seen to which pins the relevant signals are connected. CubeIde can sometimes misconfigure these pins by default. Pins need to be checked to make sure they are correct. Figure 9 shows how the pin configuration should be. If there is a pin that needs to be corrected, the signals that can be assigned to that pin can be seen by right-clicking on the corresponding pin of the chip in the `Pinout View` tab. In this way, the corresponding signals can be assigned to the corresponding pins in case of a fault with the pinout.

<div align="center"> <img src="Images/eth-pin-conf.png" alt="Figure 12"> </div>

In the `NVIC Settings` tab, the `Ethernet Global Interrupt` setting must be activated.

<div align="center"> <img src="Images/eth-nvic-conf.png" alt="Figure 13"> </div>

## LWIP Configuration

`LWIP` option should be activated under `Middleware` section. The `Enabled` box must be activated from the `Mode` section under the `LWIP` option. After this box is activated, the `Configuration` section will be active.

In the `Platform Settings` section, `Driver_PHY` option should be selected as the `LAN8742`. This option selects the ethernet chip on the board. If the option is not selected, ethernet will not work. At the same time, there is no error when the code is compiled. Therefore, the setting should not be overlooked.

<div align="center"> <img src="Images/lwip-conf.png" alt="Figure 14"> </div>

In the `Checksum` section, the `CHECKSUM_BY_HARDWARE` option should be `Enabled`.

<div align="center"> <img src="Images/lwip-checksum.png" alt="Figure 15"> </div>

`LWIP_DHCP` setting should be `Disabled` in `General Settings`. The fact that this setting is `Disabled` allows us to give a manual ip for the ethernet server. After `LWIP_DHCP` setting is `Disabled`, `IP_ADDRESS`, `NETMASK_ADDRESS` and `GATEWAY_ADDRESS` settings should be made. The settings here can be configured as desired. However, it should not be forgotten that the `IP_ADDRESS` value will be used later on ethernet connections.

<div align="center"> <img src="Images/ip-conf.png" alt="Figure 16"> </div>

`MEM_SIZE` can be left as default in `Key Options`. It is defined as `10*1024 Bytes` in the sample project.

<div align="center"> <img src="Images/heap-mem-conf.png" alt="Figure 17"> </div>

The remaining settings are left as default.

# Software Setup

## TCP Server

Sample project files include `tcpServerRAW.c` and `tcpServerRAW.h` files. These files contain basic functions to be used for TCP ethernet server. Functions are written by `ST` and files are edited by `ControllersTech`. The code has been finalized in the sample project by making some desired additions.

`tcpServerRAW.h` should be added in the `Private Includes` section of the main code.

<div align="center"> <img src="Images/include-header.png" alt="Figure 18"> </div>

`extern struct netif gnetif` code should be added to `Private Define` section. The struct, which contains the Ethernet-related configurations, is thus callable in the main code.

<div align="center"> <img src="Images/netif-call.png" alt="Figure 19"> </div>

TCP server and TIM1 must be init within the main function. It will be healthier to init the timer after the TCP server is inited.

<div align="center"> <img src="Images/init-timer-server.png" alt="Figure 20"> </div>

The `ethernetif_input(&gnetif)` and `sys_check_timeouts` functions must be called within the while loop inside the Main function. The `ethernetif_input(&gnetif)` function takes ethernet configurations with `gnetif` as input and listens on the ethernet port. The `sys_check_timeouts` function calls the timeout handler when the timeout expires.

<div align="center"> <img src="Images/while-one.png" alt="Figure 21"> </div>

The arrangements in the main code are completed at this point. To better understand the functions, the file `tcpServerRAW.c` can be examined. Includes and structures are defined in the beginning. The `tcp_server_states` enum is where the current state definitions of the ethernet protocol are kept. The `tcp_server_struct` is the structure where the current connection information is kept directly.

<div align="center"> <img src="Images/includes-and-structs.png" alt="Figure 22"> </div>

There are prototypes of the functions defined in the continuation of the code.

<div align="center"> <img src="Images/funcs-prototypes.png" alt="Figure 23"> </div>

The definition of the timer callback function is made in the `tcpServerRAW.c` file. The timer enters the callback function once per second. Data printing to the Ethernet port will be done within the callback function. A char array is defined at the beginning of the function. A string is assigned to the char array defined using sprintf, and the string length is returned to the variable named "len".

In cases where the counter value is not 0, pBuf is allocated in a size equal to the string length. The buffer containing the string is assigned to the region allocated with the `pbuf_take` function. Afterwards, the `tcp_server_send` function is called and the data is sent to the ethernet port, and then pBuf is released with the `pbuf_free` function.

<div align="center"> <img src="Images/tim-callback.png" alt="Figure 24"> </div>

The `tcp_server_init` function can be examined. At the beginning of the function, a pointer object named `tpcb` is created from the `tcp_pcb` structure. This object is initialized with the `tcp_new` function. It has `err_t` type error codes. It is defined with typedef. An object of type `ip_addr_t` named `myIPADDR` is defined. This object holds the IP address information. In the next line, the desired IP address values are defined to the object with the `IP_ADDR4` function. The IP address value given here must be the same as the IP value set in the Ethernet configuration section. A connection has been established with the port and IP information provided with the `tcp_bind` function. The returned error code is assigned to the `err` variable. If the returned error code is `ERR_OK`, TCP is put into listening for the pcb and LwIP is initialized by calling the `tcp_accept` function. If the error code is not `ERR_OK`, the allocated memory has been released.

<div align="center"> <img src="Images/tcp-server-init-func.png" alt="Figure 25"> </div>

The `tcp_server_accept` function can be examined. In `err_t` type, `ret_err` object has error status and `es` pointer has ethernet configurations in `tcp_server_struct` type are defined. The `LWIP_UNUSED_ARG` function is used so that unused objects do not cause problems during compilation. The `tcp_setprio` function is called to init Ethernet configurations and assign priorities. The `mem_malloc` function is used to allocate as much space as `tcp_server_struct` in RAM.

<div align="center"> <img src="Images/tcp-server-accept-func.png" alt="Figure 25"> </div>

If the `es` object is not NULL and the required space can be allocated in RAM, the code goes inside the if block. Otherwise it will go inside the else block. In case of entering the if block, necessary assignments are made to the `es` object and the functions tcp_arg, tcp_recv, tcp_err and tcp_poll are called respectively. The purposes of the functions are explained in the relevant parts of the report. At the end of the block, the `ret_err` variable is assigned to indicate that no problems were encountered. In case of entering the else block, the tcp connection is terminated by calling the `tcp_server_connection_close` function. Finally, `ERR_MEM` is assigned to the `ret_err` variable. This means that there is a problem in memory allocation and the connection is terminated.

---Sayfa 15---

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->

[instagram-shield]: https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white
[github-shield]: https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white
[linkedin-shield]: https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white

[instagram-url]: https://www.instagram.com/arslanalperen55/
[github-url]: https://github.com/arslanalperen
[linkedin-url]: https://www.linkedin.com/in/arslanalperen/

[fifo-diagram]: Images/fifo-diagram.png
