> [!div class="op_single_selector"]
- [门户](../articles/virtual-network/virtual-network-multiple-ip-addresses-portal.md)
- [PowerShell](../articles/virtual-network/virtual-network-multiple-ip-addresses-powershell.md)
- [CLI](../articles/virtual-network/virtual-network-multiple-ip-addresses-cli.md)
- [模板](../articles/virtual-network/virtual-network-multiple-ip-addresses-template.md)

一个 Azure 虚拟机 (VM) 具有一个或多个附加的网络接口 (NIC)。可为任何 NIC 分配一个或多个静态/动态的公共和专用 IP 地址。将多个 IP 地址分配给 VM 可实现以下功能：

* 在一台服务器上托管具有不同 IP 地址和 SSL 证书的多个网站或服务。
* 用作网络虚拟设备，例如防火墙或负载均衡器。
* 能够将任何 NIC 的任意专用 IP 地址添加到 Azure Load Balancer 后端池。过去，只有主 NIC 的主 IP 地址才能添加到后端池。若要了解有关如何对多个 IP 配置进行负载均衡的详细信息，请阅读[对多个 IP 配置进行负载均衡](../articles/load-balancer/load-balancer-multiple-ip.md)一文。

每个附加到 VM 的 NIC 都具有一个或多个与之关联的 IP 配置。系统会为每个配置分配一个静态或动态专用 IP 地址。每个配置还可拥有一个与之关联的公共 IP 地址资源。系统会为公共 IP 地址资源分配一个动态或静态公共 IP 地址。若要详细了解 Azure 中的 IP 地址，请阅读 [Azure 中的 IP 地址](../articles/virtual-network/virtual-network-ip-addresses-overview-arm.md)一文。

<!---HONumber=Mooncake_1226_2016-->