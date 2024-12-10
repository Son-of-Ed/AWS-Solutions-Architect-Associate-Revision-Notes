The SNOW family of products are portable offline devices used to bring computational resources closer to data sources which can help reduce latency and save bandwidth. The SNOW products are also useful  for data migration into and out of AWS (specifically into and out of S3 buckets and block storage).
**NOTE: You cannot put SNOW data directly into the S3 Glacier tier when it is uploaded to S3, you have to first load it into S3 then create a lifecycle policy.**

AWS will send you a physical device to copy your data onto which you then send to its destination. *If it takes more than a week to transfer over your network, use a snowball device.* This allows you to reduce bandwidth usage which may block others in your network from using it.

- Snowcone - small, portable device for Edge computing or data transfer. You have to provide your own cables and batteries. Used in space-constrained environments where a Snowball wouldn't fit. Data can be sent to AWS offline or it can use the Internet and AWS DataSync installed on the device. Has 2 CPUs + 4 GB RAM.
	- Snowcone - 8TB of HDD storage
	- Snowcone SSD - 14TB of SSD storage
- Snowball Edge - used for data transfer (<80TB) or for greater processing power + storage capacity than the Snowcone can handle.
	- Storage Optimised - 80TB HDD, 80GiB RAM, 40 vCPUs
	- Compute Optimised - 42TB HDD or 28TB NVMe capacity, 104 vCPUs, 416 GiB RAM (optional GPU as well).
- Snowmobile - like a Snowball Edge but an *actual* truck. Used to transfer PBs of data. Each truck has 100PB of storage but you can use multiple in parallel. They are very secure compared to the other SNOW offerings. Tent to be used for data transfers >10PB.

Both the Snowcone and Snowball Edge devices are capable of running EC2s and Lambda functions (using AWS IoT Greengrass). They also come with 1/3 year reservations for discounted pricing.

Points of Presence (Edge locations) are any locations that do not have access to the Internet or compute resources but have a source of data that needs to be processed. You can begin to process data while the edge data is in transit or perform machine learning on data separately from the cloud.

All of the above services can run EC2s and Lambda functions.

## OpsHub
This is a downloadable programme that allows you to interact with your SNOW devices.
You can transfer files, launch + manage instances running on SNOW devices, monitor device metrics, and launch compatible AWS services on your devices.






#aws