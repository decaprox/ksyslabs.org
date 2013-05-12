---
title: Gateway
---

Gateway
==========

This example shows the use network between two L4Linux on Genode. We have two instances of L4linux and two network interfaces.

~~~
                       +----------------+              +----------------+
                       |   L4Linux 1    |              |   L4Linux 2    |
  +-------+            |----------------|              |----------------|          +-------+
  | NIC 1 |       +----+                +----+    +----+                +----+     | NIC 2 |
  |-------|       |Nic |                |Nic |    |Nic |                |Nic |     |-------|
  |       |<------+clnt|                |srv |<---+clnt|                |clnt+---->|       |
  |       |       |eth0|                |eth1|    |eth0|                |eth1|     |       |
  +-------+       +----+                +----+    +----+                +----+     +-------+
                       +----------------+              +----------------+
~~~
{:.language}

The first issue is that the current version of Genode dde_ipxe cannot use two instances NIC driver. We've extend the NIC driver configuration and required NIC interface can be selected via <config>. The second part of this issue is IRQ remapping for PCI devices in Genode. PCI IRQ can be remapped via ACPI or MSI. But Genode's ACPI driver is broken. MSI isn't implemented yet. We have temporary hacked this issue via moving PCI IRQ handling to PCI driver.

Second issue is second NIC interface in L4Linux. We have wrote driver for l4linux which implemented NIC service and extend Genode's net driver for l4linux for working with more that one NIC interface.

Sources for this example is available in l4linux-net branch on our [Genode fork](https://github.com/Ksys-labs/genode/tree/l4linux-net) at Github.

Example of configuration is available in iloskutov/run/l4linux2.run

About this configuration:

~~~
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
~~~
{:.language-xml}

We configure which NIC use via <config> <nic device=“00:04.0”/> </config>, where 00:04.0 is address of NIC device on PCI bus.

2. L4Linux's configuration

~~~
	<start name="l4linux.1">
		<binary name="l4linux-srv"/>
		<resource name="RAM" quantum="200M"/>
		<config args="mem=128M console=tty1 console=ttyS0 l4x_rd=initrd.gz l4x_cpus=1 l4x_cpus_map=0"/>
		<provides>
			<service name="Nic"/>
		</provides>
		<route>
			<service name="Input">       <child name="linux.1.fb"/> </service>
			<service name="Framebuffer"> <child name="linux.1.fb"/> </service>
			<service name="Nic">         <child name="nic_drv.1"/> </service> 
			<service name="Terminal">    <child name="linux.1.log"/> </service>
			<any-service> <parent/> <any-child/> </any-service>
		</route>
	</start>
	<start name="l4linux.2">
		<binary name="l4linux-clnt"/>
		<resource name="RAM" quantum="200M"/>
		<config args="mem=128M console=tty1 console=ttyS0 l4x_rd=initrd.gz l4x_cpus=1 l4x_cpus_map=1 genode_net.if_nums=2"/>
		<route>
			<service name="Input">       <child name="linux.2.fb"/> </service>
			<service name="Framebuffer"> <child name="linux.2.fb"/> </service>
			<service name="Nic">
				<if-arg key="label" value="eth0"/> <child name="nic_drv.2"/>
			</service>
			<service name="Nic">
				<if-arg key="label" value="eth1"/> <child name="l4linux.1"/>
			</service>
			<service name="Terminal">    <child name="linux.2.log"/> </service>
			<any-service> <parent/> <any-child/> </any-service>
		</route>
	</start>
~~~
{:.language-xml}

The first l4linux provides Nic service. For enable it add CONFIG_NET_NIC_SRV=y to l4linux configuration, see iloskutov/src/l4linux-srv/x86_32/linux_config.x86_32
The second l4linux uses two Nic interfaces. For enable it add genode_net.if_nums=2 to linux command line. For route Nic interfaces is used if-arg mechanism, lines 23-28

Building
==========

Pull sources from Github and switch to l4linux-net branch.

git clone git://github.com/Ksys-labs/genode.git ksyslabs-genode
cd ksyslabs-genode
git checkout l4linux-net
Prepare modules base-foc, libport, dde_ipxe and ports-foc

~~~
make prepare -C base-foc
make prepare -C libport
make prepare -C dde_ipxe
make prepare -C ports-foc
~~~
{:.language-text}


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