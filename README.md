## What is MYPROXY

 - A proxy application that provides: 
    1. management on backend MYSQL databse tables 
    2. shardings on these tables
 - User may access backend tables through myproxy by normal MYSQL client tools 


## Features
 - supports native MYSQL server/client communication protocols v4.1
 - supports 10k+ clients concurrency 
 - supports dynamic scalability of MYSQL databases
 - supports the commonly used SQLs
 - supports SQL proxy functionalities, for example, ORACLE SQL -> MYSQL SQL
 - supports 2 sharding modes: horizontal and vertical


## Structure
 ![Alt text](https://github.com/oun111/images/blob/master/myproxy_structure.png)

Here the diagram consist of 3 parts:
 - MYSQL clients: sending requests and fetching datas from ***myproxy***
 - MYPROXY: 
    1. accepting and processing client requests
    2. forwarding and routing them to backend MYSQL servers
    3. fetching responses from backend and forwarding them to clients
 - MYSQL servers: storing real datas and processing requests from ***myproxy***


## Process Flow

### Case A:

 ![Alt text](https://github.com/oun111/images/blob/master/myproxy_process_flow_A.png)
 
 In this case, the client A sends "local" commands that need not to be processed at backend, such as login, show XXX, desc XXX, use XXX etc, ***myproxy*** executes them and returns to client immediately.
 
### Case B:

 ![Alt text](https://github.com/oun111/images/blob/master/myproxy_process_flow_B.png)
 
 - In this case, client B sends a "query mode" request, task 1 in ***myproxy*** will do:
    1. parsing the SQL into syntax tree
    2. hooking to modify the tree
    3. fetching route informations
    4. calculating routes
    5. forwarding request to every destination MYSQL server(s)
    
 - on the other hand, the task 2 or(and) task X will do:
    1. reading the response stream from MYSQL server(s)
    2. assabling streams into MYSQL response packets
    3. if there're more than 1 MYSQL server(s), caching responses
    
 - at last, task X+1 will merge responses in cache and returning to client
 
### Case C:

 ![Alt text](https://github.com/oun111/images/blob/master/myproxy_process_flow_C.png)
 
 - In this case, client C sends a "prepare" request in "prepare" mode , task 1 in ***myproxy*** will do:
    1. parsing the SQL into syntax tree
    2. hooking to modify the tree
    3. collecting routing informations
    4. forwarding request to all MYSQL server(s) at backend
    
 - on the other hand, the task 2 or(and) task X will do:
    1. reading the response stream from MYSQL server(s)
    2. assabling streams into MYSQL response packets
    
 - at last, task X+1 will filter the responses and returning to client
 
### Case D:
 
 ![Alt text](https://github.com/oun111/images/blob/master/myproxy_process_flow_D.png)
 
 - In this case, client C sends a "execute" request in "prepare" mode, task 1 in ***myproxy*** will do:
    1. calculating routes by infos from Case C
    2. forwarding request to every destination MYSQL server(s)
    
 - on the other hand, the task 2 or(and) task X will do:
    1. reading the response stream from MYSQL server(s)
    2. assabling streams into MYSQL response packets
    3. if there're more than 1 MYSQL server(s), caching responses
    
 - at last, task X+1 will merge responses in cache and returning to client


## Shardings
 - In my thought, ***sharding*** is how a database proxy organises table datas of real database at its backend. In ***myproxy***, there're 2 kinds of shardings:
    1. `Vertical sharding` is the ability of managing lots of unique tables in different database servers, see diagram below:
    
  ![Alt text](https://github.com/oun111/images/blob/master/myproxy_sharding_v.png)

   * in this case, tables are located completely in a unique server, table A B C in server 0, and table D and E in server 1
   * while requesting for table B, ***myproxy*** will calculate and then route only to database 0, while for table D, the path is to database 1
   
   2. `horizontal sharding` is to manage tables that are scattered partially in many database servers, see diagram below:
   
  ![Alt text](https://github.com/oun111/images/blob/master/myproxy_sharding_h.png)

   * in this case, table A is scattered into 3 parts and locates in 3 servers
   * while requesting for table A, ***myproxy*** will route to these 3 servers respectively for all parts of data


## Dynamic Scalability
  The `Dynamic Scalability` for ***myproxy*** is that at backend table(s) may be added into or be removed from without restarting ***myproxy*** which will update its backend configs by sending a `SIGUSER2` by the admin manually, see diagram below:
  
  ![Alt text](https://github.com/oun111/images/blob/master/myproxy_dynamic_scalability_A.png)
  
  * in case 1, table A is added into backend DB 0, in order to notify ***myproxy*** the admin may send a `SIGUSER2` to it
  * in case 2, table B is removed from backend DB 1, the admin should send a `SIGUSER2` to force a backend config update
  

## SQL Proxy


## Cache layers


## The hook modules


## HOWTO

 * start up ***myproxy***
 * access it just as you are accessing a real MYSQL server like this:
 
 ![Alt text](https://github.com/oun111/images/blob/master/myproxy_screen.jpg)
