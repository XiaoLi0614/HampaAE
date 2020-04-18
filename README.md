# HampaAE
Solver-aided Recency-Aware Replicated Objects (CAV2020)

**Abstract:** Replication is a common technique to build reliable and scalable systems. Traditional strong consistency maintains the same total order of operations across replicas. This total order is the source of multiple desirable consistency properties: integrity, convergence and recency. However, maintaining the total order has proven to inhibit availability and performance. Weaker notions exhibit responsiveness and scalability; however, they forfeit the total order and hence its favorable properties. This project revives these properties with as little coordination as possible. It presents a tool called Hampa that given a sequential object with the declaration of its integrity and recency requirements, automatically synthesizes a correct-by-construction replicated object that simultaneously guarantees the three properties. It features a relational object specification language and a syntax-directed analysis that infers optimum staleness
bounds. Further, this paper defines coordination-avoidance conditions and the operational semantics of replicated systems that provably guarantees the three properties. It characterizes the computational power and presents a protocol for recency-aware objects. Hampa uses automatic solvers statically and embeds them in the runtime to dynamically decide the validity of coordination-avoidance conditions. The experiments show that recency-aware objects reduce coordination and response time.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes before you start.

### Prerequisites

1. VirtualBox installed.

2. Download the virtualbox image from :
```
https://drive.google.com/file/d/1MrH-1O0AYLx_z4WZVKLHs6a6UoSNpGP0/view?usp=sharing
```


3. Import the virtual machine image on your local machine.

```
vboxmanage import HampaAE.ova
```
## Road Map

### Main paper

Fig.4 ->  **2. Bound inference and optimization**

Fig.7 ->  **3. The Recency-Aware protocol**

Fig.8 -> **5. Baseline: Sequential Object**

### Appendix

2 Bound Inference Example -> **2. Bound inference and optimization**

4 Coordination Conditions -> **1. Static analysis test for conflict and dependency relations**

5 Protocol  -> **3. The Recency-Aware protocol**


## Running the test

### 1. Static analysis test for conflict and dependency relations

The first step is to conduct static analysis tests on the give use-cases (Bank Account and Movie Reservation). We have already translate the use-cases from our relational language (appendix p3-p4) to corresponding AST node specifications in java.

Go to the directory: ```/home/user/CoordinationSynthesis/static_analysis```

```
cd /home/user/CoordinationSynthesis/static_analysis
```
Run the bash file for bank use-case: ```BankStaticAnalysis.sh```

```
./BankStaticAnalysis.sh 
```
You should see three tables, which are the conflict and dependency tables we claimed in the **appendix P33**.

Run the bash file for movie use-case: ```MovieStaticAnalysis.sh``` 

```
./MovieStaticAnalysis.sh 
```
You should see three tables, which are the conflict and dependency tables we claimed in the appendix P33.

### 2. Bound inference and optimization

The second step is to input the bound you want on each method. The bound inference will automatically output the optimal bound on the whole object state. This optimal bound makes you buffer more and boosts performance. 

To test the bound inference, go to the directory ```/home/user/CoordinationSynthesis/bound_inference```

```
cd /home/user/CoordinationSynthesis/bound_inference
```

Run the bash file ```BoundInference.sh```

```
./BoundInference.sh
```

The integer arguments are the bound for ```querySpace```, ```queryReservations``` and ```querySpaces``` respectively in the following command ( the content of ```BoundInference.sh```).

```
java -jar /home/user/CoordinationSynthesis/bound_inference/CVCAutomation_Xiao_jar/CVCAutomation_Xiao.jar 3 4 6
```
Join opeation for relations involves multiplication in our bound constrain deviation process, which falls out of the scope of theory of linear and integer programming. Please notice that arbitary large input arguments are not supported at this time.

### 3. The Recency-Aware protocol

The third step is to take the first and second step's results as input to run the recency-aware protocol.

**3.1 Bank account use-case:**

The callset of bank use-case are under this directory: ```/home/user/CoordinationSynthesis/etc```

There are four callsets. Each contains 125 calls, 500 calls in total: ```calls-bank-4-125_1, calls-bank-4-125_2, calls-bank-4-125_3, calls-bank-4-125_4```

All the calls are randomly produced and have the distribution for three meothods: deposit/withdraw/getbalance -> 75%, 25%, 5%

The arguments of the calls are randomly produced between 10 to 20.

To run the experiments, go to the directory and select the bound you wish to test: ```/home/user/CoordinationSynthesis/updated_run_movie/block/4/125```

We have tested the run-time performance of different bounds in their corresponding subdirectory: ```21, 40, 60, 80, 100, 120, 140, 160, 180, 200```. 

The reason to select 21 as the lowest recency bound is that we need to gurantee safety and liveness property at the same time. The maximum weight of a single call is 20. If a bound samller than 21 is set, it is possible that a call with larger weight stucks in the blocking queue and never gets executed.  

Run the runner.sh file under the bound directory.

```
./runner.sh
```
Copy the output and past back to command line. This command will run 4 processes at the same time to mimic 4 replicas communicating with each other.

```
./run0-block-4-125.sh & ./run1-block-4-125.sh & ./run2-block-4-125.sh & ./run3-block-4-125.sh
```
Wait for about 5 minutes for all the processes to complete.

Open the produced file in editor to collect results: ```0.txt, 1.txt, 2.txt, 3.txt ```

**3.2 Movie reservation use-case:**

The callset of movie use-case are under this directory: ```/home/user/CoordinationSynthesis/etc```

There are four callsets. Each contains 125 calls, 500 calls in total: ```calls-movie-4-125_1, calls-movie-4-125_2, calls-movie-4-125_3, calls-movie-4-125_4```

