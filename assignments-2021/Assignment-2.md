# CS6650 Spring 2021  Assignment 2

## Scaling the Giant Tigle

## Overview
A old decrepit supermarket chain has just had a huge injection of cash from a mystery multi-billionaire and is embarking on a rapid acquisition strategy to buy up new stores nationwide. 
These new stores need to be integrated into their existing business systems. They hire your development business to help them undertake this massive exercise.

## Assignment 2: Building the Server
In Assignment 2, we'll build the server and database. This will give us the foundation to start thinking about more complex features in assignments 3 and 4. 

The server API is specified using [Swagger](https://app.swaggerhub.com/apis/gortonator/GianTigle/1.0.0#). You should have implemented stubs for the API in your servlet in assignment 1. 
 
Next we need to design a database schema and deploy this to your MySQL RDS instance. The database must store each individual purchase request for subsequent processing in later assignments. 
You can choose another database if you like, but just be aware you may be on your own!

When creating your instance, don't choose backup or mirroring options - this will save $$s.Turn on monitor though - it will be useful. 

Also - see the discussion at the end of this assignment about Burst Mode - tl;dr choosing 'Provisioned SSD' is probably wise. 
You can read more about [burst mode here](https://aws.amazon.com/blogs/database/understanding-burst-vs-baseline-performance-with-amazon-rds-and-gp2/) and 

Think carefully about the schema design as you need to support an insert-heavy workload. 

Also thnk carefully about indexing as inserts are slower when indexes exist on a table. 
You can read all about SQL indexes [here](https://www.tutorialspoint.com/mysql/mysql-indexes.htm).

A simple servlet example for accessing MySQL is [here](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-usagenotes-glassfish-config-servlet.html) and Lab 7 expands on this.

You then need build the servlet business logic to implement this API. The API should:

1. Accept the parameters for each operations as per the specification
1. Do basic parameter validation, and return a 4XX response code and error message if invalid values/formats supplied
1. If the request is valid, store the request payload in the database.
1. Construct the correct response message and return a 200/201 response code 

Test the servlet API with [POSTMAN](https://www.getpostman.com/downloads/) or an equivalent HTTP testing tools.

Make sure you can load the resulting .war file onto your EC2 free tier instance you have created and configured in lab 1 and call the API successfully.

## Modify the Client 
Business is improving at the Tigle! So we want to modify the client to send 300 purchase (POST) requests per hour instead of 60. This will generate a few more requests :)

## Performance Testing
As in assignment 1, we want to test your new server/database with our load generating client. Test with {32, 64, 128, 256} client threads and report the outputs for each.

You may find you get database deadlocks. You will need to find a way to work around these through SQL/schema changes or request retries. Some useful advice on MYSQL deadlocks is [here](https://dev.mysql.com/doc/refman/8.0/en/innodb-deadlocks.html).

Your tests should successfully execute every request and store each purchase in the database.

Also don't forget the information below about Burst Mode. This may be influential in your testing :)

## Load Balancing
The previous section probably has a bottleneck in the single server instance. So let's try and add capacity to our system and see what happens,

Set up [AWS Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/features/?nc=sn&loc=2) using either _Application_ load balancers. Enable load balancing with 4 free tier EC2 instances and see what effect this has on your performance.  

You will need to create scripts to automatically start tomcat on instance boot. [Here's an example for AWS Linux 2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html), and for AWS Linux 1 look [here](https://medium.com/@shrunk7byadagi/automatically-start-tomcat-on-instance-startup-reboot-in-amazon-ec2-ubuntu-instance-33849a9d9090).

Depending on your data model, you may find you free tier RDS server becomes a bottleneck. If so then allocate a more powerful RDS instance and see what effect it has. Just watch your costs.

Again test with {32, 64, 128, 256} clients and compare your results again the ones from the previous section with a single server.

## Submission Requirements
Submit your work to Canvas  Assignment 2 as a pdf document. The document should contain:

1. The URL for your git repo. 
1. A 1-2 page description of your server design. Include major classes, packages, relationships, whatever you need to convey concisely how your client works
1. Single Server Tests - run your client with 32, 64, 128 and 256 store threads, and default values elsewhere. Include the output window of each run in your submission (showing the wall time and performance stats) and plot a simple chart showing the throughput and mean response by the number of threads
1. Load Balanced Server Tests - run the client as above, showing the output window for each run. Also generate a plot of throughput and mean response time against number of threads.

## Grading:
1. Server implementation working (15 points)
1. Server design description (5 points) - clarity of description, good design practices used
1. Single Server Tests - (10 points) - 1 point per run output, 1 point for the chart, 5 points for sensible results. 
1. Load Balanced Tests - (10 points) - 1 point per run, 1 point for the chart. 5 points for sensible results. 
1. Bonus  - (3 points) - A successful test run with 512 clients as max threads.

# Deadline: March 10th 11.59pm

[Back to Course Home Page](https://gortonator.github.io/bsds-6650/)

# Additional Notes
## Burst Mode
You may experience extreme slowdowns after bursting your database with a heavy number of requests because of something known as I/O credit balance AKA Burst Balance.

Thus is explained [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html#EBSVolumeTypes_gp2).

To sum up: If you set your RDS instance's "Storage Type" to "General Purpose" as the RDS tutorial from lab 1 taught you to do, then your RDS instance's ability to handle large loads is limited. 

General Purpose SSD storage has a limited amount of "I/O burst credits", which are credits allocated to your storage that depletes when your RDS instance experiences extremely large I/O loads. Once these burst credits are depleted, your RDS instance will return to "baseline performance", which is dependent on how large your storage is (generally 3 I/O operations per second (IOPS) PER gigabyte of storage).

So, if you configured your RDS instance to "General Purpose (SSD)" with a size of 20GB, then your IOPS will reduce to 60 IOPS once your "burst balance" has been depleted.

You may notice this problem if your RDS instance slows down after sustaining high IOPS levels for prolonged periods of time. 

So how do you fix this? Well, you can modify your "Storage Type" to "Provisioned SSD" instead of "General Purpose SSD". With Provisioned SSD, you can achieve consistent performance by setting the IOPS to what ever you desire. 2,500 IOPS? 3,000 IOPS? You choose :)

You can view your Read/Write IOPS AND Burst Balance under the "Monitoring" tab (if you turned it on) of your AWS RDS Console when viewing your database instance. (Burst Balance is on page 3 of this tab). If your Burst Balance goes to 0%, then expect slow performance.

You may also start an entirely new RDS instance with a fresh set of burst credits for fimal testing.

Finally - effect on costs not obvious - so be careful!

[Back to Course Home Page](https://gortonator.github.io/bsds-6650/)