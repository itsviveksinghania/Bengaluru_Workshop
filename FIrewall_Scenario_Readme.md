# Scenario:- Firewall Problem Detection

### Problem Statement
```
Client is not able to connect to the "ec2-44-203-168-246.compute-1.amazonaws.com" kafka broker on Port No 9091. 
Client is outside the Kafka Network.
```

### Additional Information
```
Q. Can Client Connect to other port that has kafka broker running? eg: 9092 on same HostName
A. Yes

Q. What Command Client is Using, to connect to the broker? Or,What Client wants to do?
A. Client wants to Read from the topic. 

Using this command:
kafka-console-consumer --bootstrap-server  ec2-44-203-168-246.compute-1.amazonaws.com:9091 --consumer.config client.properties --topic t_vivek --from-beginning --group vivek

Q. What the Error message Client is getting?
A. [2023-06-23 18:13:30,779] WARN [Consumer clientId=console-consumer, groupId=vivek] Bootstrap broker ec2-44-203-168-246.compute-1.amazonaws.com:9091 (id: -1 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
```

### Possible Failure Scenario
Authentication
```
This cannot be the Reason because Error message will contain Handshake Failed Error.
```

Authorization
```
This cannot be the Reason because Error message will contain Type of Authorization Exception Error.
```

Topic Not Present
```
This cannot be the Reason because Error message will contain UNKNOWN Topic Error.
```

Connecting to the wrong broker
```
This is not Valid Failure Scenario because, 
we can connect to any broker and can Consume to the topic leader, 
as MetaData contains which broker has the leader for that topic.
```

Broker Down
```
This error message could indeed suggest that the broker might be down or unreachable.

While this is a strong indicator that there might be a problem with the broker, it's not definitive proof.
There could be other network or configuration issues at play.

1. Check the services running
systemctl list-units --type=service --state=running
Look for Kafka server

2. Reconfirm if Broker is running by using this Command
sudo systemctl status confluent-server

3. Check if Zookeeper is running by using this Command
sudo systemctl status confluent-zookeeper

4. Check the logs of broker
From this command
sudo systemctl status confluent-server

Get the Main PID e.g: 114462 then, 
ps aux | grep 114462 
this give the logs location Ex -Dkafka.logs.dir=/var/log/kafka
and the server.properties location

Then, Check the logs for any ERROR

5. Check the Ports Kafka is running
sudo netstat -tuln

6. Check the ports Open and Listen
Try to Connect to the port from Broker
telnet localhost 9091

Try to Connect to the port from outside kafka network
telnet ec2-44-203-168-246.compute-1.amazonaws.com 9091


7. Check for listeners and advertised listeners in server.properties file
In step 5 we got server.properties file location. cd to that location and check the server.properties file
vim server.properties
```

Resource Problem from Broker end
```
Make Sure CPU RAM and DISK Usage are Optimal. Commands are:
DISK df -h
RAM free -h
CPU top
```

Client Resource Problem
```
Try to run the 
kafka-console-consumer \
--bootstrap-server  ec2-44-203-168-246.compute-1.amazonaws.com:9091 \
--consumer.config client.properties \
--topic t_vivek \
--from-beginning \
--group vivek

this command using your machine. Then try to run the command inside the broker. 
If Both gets the Desired result, Then it's Client Resource Problem

If it works inside the broker, but not in your machine.
Then we can say it's Firewall issue.

PS: Make sure KeyStore and trustStore are Present.
```

Network connnection
```
If after all checklist we can possibly say it's Firewall issue.
Port is working fine (Open & Listen) but cannot be accessed by outside of network
```
