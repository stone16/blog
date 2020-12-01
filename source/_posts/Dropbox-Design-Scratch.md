---
title: Dropbox Design Scratch
date: 2020-11-30 18:54:17
categories: SystemDesign
tags:
top:
---

# 1. Intro and requirement 

+ cloud file storage services 
    + simplify the storage and exchange of digital resources among multiple devices 
    + benefits 
        + availability 
        + reliability and durability 
        + scalability 

+ requirements and goals 
    + users should be able to upload and download their files/ photos from any device 
    + users should be able to share files or folders with other users 
    + service should support automatic synchronization between devices 
    + support storing large files 
    + ACID support 
    + Offline editing, CURD offline, and get synced when online 
    + snapshotting of the data 


# 2. Thoughts 

+ size, limit , pattern 
    + read heavy 
    + write huge file 
+ synchronize among different devices
    + all devices make call to backend to grab newest status 

+ file/ folder share 
    + permission management 

+ Offline editing 
    + Use queue to store the data locally? 


# 3. Design 


## 3.1  considerations 
    + huge read and write volumes
        + read write almost equal 

    + files can be stored in <u>**small parts or chunks (4MB)**</u>, when fails, only the failed chunk should be retried 
    + reduce the amount of data exchange by transfering updated chunks only 
    + keep a local copy of metadata (file name, size, etc.) with the client can save us a lot of round trips to the server 

+ high level design 
    + need to store files and metadata information like File name, file size, directory, who this file is shared with 
    + need some servers that can help the clients to upload/ download files to cloud storage and some servers that can facilitate updating metadata about files and users
    + need some mechanism to notify all clients whenever an updates happens so they can synchronize their files 

## 3.2 high level design 
    + user specify folder as the workspace on the device 
        + file in the folder will be uploaded to the cloud 
        + updater/ delete will be reflected in the same way 

    + modification on one device should be freely synced across others 
    + systems we need based on requirements 
        + a storage server to store all real files 
        + a metadata service store file metadata, thus we could quickly get info about it 
        + a synchronizer service, to notify all clients whenever an update happens so they can synchronize files 
        
## 3.3 Component Design 

### 3.3.1 Client 
+ responsibility 
    + need to monitor the workspace folder on user's machine 
    + sync files/ folders with remote cloud storage 
    + interact with the remote synchronization service to handle file metadata updates 

+ how to handle file transfer efficiently
    + break each file into smaller chunks so that we transfer only those chunks that are modified and not the whole file 
    + calculate chunk size based on:
        + storage devices we use in the cloud 
        + network bandwidth 
        + average file size in the storage 

    + need to keep a record of each file and the chunks in metadata 

+ shall we keep a copy of metadata with client? 
    + yes, it then allows us to do offline updates 

+ how can clients efficiently listen to changes happening with other clients? 

    + Solution 1
        + clients periodically check with the server if there are any changes 
        + issues 
            + have a delay in reflecting changes locally as clients will be checking for changes periodically compared to server notifying whever there is some change 

    + Solution 2
        + HTTP long polling 
        + client requests information from the server with the expectation that the server may not respond immediately 
        + servers hold the request open and waits for response information to become available 

+ components 
    + inernal metadata db 
        + keep track of all the files, chunks, versions and location 

    + chunker 
        + split files into smaller pieces 
        + reconstruct a file from its chunks 

    + watcher 
        + monitor the local workspace folders and notify the indexer of any action performed by the users 

    + indexer 
        + process the events received from the watcher and update the internal metadata database with information about the chunks of the modified files 
        + once confirm chunks are successfully uploaded to the cloud storage, the indexer will communicate with the remote synchronization service to broadcast changes to other clients and update remote metadata database 

+ for phone users, probably should sync on demand to save bandwidth and capacity 

### 3.3.2 Metadata Database 

+ responsible for maintaing the versioning and metadata information about files/ chunks, users and workspaces 
+ store info like 
    + chunks 
    + files 
    + users
    + devices
    + workspace 

### 3.3.3 Synchronization Service 
+ component that processes file updates made by a client and apply these changes to other subscribed clients 
+ also synchronizes clients local databases with the information stored in the remote metadata db 
+ Implement a differenciation algo to reduce the amount of the data that needs to be synchronized 
    + just transmit the difference between two versions instead of the whole file 

+ to support a scalable synchronization protocol
    + use a communication middleware 
    + messaging system 
        + push or pull strategies 

### 3.3.4 Message Queuing Service 

+ Message queue service 
    + supports asynchronous message based communication between clients and the synchronization service 

### 3.3.5 Cloud Storage 
Cloud/Block Storage stores chunks of files uploaded by the users. Clients directly interact with the storage to send and receive objects from it. Separation of the metadata from storage enables us to use any storage either in the cloud or in-house.
# Reference 
1. [Grokking the system design](https://www.educative.io/courses/grokking-the-system-design-interview/m22Gymjp4mG)
2. [Tech Dummies](https://www.youtube.com/watch?v=U0xTu6E2CT8&t=9s&ab_channel=TechDummiesNarendraL)