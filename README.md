# A fault-tolerant distributed file system
This project based on Java. It's a fault-tolerant distributed file system that supports concurrent read/write operations using AWS, Java RMI, and the Primary protocol.

The system has good scalability which can handle 10,000 requests with a maximum of 10,000 requests running concurrently. Amazon EC2 instances are used to handle large data transfer workloads. Comparing with other OS such as Google File System, our system is relatively easier to be implemented and maintained. 

# System Design Overview
In designing a distributed file system for our goals, we describe our system in more detail:
   1. Each server and each client maintains metadata about the replicas and their IP addresses. To choose the primary server, a client ping all the server and choose the server with the lowest ping as the primary server. 
   2. The user can submit multiple operations that are performed atomically on the shared files stored in the distributed file system. Each transaction accesses only one file. A transaction that happens out of the lease period will be dropped.
   3. Files are not partitioned and each server maintains the same files. This means that once all transactions have been done, a primary server needs to send updates to other replicas to ensure that the file system remains in a consistent state.
   4. Files are read entirely. For write operate, a client sends a write request to the server. The server locks the file and sends a lock command to all the other servers so that it could not be modified. Then the primary server grants a lease of 10 mins to the client and sends the file the to client.On client side the file is opened using to notepad. The client can edit the content on the notepad file. Once saved, the client using commit command can update the file. Now again on the server side, first the file is updated on primary server and then on remaining servers. Then an unlock operation is performed on primary server followed by replica servers.
 
   5. Heartbeat : To allow seamless and error free execution of operations we heartbeat. To replicate the files on replicas the servers needs to be available. We check the availability of servers we send heartbeat messages constantly. If the server doesn't receive any response for the heartbeat message it sent, then it is marked as unavailable. We have implemented the heartbeat by pinging the servers. We maintain two files Servers.txt which contains the list of all servers and other file AvailableServers.txt which contains the list of available servers.
   6. Lease: We use age-based leases in our system design. We assume that most of the files are .txt files. A file has not been modified for a long time is expected to remain unmodified for some time yet to come. By using age-based leases strategy, we can simplify our implementation and reduce the updates message by granting long leases to the unmodified data.

# Client side and Server side implementations
The client’s job is to find the server with lowest latency and then connect to it. The connection is a TCP based connection and the command line acts a user interface to communicate to the server for any operation. When the client runs the client program it gets options such as 1 for creating a file, 2 for reading, 3 for appending, 4 for deleting, 5 for Restoring and 6 for Writing. Each operation is atomic as per our implementation.

The server’s task is to wait and accept a client connection. For the purpose of simplicity we started implementing a single server and multiple clients. So when a client gets connected to server, a new thread is spawned that is responsible for serving requests to that particular client. We implement ServerThread class to handle different operations including: 

Create Operation: This operation creates a file server side. The client needs to pass the username, filename and filetext to create a new file. 

Read Operation: Files are read entirely. A client sends the read request along with filename to the server. A server returns the file back to the client entirely.

Append Operation: Append operation can add additional content to the file.

Write Operation: A client sends a write request to the server. The server locks the file and sends a lock command to all the other servers so that it could not be modified. Then the primary server grants a lease of 1 min to the client and sends the file the to client.

On client side the file is opened using to notepad. The client can edit the content on the notepad file. Once saved, the client using commit command can update the file.

Now again on the server side, first the file is updated on primary server and then on remaining servers. Then an unlock operation is performed on primary server followed by replica servers.

Now if the file is locked by some user and another user tries to access it then we return a message to the client saying the file is locked.

Delete Operation: Calling this operation will send a request to server to delete the file. On the server side the file is actually being renamed.

Restore Operation: This operation will restore a previously deleted file. As the file is not being actually deleted on the server, we can rename the file to old name and restore it.
