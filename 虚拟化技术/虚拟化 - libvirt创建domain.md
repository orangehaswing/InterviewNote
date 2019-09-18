# libvirt 创建domain

函数调用流程

libvirt -domain.c virDomainCreate

libvirt-host.h virConnectPtr：conn ->driver->domainCreate












=qemuDomainCreate 
qemuDomainCreateWithFlags
qemuDomainObjStart 
qemuProcessStart
virCommandPtr cmd = NULL;
virQEMUDriverConfigPtr cfg ;
virCommandRun
virCommandRunAsync 
virExec 
详细分析

## virDomainCreate

G:\libvirt-1.1.1\src\libvirt.c 

virDomainCreate 

主要流程就是调用
ret = conn->driver->domainCreate(domain); 

## qemuDomainCreate

G:\libvirt-1.1.1\src\qemu\qemu_driver.c 

对于 qemu_driver 

注册的驱动为
static virDriver qemuDriver = {
.no = VIR_DRV_QEMU,
.name = QEMU_DRIVER_NAME ,
connectOpen = qemuConnectOpen, /* 0.2.0 */
.connectClose = qemuConnectClose , /* 0.2.0 */
.connectSupportsFeature = qemuConnectSupportsFeature , /* 0.5.0 */
.connectGetType = qemuConnectGetType , /* 0.2.0 */
.connectGetVersion = qemuConnectGetVersion , /* 0.2.0 */
.connectGetHostname = qemuConnectGetHostname , /* 0.3.3 */
connectGetSysinfo = qemuConnectGetSysinfo , /* 0.8.8 */
.domainCreate = qemuDomainCreate , /* 0.2.0 */
.domainCreateWithFlags = qemuDomainCreateWithFlags , /* 0.8.2 */
.domainDefineXML = qemuDomainDefineXML , /* 0.2.0 */
...
...
};
从中可以查到，对应的函数为 qemuDomainCreate


static int
qemuDomainCreate(virDomainPtr dom )
{
return qemuDomainCreateWithFlags (dom, 0);
}
qemuDomainCreate 实际使用的是 qemuDomainCreateWithFlags(dom, 0);

qemuDomainCreateWithFlags
查询 domain 是否 define，没 define，则返回
进行 acl 检查，必须有足够的权限
查询 domain 是否已经存在，存在，则返回不创建。
调用 qemuDomainObjStart 进行创建 Domain object

qemuDomainObjStart
如果这个对象以前被创建过，只是目前被 destory 了，那么会有一些管理这个对象的配置信息
已经 save 到某个路径下。那么需要从原先路径下，删除它的原先管理信息。
调用函数 qemuProcessStart 进行创建。
virDomainAuditStart 函数是审计信息。进行一些运行时内容的统计。



5. qemuProcessStart
   这个函数是 guest OS 创建的主要函数。里面最重要的 2 个参数是
   virCommandPtr cmd
   virQEMUDriverConfigPtr cfg
   virCommandPtr 是保存创建命令的数据结构。重要的命令传入信息，都通过字符串格式传入，
   然后解析为对应的数据项，保存在这个结构中。
   struct _virCommand {
   int has_error; /* ENOMEM on allocation failure, -1 for anything else.
   */
   char **args; size_t nargs ; size_t maxargs ; char **env; size_t nenv ; size_t maxenv ; char *pwd; size_t npassfd ; virCommandFDPtr passfd ; unsigned int flags; char *inbuf; char **outbuf; char **errbuf;
   int infd;
   int inpipe;
   int outfd;
   int errfd;
   int *outfdptr;
   int *errfdptr; virThreadPtr asyncioThread ;








