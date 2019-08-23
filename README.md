# qs-oracle
Oracle Database on the KSC: Quick Start Reference Deployment



Oracle应用迁移上云解决方案



    用PPAS进行Oracle应用迁移上云架构
    共享存储实现Oracle应用云上HA架构
    共享存储实现Oracle应用云上RAC架构
    Oracle应用云上容灾备份架构
    Oracle应用云上异地容灾架构
    Oracle应用云上/云下容灾备份架构



随着云计算的发展趋于成熟，上云是大势所需，企业需要一站式的Oracle解决方案和服务。



利用阿里云VPC构建安全网络环境，在不同地域部署Oracle主备实例，例如华东区部署Oracle实例主库，在华南区部署Oracle实例备库，通过ADG做2个数据库之间灾备架构，备库通过rman方式将数据备份普通云盘，再上传到OSS对象存储中，可自定义设置备份策略，包括全量、增量等方式。 通过云上跨区域的容灾环境，可以实现简单高效的Oracle容灾架构以及云上备份环境，可在备库上备份，减小主库压力，可使用最大保护模式，确保数据不丢失。



https://market.aliyun.com/oraclemigration


http://www.cldera.com/


北京君云时代科技有限公司是国内少数几家业务完全基于云计算的服务型公司，专注互联网业务，提供一站式运维服务解决方案。

https://d0.awsstatic.com/whitepapers/aws-advanced-architectures-for-oracle-db-on-ec2.pdf

StorageFor database storage, AWS users are encouraged to use Amazon EBS. For high and consistent IOPS, wehighly recommend usingGeneral Purpose (GP2) volumesor Provisioned IOPS (PIOPS)volumes. GP2 can provide upto 10,000 IOPS per volume,and PIOPS can provide up to 20,000 IOPS per volume.GP2volumes provide an excellent balance of priceand performance for most database needs. When very high IOPS is required,PIOPS volumes aretheright choice.Stripe multiple volumes together for more IOPS and larger capacity. You can use multiple Amazon EBSvolumes individually for different data files,but striping them together allows better balancing and scalability.Oracle Automatic Storage Management (ASM)can be used for striping.Keep datafiles, log files,and binaries on separate EBSvolumes,and take snapshots oflog filevolumes on a regular basis.Choosing an instance type with local SSD storage allowsyou to boost the database performance by using Smart Flash Cache (if the operating system is Oracle Linux) and by using local storage for temporary filesand table spaces.Most Oracle Database users take regular hot and cold backups. Cold backupsare done whilethe database isshutdown, whereas hot backupsare taken while the database is active. Store your hot and cold backups inAmazon Simple Storage Service (Amazon S3)for high durability and easy access.Amazon Storage Gateway or Oracle Secure Backup Cloud Module can be used to directly backup the database to Amazon S3. Life-cycle policies can be applied to the backups in Amazon S3to move older backups to Amazon Glacier for archiving


https://amazonaws-china.com/cn/blogs/china/idata-aws-oracle-rac-group/
使用 iData 在 AWS 云上实现高性能的 Oracle RAC 集群

提供了测试方法


在企业上云的过程中， 我们经常不免碰到很多传统 Oracle RAC 的企业用户，他们使用Oracle RAC 来运行其关键任务应用程序，包括大多数金融机构、电信运营商、甚至一些大型零售商，其中高可用性和数据完整性至关重要。如何帮助他们将本地数据中心的 Oracle RAC 迁移到 AWS 云上，客户通常存在很多顾虑和疑问，比如：

    企业应用软件采用的是第三方的商业应用软件，在上云过程中无法对软件做适应性改造
    单节点的 AWS 托管 Oracle RDS 如何解决高 IOPS 吞吐率要求， 5万+ 、 10万+ 、甚至30万+的 IOPS 要求。
    AWS EC2 上自建 Oracle RAC 如何解决 Share disk 的问题，如何在线弹性增加容量，增加 IOPS 容量。
    如何最小化 Oracle RAC 应用迁移的风险。


Amazon Web Services (AWS) : Installation of Oracle on EC2

https://oracle-base.com/articles/vm/aws-ec2-installation-of-oracle