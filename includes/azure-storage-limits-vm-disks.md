Azure 虚拟机支持附加许多的数据磁盘。 本文介绍 VM 的数据磁盘的可伸缩性和性能目标。 使用这些目标可帮助确定满足性能和容量要求所需的磁盘数量和磁盘类型。 

> [!IMPORTANT]
> 为了获得最佳性能，需要限制附加到虚拟机的、重度使用的磁盘数，以避免可能的性能限制。 如果附加的所有磁盘未在同时重度使用，则虚拟机可以支持更多的磁盘。

* **对于 Azure 托管磁盘：**托管磁盘按区域和按磁盘类型进行磁盘限制。 一个订阅的最大限制（也是默认限制）为每个区域、每个磁盘类型 10,000 个托管磁盘。 例如，可以在每个订阅及一个区域中最多创建 10,000 个标准托管磁盘和 10,000 个高级托管磁盘。

    托管的快照和映像计入托管磁盘限制。

* **标准存储帐户：**标准存储帐户的总请求率上限为 20,000 IOPS。 在标准存储帐户中，所有虚拟机磁盘的 IOPS 总数不应超过此限制。
  
    可以根据请求率的限制，大致计算单个标准存储帐户可支持的重度使用磁盘数。 例如，对于基本层 VM，重度使用的磁盘数上限约为 66（每个磁盘 20,000/300 IOPS）；对于标准层 VM，约为 40（每个磁盘 20,000/500 IOPS）。 

* **高级存储帐户：**高级存储帐户的总吞吐量速率上限为 50 Gbps。 所有 VM 磁盘的总吞吐量不应超过此限制。

