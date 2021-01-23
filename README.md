# Project 0: Get started with Mininet and Topology Configurations

## Objectives

* Get familiar with the experiment environment: Mininiet, Topology configurations, controller, etc.
* Create your own circle topology
* Run example applications on the simple topology and learn how to read their performance numbers

## Tutorial: The line topology example

To start this tutorial, you will first need to get the [infrastructure setup](https://github.com/minlanyu/cs145-site/blob/master/infra.md)

Please enter in the project directory

```
$ cd projectX-YYY
```

Run `./pull_update.sh` to pull project updates (if any). You might need to merge conflicts manually: most of the time, you just need to accept incoming changes; reach to TF if it is hard to merge. This step also applies to all subsequent projects. 

### Create the line topology in Mininet

Let's first create the physical topology of the network in [Mininet](http://mininet.org/). The Mininet program automatically creates a virtual topology based on a *JSON configuration file*.
The JSON configuration file should:
- Define hosts and switches.
- Define the links, i.e., how hosts and switches connect with each other to form your topology.

As an example, we provide you with a line topology in the file `line/p4app_line.json`. 
The line topology has three hosts ("h1", "h2" and "h3") and three switches ("s1", "s2", and "s3"). There are five links: one connecting "h1" and "s1", "h2" and "s2", "h3" and "s3", "s1" and "s2", "s2" and "s3".

<img src="./figures/line_topo.png" width="500">

In this configuration file `line/p4app_line.json`, `topology` includes `assignment_strategy`, `links`, `hosts`, and `switches`. The `assignment_strategy` indicates all the switches run at layer 2 (`l2`). We put the three hosts in the `hosts` subfield, and put the three switches in the `switches` subfield. We also put the five links in the `links` subfield.

**Run Mininet with the topology**
```
$ sudo p4run --config topology/p4app_line.json
```
You will see the `mininet>` prompt if the command ran successfully.
You can choose different `json` files for differnet topologies.

**Verify your topology**
In the Mininet CLI, you can check out the nodes and links you create by running the following commands:

- `nodes`: list all the hosts and switches you just created.
- `net`: list all the links you just created. For example,

    ```
    h1 h1-eth0:t1-eth1
    # port 0 of host h1 and port 1 of switch t1 are connected.
    a1 lo:  a1-eth1:t1-eth3 a1-eth2:t2-eth3 a1-eth3:c1-eth1 a1-eth4:c2-eth1
    # port 1 of switch a1 and port 3 of switch t1 are connected
    # port 2 of switch a1 and port 3 of switch t2 are connected
    # port 3 of switch a1 and port 1 of switch c1 are connected
    # port 4 of switch a1 and port 1 of switch c2 are connected
    ```

- `links`: list all the links you just created.
- `help`: list all the supported commands.
- `exit`: quit the Mininet CLI.

For more information on how to use Mininet and other useful commands you can check out the [walkthough](http://mininet.org/walkthrough/) and [Mininet documentation](https://github.com/mininet/mininet/wiki/Documentation).


### Create networking software to run on the topology

There are two pieces of software that can control traffic in the topology. The first is a *controller program*, which is responsible of running routing algorithms, generating forwarding rules, and installing the rules into the tables at switches. The second is a *p4 program* that specifies packet processing logics at switches.  
For the line topology, we provide the controller program at `controller/controller_line.py` and the p4 program at `p4src/l2fwd.p4`. We do not need to touch the p4 program in this project. Basically, the p4 program defines a **dmac** table which maps the destination MAC address to the output port. 

Now let's delve more into the controller code. The controller implements the `route` function which installs entries in Table *dmac* to forward traffic.
Our controller file already implemented some small functions that use the `Topology` and `SimpleSwitchAPI` objects from the `p4utils library` (this library locates in `/home/p4/p4-tools/p4-utils/README.md`, and you can also check the library in this [online site](https://github.com/minlanyu/cs145-site/blob/master/p4-utils/README.md)). 
At a high level, the `route` function uses `table_add` function to insert forwarding rules. 
```
def table_add(self, table_name, action_name, match_keys, action_params=[], prio=None):
        "Add entry to a match table: table_add <table name> <action name> <match fields> => <action parameters> [priority]"
```
(You can find other functions and their definitions at [here](https://github.com/minlanyu/cs145-site/blob/master/p4-utils/p4utils/utils/runtime_API.py#L920), you can also check `/home/p4/p4-tools/p4-utils/p4utils/utils/runtime_API.py#L920` locally if you are curious).

For example, 
```
controller.table_add("dmac", "forward", ["00:00:0a:00:00:01"], ["1"])
```
This line adds a rule in the `dmac` table, with the `forward` action. The rule says that if you receive a packet whose destination MAC address `dmac` is `"00:00:0a:00:00:01"`, `forward` the packet to port `"1"`.

In Mininet, by default hosts get assigned MAC addresses using the following pattern: `00:00:<IP address to hex>`. For example if h1 IP's address were `10.0.1.5` the MAC address would be: `00:00:0a:00:01:05`. The default IP address of host hX is `10.0.0.X`.

To find out the port mappings for each switch, you can check the messages printed by when running the `p4run` command:
	
```
Switch port mapping:
s1:  1:h1	2:h2
```

You can also find out the port mappings using the `links` CLI command in the mininet terminal:

```
mininet> links
h1-eth0<->s1-eth1 (OK OK) 
h2-eth0<->s2-eth1 (OK OK) 
h3-eth0<->s3-eth1 (OK OK) 
s1-eth2<->s2-eth2 (OK OK) 
s2-eth3<->s3-eth2 (OK OK) 
```

**Run the controller**

Start another terminal, and run
```
$ python controller/controller_line.py
```

**Verify your controller**
If your controller installs routing rules successfully, hosts should be able to communicate with each other. 
To test the connectivity between all pairs of hosts, you can run the following commands in Mininet CLI 
- `pingall`: ping between any host pair
If `pingall` fails, you will see prompt stopped. (ctrl + c will kill the `pingall` command).

For debugging, you can also run:
- `h1 ping h2`: ping h2 from h1, to test the connectivity between two hosts
- `h1 <commands>`: run a command on host h1.

## Your Task: Build the Circle Topology

In this project, your task is to build the circle topology in `topology/p4app_circle.json` as shown in the following figure.
<img src="./figures/circle_topo.png" width="500">

You **only** need to write your own code in places marked with a ``TODO`` (ie, the `topology` field). 

Next, you need to write forwarding rules for the circle topology in `controller/controller_circle.json`. You **only** need to write your own code in places marked with a ``TODO`` (i.e., within the `route` function). 

### Testing
You can test your solution in the following steps:
1. Start the topology
	```
	$ sudo p4run --config topology/p4app_circle.json
	```
2. Run the controller
	```
	$ python controller/controller_circle.py
	```
3. We provide you with a testing script in `tests/test_circle_topo.py`. Run it and your network should pass all tests.
	```
	$ python tests/test_circle_topo.py
	```

## Running Applications on your network

Now that you are running a network in your own laptop. You can run real networked applications on these hosts and let them talk to each other. 
We provide three applications: Memcached, Iperf, and video streaming.
These applications will also be used in future projects.

### Video streaming

**Start a video streaming server at host `h1`**
```
$ ./apps/start_vid_server.sh h1 10.0.0.1 
```
This command starts a video streaming server at host `h1`, and the IP address of `h1` is `10.0.0.1`. 

**Open the terminal for host `h2` in the mininet terminal**
```
mininet> xterm h2
```
You will see another terminal (xterm) popped up, which belongs to host `h2`.

**Start the client for `h2` at xterm**

In the terminal popped up, type:
```
$ ./apps/start_vid_client.sh h2 10.0.0.1
```
This command opens a Chrome web browser on host `h2` which visits a video website served on `10.0.0.1`. If you ran your server on a host other than `h1`, then change `10.0.0.1` to that IP.
Try playing the video. You should see something like this:

<img src="./figures/video.png" width="600">

**Video streaming performance**

If you are curious about the intricacies of video streaming, you can try testing out how the performance of the video stream changes as you increase and decrease the link bandwidth.
You can set the link bandwidth in the topology configuration file as follows
```
"topology": {
    "assignment_strategy": "l2",
    "links": [["h1", "s1", {"bw": 1}], ["h2", "s2", {"bw": 1}], ["h3", "s3", {"bw": 1}], ["s1", "s2", {"bw": 1}], ["s2", "s3", {"bw": 1}]],
    ...
}
```
The `bw` field defines the link bandwidth, whose unit is Mbps.

How does the video bitrate change over time for 100 Kbps, 1 Mbps, 2 Mbps and 4 Mbps? When is bandwidth no longer a bottleneck?

### Memcached and Iperf

In this course project, we mainly use two applications, Memcached and Iperf. Memcached is an in-memory key-value store system, which distributes key-value pairs across different servers. Memcached mainly has two operations: `set` and `get`. Usually the key-value pair is a very short message, and thus for each operation, the system only generates a short TCP message, which makes the TCP flow short (less than 200 Byte). For more information, please refer to [Memcached website](http://memcached.org/). 

Iperf is a measurement tool for measuring IP network bandwidth. We usually run Iperf in two servers, and let those two servers to send packets as fast as possible, so that Iperf could measure the maximum bandwidth between the two servers. As a result, Iperf generates long and persistent flows, which is in contrast with Memcached. For more information, please refer to [Iperf website](https://iperf.fr/).

**Generate request trace**

We provide you with a trace generator which generate requests for Memcached and Iperf.
```
$ python ./apps/trace/generate_trace.py [hosts running memcached] [hosts running iperf] [trace length (sec)] [trace file name]
```
For example, if you want to run Memcached on host `h1-h3`, run Iperf on host `h1` and `h3`, and generate a trace for 60 seconds, you can type:
```
$ python ./apps/trace/generate_trace.py 1-3 1,3 60 ./apps/trace/test.trace
```

After generating the trace, you will find a file named `test.trace` in `apps/trace` directory. 

**Run Memcached and Iperf**

We provide you with an easy script to run Memcached and Iperf servers and clients on hosts:
```
$ sudo python ./apps/send_traffic.py ./apps/trace/test.trace 1-3 60
```
Then you will run your applications for 60 seconds. After finishing running, you will get the results, including the latency of memcached requests and the throughput of iperf requests.
```
start iperf and memcached servers
wait 1 sec for iperf and memcached servers to start
start iperf and memcached clients
wait for experiment to finish
stop everything
wait 10 sec to make log flushed
Average latency of Memcached Requests: 326.585716909 (us)
Average log(latency) of Memcached Requests: 1.37206216773
Average throughput of Iperf Traffic: 23454086.8092 (bps)
Average log(throughput) of Iperf Traffic: 6.24738717264
4.87532500491
```

**Check logs**

You can check the result files in `logs` directory. In this directory, you can find files like `hX_iperf.log` or `hX_mc.log`.
In files `hX_iperf.log`, each line represents the average throughput of one iperf request, which is issued by host `hX`.
In files `hX_mc.log`, each line represents the request latency of one memcached request, which is issued by host `hX`.

*Note*: if you are using Windows laptop, please be aware about that Windows and Linux handle newlines differently. Windows uses `\r\n` at the end of a line, while Linux uses `\n`. Therefore, please do not edit those files in your Windows host. If you want to use some text editor to edit the files in the VM through SSH or SFTP, be sure to set the correct newline symbol.

## Debugging and Troubleshooting

Here's a brief introduction on how to read switch logs. You can also check out more debugging and troubleshooting tips [here](debug.md).
**These debugging tips will be useful for all future projects. Please come back and revisit this in future projects**

Each p4 switch provides a log file for debugging. Those files are located in the `log` directory.
Within this directory, you can see files named `sX.log`, which indicates this file is the log file for switch `sX`.
The log file records all operations happen within this switch.
You only need to focus on two kinds of records: adding table entry and packet processing.

When you use your controller script to add table entry in the P4 switch, there will be some words in the log file:
```
[11:18:35.237] [bmv2] [T] [thread 15987] bm_table_add_entry
[11:18:35.237] [bmv2] [D] [thread 15987] Entry 0 added to table 'MyIngress.dmac'
[11:18:35.237] [bmv2] [D] [thread 15987] Dumping entry 0
Match key:
* hdr.ethernet.dstAddr: EXACT     00000a000001
Action entry: MyIngress.forward - 1
```
which means you add a table entry, whose key is the destination MAC address `00000a000001`, and the action is `forward(1)`.

After the switch processes a packet, there will also be some words in the log file:
```
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Processing packet received on port 2
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Parser 'parser': start
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Parser 'parser' entering state 'start'
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Extracting header 'ethernet'
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Parser state 'start' has no switch, going to default next state
[11:19:01.206] [bmv2] [T] [thread 15855] [14.0] [cxt 0] Bytes parsed: 14
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Parser 'parser': end
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Pipeline 'ingress': start
[11:19:01.206] [bmv2] [T] [thread 15855] [14.0] [cxt 0] Applying table 'MyIngress.dmac'
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Looking up key:
* hdr.ethernet.dstAddr: 00000a000001

[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Table 'MyIngress.dmac': hit with handle 0
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Dumping entry 0
Match key:
* hdr.ethernet.dstAddr: EXACT     00000a000001
Action entry: MyIngress.forward - 1,

[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Action entry is MyIngress.forward - 1,
[11:19:01.206] [bmv2] [T] [thread 15855] [14.0] [cxt 0] Action MyIngress.forward
[11:19:01.206] [bmv2] [T] [thread 15855] [14.0] [cxt 0] p4src/line_topo.p4(72) Primitive standard_metadata.egress_spec = egress_port
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Pipeline 'ingress': end
[11:19:01.206] [bmv2] [D] [thread 15855] [14.0] [cxt 0] Egress port is 1
[11:19:01.207] [bmv2] [D] [thread 15857] [14.0] [cxt 0] Pipeline 'egress': start
[11:19:01.207] [bmv2] [D] [thread 15857] [14.0] [cxt 0] Pipeline 'egress': end
[11:19:01.207] [bmv2] [D] [thread 15857] [14.0] [cxt 0] Deparser 'deparser': start
[11:19:01.207] [bmv2] [D] [thread 15857] [14.0] [cxt 0] Deparsing header 'ethernet'
[11:19:01.207] [bmv2] [D] [thread 15857] [14.0] [cxt 0] Deparser 'deparser': end
[11:19:01.207] [bmv2] [D] [thread 15860] [14.0] [cxt 0] Transmitting packet of size 74 out of port 1
```
which describes how the P4 switch processes this packet.

## Submission and Grading

### What to Submit

You are expected to submit the following files. Please make sure all files are in the root of your git branch.
- `topology/p4app_circle.json`. This file describes the topology you build.
- `controller/controller_circle.py`. This file contains how you insert forwarding rules into P4 switches.
- `report/report.md`. Please write the description of your work in `report/report.md` file (the `report` directory locates at the root directory of project 0). The description includes:
	- How do you write forwarding rules in the `controller/controller_circle.py` file, and why do those rules work to enable communications between each pair of hosts.
	- After running the applications, you can get the evaluation results. Please run both applications on all hosts, i.e., on host `h1-h3`. You will get average log(latency) of memcached requests, and average log(throughput) of iperf. Write these evaluation results in this file.

You are expected to use Github Classroom to submit your project. 
After completing a file, e.g., the `topology/p4app_circle.json` file, you can submit this file by using the following commands:
```
$ git commit topology/p4app_circle.json -m "COMMIT MESSAGE" # please use a reasonable commit message, especially if you are submitting code
$ git push origin master # push the code to the remote github repository
```

### Grading
The total grade for project 0 is 100 as follows:
- *40*: Pass the first three tests: topology tests
- *40*: Pass the fourth test: connectivity test
- *20*: for `report/report.md` file.
- *10*: Extra credit
- *deductions based on late policies*.

### Survey

Please fill up the survey when you finish your project.

[Survey link](https://docs.google.com/forms/d/e/1FAIpQLSdY_2e633IKgmNcb49bLaWxTk8hkZh4cH5QOG3EZgcVXiHgrA/viewform?usp=sf_link)
