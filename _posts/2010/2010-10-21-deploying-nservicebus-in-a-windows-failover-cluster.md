---
layout: post
status: publish
published: true
title: Deploying NServiceBus in a Windows Failover Cluster
author:
  display_name: David Boike
  email: david.boike@gmail.com
  url: http://www.make-awesome.com
date: '2010-10-21 10:30:11 -0500'
date_gmt: '2010-10-21 15:30:11 -0500'
categories:
- Development
tags:
- NServiceBus
- load balancing
- TimeoutManager
- failover cluster
- cluster
- msmq
comments: true
---
NServiceBus is made from the ground up for scalability and reliability, but to take advantage of these features, you need to deploy it in a Windows Failover Cluster. Unfortunately, information on how to do this effectively is, as yet, incomplete and scattered. This article will describe the process for deploying NServiceBus in a failover cluster.

<!-- more -->

What this article does *not* cover is the generic setup of a failover cluster itself. There are other, better resources for that. Try [Creating a Cluster in Windows Server 2008](http://blogs.msdn.com/b/clustering/archive/2008/01/18/7151154.aspx) to start. The focus here is the setup related to NServiceBus.

## Planning Your Infrastructure

A simple setup for scalability and reliability will include at least two servers in a failover cluster, and two additional servers.

The failover cluster servers run the following:

-   A Distributor process for each logical message queue.
-   A TimeoutManager, if you require one to support Sagas.
-   Commander application(s)
    -   A Commander application is an application that contains one or more classes that implement IWantToRunAtStartup and coordinate tasks among the other handlers, doing very little work itself but sending messages to start other processes based on timers or other stimuli.
    -   It is important with these applications to set everything up during the Start method and make sure everything is torn down during the Stop method, as the Start method will be called when the service is started, and the Stop method will be called when the service is stopped (to be transferred to the other cluster node).
    -   The Commander application also can have some message handlers of its own, usually to subscribe to events published from other endpoints in the service as a kind of feedback loop to control overall processing flow.

The other two servers are Worker Nodes, and contain only endpoints with simple message handlers. The endpoints request work from the clustered distributors, do the work, and then ask for more.

Originally, I thought that the cluster node servers could also host worker processes, however I could not get this to work, and I'm not really sure why. When the clustered Message Queueing instance was running on the same node as a worker process, the worker process could not send control messages to the distributor in order to get more work. In other words, with ClusterA between N1 and N2, if ClusterA was running on N1, Worker@N2 could send messages to Queue@ClusterA, but Worker@N1 could not. The messages would be sent but would never arrive. I eventually gave up. I would love some input (perhaps from an expert on MSMQ) on what causes this, but for now, I'm comfortable keeping clustered resources and worker resources segregated.

## Setting up the Clustered Service

While technically it shouldn't matter which clustered server you set stuff up from, generally I found it was more reliable to set up everything from whatever server currently held the Quorum disk. Find the server that has it (it moves around when the server holding it is restarted), and open up Failover Cluster Management under Administrative Tools.

The first step is to set up a clustered DTC access point. Right-click Services and Applications and select Configure a Service or Application. Get through the intro screen and select Distributed Transaction Coordinator (DTC) and click Next. Follow the wizard, assigning a DNS name for the clustered DTC (ClusterNameDTC would work), IP address, and storage. When finished, you should get a service with the DTC icon.

Now you need to configure DTC for NServiceBus. On each server, go to Administrative Tools - Component Services, then expand Component Services - Computers - My Computer - Distributed Transaction Coordinator. Here you will see Local DTC, and if the clustered DTC is on the current node, a Clustered DTCs folder with your clustered DTC name inside it. On both instances (so three times counting each node and the clustered instance) right-click, select Properties, and switch to the Security tab. At the very least, check "Network DTC Access" and "Allow Outbound." This is the minimum that is configured by the RunFirst tool included with NServiceBus. Optionally, you may want to check "Allow Remote Clients" and "Allow Inbound." In all my experimenting trying to get the cluster up and running, these got checked on my clustered nodes' local and clustered DTC, and I'm quite frankly unwilling to experiment with removing them on my application. Please, give it a try without and let me know if you have success.

The next step is to set up a MSMQ cluster. This will be the cluster NServiceBus cares about; the name we append to our queue names, where we will later add all the rest of our clustered resources (the distributors, timeout manager, and commander).

In Failover Cluster Management, from the server with Quroum:

1.  Right-click Services and Applications and select Configure a Service or Application.
2.  Skip the intro, and select Message Queuing.
3.  Finish the wizard, configuring the cluster name, IP address, and storage.

This should give you a clustered MSMQ instance. Click the instance under Services and Applications to see the summary, which will contain the Server Name, Storage, and MSMQ instance. Right-click the MSMQ instance, select Properties, and go to the Dependencies tab. Make sure that it contains dependencies for the Cluster Name AND IP Address AND Storage.

You can view the MSMQ MMC snap-in (crappy though it is, I strongly suggest you consider investing in a commercial MSMQ manager option) by right-clicking on the MSMQ cluster in the left pane and selecting Manage MSMQ, which opens the Computer Management tool geared toward the clustered instance. You get to MSMQ from there by expanding Services and Applications - Message Queuing. Keep in mind that this only seems to work if you're viewing Failover Cluster Management from the server where the MSMQ cluster currently resides. If you are on Server A and you try to manage MSMQ on a cluster residing on Server B, you won't see Message Queuing in the Computer Management window.

Try swapping the cluster back and forth between nodes a few times. It's best to make sure that everything is working properly now before continuing.

## Installing the Clustered Services

Before you can cluster the NServiceBus.Host.exe processes, you need to install them as services on all clustered nodes.

Copy the Distributor binary as many times as you have logical queues, and then configure each one as described in the [NServiceBus Distributor](http://www.nservicebus.com/Distributor.aspx) page. In order to keep everything straight, it's important to decide on a naming convention and stick with it.

My queue naming convention is:

-   **Distributor Data Bus:** ProjectName.QueueName
-   **Distributor Control Bus:** ProjectName.QueueName.Control
-   **Distributor Storage Queue:** ProjectName.QueueName.Storage

Just as a review of how the distributor works, other endpoints will send messages to the Data Bus, where they will accumulate if no worker is running. When a worker comes online, it will send a ReadyMessage to the Control Bus. If there is work to be done, the distributor will send an item from the Data Bus to the endpoint's local input queue, otherwise, it will file it in the storage queue so that when work does come in, the distributor will know who is ready to process it.

Using this naming convention, all of your applications' queues will be grouped together, and all of the queues for a logical QueueName will also be grouped together by alphabetical order. Another option is to flip it around (Storage.Project.Queue, Control.Project.Queue, and Data.Project.Queue) which will group all your control queues together, all your storage queues together, and all your data queues together. It's really a matter of personal preference. As I said, just pick a naming convention and stick with it.

Install each distributor from the command line:

    NServiceBus.Host.exe
         /install
         /serviceName:Distributor.ProjectName.QueueName
         /displayName:Distributor.ProjectName.QueueName
         /description:Distributor.ProjectName.QueueName
         /userName:DOMAIN\us
         /password:thepassword
         NServiceBus.Production

I've found that it's easier to set the service name, display name, and description to be the same. It helps when trying to start and stop things from a NET START/STOP command and when viewing them in the multiple graphical tools. Starting each one with Distributor puts them all together alphabetically in the Services MMC snap-in.

Don't forget the "NServiceBus.Production" at the end which sets the profile for the NServiceBus generic host, as described in the [Generic Host](http://www.nservicebus.com/GenericHost.aspx) page of the documentation.

Do not try starting the services. If you do, they will be running in the scope of the local server node, and will attempt to create their queues there.

Now, add each distributor to the cluster:

1.  Right-click your MSMQ cluster, and select Add a Resource - \#4 Generic Service.
2.  Select your distributor service from the list. The services are listed in some nonsensical order by typing "Distributor" will get you to the right spot if you named your services as I did above.
3.  Finish the wizard. The service should be added to the cluster, but not activated. **Don't activate it yet!**
4.  Right click the distributor resource and select Properties.
    -   Now this is where it gets weird.
    -   You need to check "Use Network Name for computer name" and you also need to add a dependency, but **you can't do both at the same time!** If you do it will complain that it can't figure out what the network name is supposed to be because it can't find it in the dependency chain, which you told it, but it hasn't been saved yet.
    -   This is maddening, but this is how you get around it:
        1.  Switch to the Dependencies tab.
        2.  Add a dependency for the MSMQ instance. From there, it finds everything else by looking up the tree.
        3.  Click Apply to save the dependency.
        4.  Now switch back to the General tab.
        5.  Check the "Use Network Name for computer name" checkbox. This is what tells the application that Environment.MachineName should return the cluster name, not the cluster node's computer name.
        6.  Click Apply again.
        7.  Thank Microsoft with an eye-roll for forcing you to do this silly tab switching multi apply clicking dance.

5.  Repeat for your other distributors.

With your distributors installed, you can repeat pretty much the same procedure for a Timeout Manager, if you need one, and any Commander applications, if you have them. You may want to skip the Commander application for now, however. It's sometimes easier to get everything else installed first as a stable system that reacts to events but has no stimulus, and then add the Commaner application which will get the whole system in motion.

Again, try swapping the cluster back and forth, to make sure it can move freely between the cluster nodes.

## Setting up the Workers

From here it's all downhill. Set up your worker processes on both worker servers (not the cluster nodes!) as services, the same as you did for the distributors. Configure the workers' UnicastBusConfig sections to point to the distributor's data and control queues as described on the [Distributor Page](http://www.nservicebus.com/Distributor.aspx) under "Routing with the Distributor".

With your distributors running in the cluster and your worker processes coming online, you should see the Storage queues for each process start to fill up. The more worker threads you have configured, the more messages you can expect to see in each Storage queue.

While in development, your endpoint configurations probably don't have any @ symbols in them, in production you're going to need to change all of them to point to the Data Bus queue on the cluster, i.e. for application MyApp and logical queue MyQueue, your worker config will look like this:

```
<UnicastBusConfig
     DistributorControlAddress="MyApp.MyQueue.Control@MyCluster"
     DistributorDataAddress="MyApp.MyQueue@MyCluster">

     <MessageEndpointMappings>
          <add Messages="MyApp.Messages.MyMessage, MyApp.Messages"
               Endpoint="MyApp.SomeOtherQueue@MyCluster" />
     </MessageEndpointMappings>
</UnicastBusConfig>
```

## Conclusion

In this article I've shown how to set up a Windows Failover Cluster and two worker node servers to run a scalable, maintainable, and reliable NServiceBus application infrastructure.

-   Scaling up can be achieved by adjusting the number of threads on each worker process.
-   Scaling out can be achieved by standing up another server to run another worker process connected to the clustered distributor.
-   For maintenance, the worker processes on either worker server can be stopped for server maintenance or application updates, while worker processes on the other server continue to process messages. All clustered resources can be failed over to one node without disrupting the system, allowing message processing to continue while the other clustered node is available for updates.
-   Reliability is achieved by never requiring that any one component be completely shut down.

I hope this article is helpful for you; I know that it is far from perfect. Please let me know what problems you encounter and I will try to improve this guide to counter them. Good luck!
