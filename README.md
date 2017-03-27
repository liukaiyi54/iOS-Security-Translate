# 介绍
苹果以安全为核心设计了iOS平台。当我们着手去打造一个尽可能最好的移动平台时，我们吸取了数十年的经验，建立一个了全新的架构。我们考虑了桌面环境的安全风险，并且在设计iOS的时候，建立了新的安全方法。我们开发并整合了创新的特性，这些特性更加牢固了移动安全性，并且会默认保护整个系统。因此，iOS是移动设备安全性的重大飞跃。

每一个iOS设备都将软件，硬件和服务集成在一起，以实现最大的安全性和透明的用户体验。iOS不仅可以保护设备及其数据，还可以保护整个生态系统，包括用户在本地，网络上，已经在一些关键的互联网服务中所做的一切。

iOS和iOS设备提供高级的安全特性，但是同时它们也易于使用。许多这些特性都是默认开启的，因此IT部门不需要执行大量的配置工作。并且像设备加密等关键安全功能是不可配置的，所以用户不会错误地禁掉它们。其他特性，像Touch ID，通过使设备使用更简单，更直观地增强了用户体验。

本文档提供了有关如何在iOS平台中实现安全技术和功能的详细信息。它还将帮助开发团队将iOS平台安全技术与功能结合自己的政策和程序来满足其特定的安全需求。

本文档分为以下主题：

* **系统安全**：集成的安全的软件和硬件，即iPhone，iPad和iPod touch的平台。
* **加密和数据保护：**在设备发生丢失或被盗，或者未经授权的人员尝试进行使用或修改时，这个架构体系和设计可以保护用户数据。
* **App安全：**系统可以安全的运行app并且不会破坏平台的完整性。
* **网络安全：**为传输数据提供安全认证和加密的行业标准网络协议。
* **Apple Pay：**苹果的安全支付方法。
* **网络服务：**苹果基于网络的用于消息传递，同步和备份的基础设施。
* **设备控制：**一些允许管理iOS设备，防止未授权使用的方法，如果设备丢失或被盗，可启用远程擦除。
* **隐私控制：**用于控制位置服务和用户数据访问的iOS功能。

# 系统安全

系统安全的设计使得软件和硬件在每个iOS设备的所有核心组件中都是安全的。这包括启动过程，软件更新和Secure Enclave（保存touch id信息的芯片模块）。这个架构是iOS安全的核心，并且永远不会妨碍设备的可用性。

iOS设备上硬件和软件的紧密集成，确保系统的每一个组件都是可信的，并会对整个系统进行验证。从最开始的启动到iOS的软件更新再到第三方app，每一步都经过分析和审查，以帮助确保硬件和软件以最理想的状态运行并且合理地使用资源。

## 安全启动链

启动过程的每一步都包含了由苹果加密签名以确保完整性的组件，并且只会在验证了信任链之后继续运行，这包括引导程序，内核，内核扩展和基带固件。这个安全启动链帮助确保了最底层的软件不会被篡改。

当一个iOS设备启动时，它的应用处理器会立即从被称为引导ROM(Boot ROM)的只读存储器执行代码。这个不可变的代码，被称作信任硬件根源(the hardware root of trust..实在不造怎么翻)，在芯片制造的时候就被装入，并且被隐式地添加信任。这个引导ROM的代码包含了Apple Root CA公钥，被用来验证iBoot引导程序在加载之前是不是被苹果签名。这是信任链中的第一步，每一步都确保下一步由苹果签名。当iBoot完成它的任务后，它会审查并运行iOS内核。对于用S1，A9或者早期的A系列处理器的设备，会有一个额外的底层引导程序（LLB,Low-Level Bootloader）阶段由引导ROM加载和验证，再反过来加载和验证iBoot。

如果这个引导过程中有一步不能正确加载或者不能验证下一步，启动就会终止，并且设备会在屏幕上显示"Connect to iTunes"。这叫做恢复模式(recovery mode)。如果引导ROM不能加载或者验证LLB，设备就会进入设备固件升级(DFU,Device Firmware Upgrade)模式。在这两种情况下，设备都必须通过USB连接到iTunes，然后恢复到出厂默认设置。想知道更多的手动进入恢复模式的信息，点击https://support.apple.com/kb/HT1808。

在有蜂窝网络接入的设备上，基带子系统还会利用由基带处理器验证的带有签名的软件和密钥来自定义自己的类似的安全启动过程。

对于有Secure Enclave模块(就是带touch id)的设备，Secure Enclave协处理器也会利用一个安全启动步骤来保证它单独的软件也经过苹果的验证和签名。

## 系统软件授权

苹果定期发布软件更新，来解决新兴的安全问题，并且提供新的功能特性，这些更新会同时提供给所有支持的设备。用户在设备上通过iTunes收到iOS更新通知，并且更新是通过无线方式推送的，鼓励用户快速地更新最新安全修复。

上述的启动过程保证了只有苹果设计的代码才能被安装在设备上。为了防止设备被降级到缺乏最新安全更新的旧版本，iOS使用了一个流程叫做系统软件授权（System Software Authorization）。如果系统降级可行，持有设备的攻击者可以安装一个旧版本的iOS，开发出一些在已经在新版本中修复了的漏洞。

在带有Secure Enclave模块的设备上，Secure Enclave协处理器会利用系统软件授权来保证它软件的完整性，并且防止降级安装。详情请看接下来的"Secure Enclave"章节。

iOS软件更新可以使用iTunes进行安装，或者通过空中下载技术(OTA)。用iTunes的话，会有一个完整的iOS系统拷贝下载并安装。OTA方式的软件更新则是采用增量更新，而不是下载整个系统，这样可以改善网络效率。另外，软件更新可以被缓存在本地网络服务器上，以便iOS设备不需要访问苹果服务器来获得必要的更新数据。

在一个iOS系统的升级过程中，iTunes(或设备本身)连接到Apple安装授权服务器，并且向其发送安装包的每个部分的密码测量列表（例如，iBoot，内核，系统操作映像），一个随机的抗重放值，还有设备的ECID。

授权服务器会根据允许安装的版本检查显示的测量列表（list of measurements），如果发现了一个配对，便将此ECID加入到测量列表里并签署结果。服务器将一组完整的签名数据作为升级的一部分传送给设备。添加ECID"个性化"了请求设备的授权。通过仅对已知的测量进行授权和签名，服务器可保证更新完全为苹果提供的。

引导时间信任链评估验证了签名来自苹果，从磁盘加载的项目测量结合设备的ECID匹配了签名的覆盖范围。

这些步骤保证了授权是在一个特定的设备上，并且一个旧的iOS版本不能从一台设备拷贝到另外一台设备。这种特定情况(The nonce)防止了攻击者保存服务器的响应并将其用于篡改设备或以其他方式更改系统软件。


