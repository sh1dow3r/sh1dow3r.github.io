---
layout: post
title:  "NAS Agent"
categories: Auth update
---
Network Attached Storage has been around for a while, however using that for backup has been increasing a non-negligible vector of the integrity of the backup. We designed an agent that implements an encrypted backup for the client to provide efficiency and fast disaster recovery. 


Network attached Storage(NAS) is an approach to making stored data more accessible among devices on a network. Also viewed as dedicated file storage that enables multiple users and heterogeneous client devices to retrieve data from centralized disk capacity. By installing specialized software on dedicated hardware, Usually, enterprises can benefit from this as single-point access with built-in security, management, and fault tolerant capabilities. NAS communicates with other devices using file-based protocols, which is one of the easiest formats to navigate. 

In the code, a function was made to make use of SSH to send data to the server side using the client's account in the server. In the server, each user has an account and password or public key that is stored in the account's home directory. Using a password or public and private keys as figure 2 shows, the client side would be able to process the SSH authentication mechanism then send data each time it processes a message from the client. An example of the data that could be sent and process, when a client wants to create or delete a partition the client's software gets the name and space from the user then sends the name of the new partition and how much space it takes. After adding or deleting a partition, the client's software sends a mount request using NFS protocol to process the next step. 

NFS is used after the client asks for a new partition to be created then the server sends the confirmation for the new disk, and then again the client uses NFS to mount the new partition from the storage server as figure 3 shows. This way a new partition is made on the client's device and the storage server. The advantages of using NFS, in this case, is that the client is able to mount different partitions from different storage servers without the need to its account if all storage servers are synced, as figure 5 shows. After mounting them, all the partitions will be mounted even after rebooting the device since they will be appended to /etc/fstab, and whenever a partition gets deleted the line associated with the partition will be removed. All that is constantly going each time a new request is being pushed to the server.

Finally, the project is made yet under tests, the idea is to implement software that can backup files always without the need to uses third-party resources, it only uses SSH and NFS and pure implementation using Python so it can be easy to use for users. Installing and configuring the client is fairly easy. Installing and configuring the server is also simple. However, each user has to be added manually to the server, which must be Unix-like operating system. The hardware used in this experiment are two devices one acts as a server and the other one acts as a client. The server was Cent OS 7.2 and the client was also using Cent OS 7.2. The server had 6 TB of space and 16 GB of RAM, 4 cores and each core was producing 3.2GHz. The client had 100 GB of space and 16 GB of RAM, 4 cores and each core was producing 3.2GHz.


