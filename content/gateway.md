---
title: Gateway
---

Gateway
==========

This example shows the use network between two L4Linux on Genode. We have two instances of L4linux and two network interfaces.

                       +----------------+              +----------------+
                       |   L4Linux 1    |              |   L4Linux 2    |
  +-------+            |----------------|              |----------------|          +-------+
  | NIC 1 |       +----+                +----+    +----+                +----+     | NIC 2 |
  |-------|       |Nic |                |Nic |    |Nic |                |Nic |     |-------|
  |       |<------+clnt|                |srv |<---+clnt|                |clnt+---->|       |
  |       |       |eth0|                |eth1|    |eth0|                |eth1|     |       |
  +-------+       +----+                +----+    +----+                +----+     +-------+
                       +----------------+              +----------------+

The first issue is that the current version of Genode dde_ipxe cannot use two instances NIC driver. We've extend the NIC driver configuration and required NIC interface can be selected via <config>. The second part of this issue is IRQ remapping for PCI devices in Genode. PCI IRQ can be remapped via ACPI or MSI. But Genode's ACPI driver is broken. MSI isn't implemented yet. We have temporary hacked this issue via moving PCI IRQ handling to PCI driver.

Second issue is second NIC interface in L4Linux. We have wrote driver for l4linux which implemented NIC service and extend Genode's net driver for l4linux for working with more that one NIC interface.

Sources for this example is available in l4linux-net branch on our [Genode fork](https://github.com/Ksys-labs/genode/tree/l4linux-net) at Github.

Example of configuration is available in iloskutov/run/l4linux2.run

About this configuration:

1. Launch two NIC drivers:
	<start name="nic_drv.1">
		<binary name="nic_drv"/>
		<config>
			<nic device="00:03.0"/>
		</config>
		<resource name="RAM" quantum="8M"/>
		<provides><service name="Nic"/></provides>
	</start>
	<start name="nic_drv.2">
		<binary name="nic_drv"/>
		<config>
			<nic device="00:04.0"/>
		</config>
		<resource name="RAM" quantum="8M"/>
		<provides><service name="Nic"/></provides>
	</start>

We configure which NIC use via <config> <nic device=“00:04.0”/> </config>, where 00:04.0 is address of NIC device on PCI bus.

2. L4Linux's configuration

 1	<start name="l4linux.1">
 2		<binary name="l4linux-srv"/>
 3		<resource name="RAM" quantum="200M"/>
 4		<config args="mem=128M console=tty1 console=ttyS0 l4x_rd=initrd.gz l4x_cpus=1 l4x_cpus_map=0"/>
 5		<provides>
 6			<service name="Nic"/>
 7		</provides>
 8		<route>
 9			<service name="Input">       <child name="linux.1.fb"/> </service>
10			<service name="Framebuffer"> <child name="linux.1.fb"/> </service>
11			<service name="Nic">         <child name="nic_drv.1"/> </service> 
12			<service name="Terminal">    <child name="linux.1.log"/> </service>
13			<any-service> <parent/> <any-child/> </any-service>
14		</route>
15	</start>
16	<start name="l4linux.2">
17		<binary name="l4linux-clnt"/>
18		<resource name="RAM" quantum="200M"/>
19		<config args="mem=128M console=tty1 console=ttyS0 l4x_rd=initrd.gz l4x_cpus=1 l4x_cpus_map=1 genode_net.if_nums=2"/>
20		<route>
21			<service name="Input">       <child name="linux.2.fb"/> </service>
22			<service name="Framebuffer"> <child name="linux.2.fb"/> </service>
23			<service name="Nic">
24				<if-arg key="label" value="eth0"/> <child name="nic_drv.2"/>
25			</service>
26			<service name="Nic">
27				<if-arg key="label" value="eth1"/> <child name="l4linux.1"/>
28			</service>
29			<service name="Terminal">    <child name="linux.2.log"/> </service>
30			<any-service> <parent/> <any-child/> </any-service>
31		</route>
32	</start>

The first l4linux provides Nic service. For enable it add CONFIG_NET_NIC_SRV=y to l4linux configuration, see iloskutov/src/l4linux-srv/x86_32/linux_config.x86_32
The second l4linux uses two Nic interfaces. For enable it add genode_net.if_nums=2 to linux command line. For route Nic interfaces is used if-arg mechanism, lines 23-28

Building
==========

Pull sources from Github and switch to l4linux-net branch.

git clone git://github.com/Ksys-labs/genode.git ksyslabs-genode
cd ksyslabs-genode
git checkout l4linux-net
Prepare modules base-foc, libport, dde_ipxe and ports-foc

make prepare -C base-foc
make prepare -C libport
make prepare -C dde_ipxe
make prepare -C ports-foc
Create build directory:

./tool/create_builddir foc_x86_32 BUILD_DIR=build.foc_x86_32
cd build.foc_x86_32
Modify etc/build.conf. Uncomment repositories gems, libport, dde_ipxe and ports-foc and add REPOSITORIES += $(GENODE_DIR)/iloskutov

Running
==========

make run/l4linux2

Testing internal network
==========

Configure eth1 in both l4linux and try to ping from one to other.