All the calls are randomly produced and have the distribution for five meothods: cancelBook/book/query(include quer, querySpace, queryReservations and querySpaces)/specialReserve/increaseSpace ->4%/6%/5%/40%/45%

The user ID of the book and cancel are randomly selected from 0 to 100. 

To run the experiments, go to the directory and select the bound you wish to test: ```/home/user/CoordinationSynthesis/updated_run_bank/block/4/125```

We have tested the run-time performance of different bounds in their corresponding subdirectory: ```2, 3, 4, 8, 12, 16, 20```. 

The reason to select 2 as the lowest recency bound is the same as the above bank account use-case. The maximum weight of a single call is 2.  

Run the 4 bash files under the bound directory as the following command. This command will run 4 processes at the same time to mimic 4 replicas communicating with each other.

```
./run0-block-4-125.sh & ./run1-block-4-125.sh & ./run2-block-4-125.sh & ./run3-block-4-125.sh
```
Wait for about 5 minutes for all the processes to complete.

Open the produced file in editor to collect results: ```0.txt, 1.txt, 2.txt, 3.txt ```

**3.3 Analysis the result**

Here are some tips to read that result:

1. The results recorded in the ```0.txt, 1.txt, 2.txt, 3.txt ``` have redundancy. You only need the response time for each method in the use-case and the RB, TOB and Ack number to reproduce Fig.7. and Fig.8. Here the ```Ack number``` in the result text file is exactly the ```P2P number``` in Fig.7. (a) and (c).

2. In order to calculate the ratio in Fig.7. (a) and (c),  you need to divid the message number by the base-line recency bound situation. 

In the bank account use-case, the baseline recency bound situation is when the recency bound is 21. 

In the movie use-case, the baseline recency bound situation is when the recency bound is 2. But for the TOB ratio, we are comparing the number of TOB messages sent with the situation where there is no buffer(the blocking protocol in the related work: Hamsaz). In this case, all the method calls in the conflict cover(book and sReserve) will go through TOB. To compute the TOB ratio for movie use-case, divide the TOB number from the experiment results by the the sum of book method number and sReserve number.  

3. We add different query methods(```querySpace```, ```querySpaces``` and ```queryReservations```) in the movie use-case to make it more consistent with other part of our paper. Just treat them as ```query``` will be fine.

(Optional) We provid python files to help with calculating the average from 4 replica's results. The following command will produce ```collect_usecase_channel_recencybound.txt ``` under the ```/home/user/CoordinationSynthesis/statistic``` folder. 

Go to the project's root directory: ```/home/user/CoordinationSynthesis```

Run the python file. The arguments are use-case name, , channel name(```block``` or ```rsm```), number of replicas, callset size and recency bound.

For bank usecase, see the example below.

```
python file_parse.py bank block 4 125 21 &>> ./statistic/collect_bank_block_21.txt
```

For movie usecase, see the example below.

```
python file_parse.py movie block 4 125 2 &>> ./statistic/collect_movie_block_2.txt
```
The output file is stored at the address you specified, eg: ```./statistic/collect_bank_block_21.txt``` from above command.


### 4. Dynamic check

All the dynamic check files are stored at the same directory with other output files. For example, for movie use-case with bound 2, the dynamic check files are under directory: ```/home/user/CoordinationSynthesis/updated_run_movie/block/4/125/2_0/movie```

The dynamic check files names looks like: ```DynamicTest_1_41.cvc4```

The first number represent the replica number, which is from 0 to 3. The second number is the identifer number, which indicats the order of the dynamic check regarding this replica only.

### 5. Baseline: Sequential Object

For the baseline performance mentioned in our paper, please go through the instructions for **The Recency-Aware Protocol**.

For bank use-case: do the experiments under directory: ```/home/user/CoordinationSynthesis/updated_run_bank/rsm/4/125```

For movie use-case: do the experiments under directory: ```/home/user/CoordinationSynthesis/updated_run_movie/rsm/4/125```

Note that there is only one subdirectory ```/0_0```. Because for a sequential object, there is no recency bound on it.

The baseline recency bound is the same as we mentioned in **The Recency-Aware Protocol**. To compare with them, go to the directory ```/home/user/CoordinationSynthesis/updated_run_bank/block/4/125/21``` and
```/home/user/CoordinationSynthesis/updated_run_movie/block/4/125/2``` to find the experiment results.

## Deployment

### Before running

Before run any instructions from **The Recency-Aware Protocol** and **Baseline: Sequential Object**, make sure there is no remaining alive java process from previous test. Otherwise the ports are occupied.

First check the pid of all processes:

```
ps aux |grep java
```

If you find any process name begins with ```java -jar```, kill the process.

```
kill pid_of_previous_java_process
```

Clean all the previously produced text files and CVC4 file by the following command

```
rm *.txt
rm -R ./bank
rm -R ./movie
```

### During running process

If you observe any abnormal result for a specific setting, please re-run the experiment under the same setting again later. Preferabley kill some unnecessary processes and release some resource for the experiment. We observe that on the virtual machie, the resources available at that moment may have huge impact on the result.

### Experiment results obtained from the virtual machine

We are able to get the same tendency or pattern as we reported in the paper on the virtual machine. You can find the experiment results obtained from the virtual machine(the same one as you get) in the directory ```/home/user/AEResult``` or in this repo. 

The absolute value for response time is not the same as we reported in the paper. Because our reported numbers are obtained from a more powerful cluster. In order to accord with the offline working requirement for CAV artifact evaluation, we use 4 processes to minic 4 nodes on the cluster and make them communicate through localhost address.

## Built With

* [APPIA](http://appia.di.fc.ul.pt/wiki/index.php?title=Main_Page) - A Java library of basic communication abstractions used
