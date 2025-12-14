 https://planetscale.com/blog/io-devices-and-latency

## Non volatile
- We mainly talk about Disks for io in this blog , Tapes , spinnig disks , Nvme , the main object was to show how these days we are moving to architecture which seperate's compute and storage in the cloud , Planetscale's metal is solution for this kind of approach .
- Metal uses locally attached NVMe drives to run cloud dbs ,while most cloud providers use a slower network attached storage .Metal performs with a unlimited io (Other cloud providers Cap this for even performance for users ) & low latency. 
- Tape storage - cheap , works best for data which is not accessed for a long time but random reads/writes are not great (sequential situations are best case scenarios only)
- HDD - Vanilla , we have good random r/w as we have full view of the storage block compared to tape storage where we need move the entire window .
- SSD - Its a HDD but on more moving parts , it works electronically entirely , we use a special transistor for r/w , NAND flash . HDD 2ms vs SSD 16mew-s ,Organisation of data still matters as we can parallelise this as well and plays a important role when we need to evict pages as well .Garbage collection for removing/evict the dirty pages (places where we have data which is not required ) effectively 
- Storage in the cloud - Network attached storage
- Main thought data durability & scalability while keeping IOPS performance , DD through replication , scalability when we use fixed disk sizes will reach capacity and we have to manually mange that (Metal ................ magic)