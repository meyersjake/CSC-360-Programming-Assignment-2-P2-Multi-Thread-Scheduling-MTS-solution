# CSC-360-Programming-Assignment-2-P2-Multi-Thread-Scheduling-MTS-solution

Download Here: [CSC 360 Programming Assignment 2 (P2): Multi-Thread Scheduling (MTS) solution](https://jarviscodinghub.com/assignment/programming-assignment-2-p2-multi-thread-scheduling-mts-solution/)

For Custom/Original Work email jarviscodinghub@gmail.com/whatsapp +1(541)423-7793

In P0 (Simple Shell Interpreter, SSI) and P1 (Realistic Shell Interpreter, RSI), you have built a shell environment to
7 interact with the host operating system. Great job! But very soon you find that both SSI and RSI are missing one
8 of the key features in a real multi-process or multi-thread operating system: scheduling, i.e., all processes or threads
9 created by your RSI are still scheduled by the host operating system, not yours!
10 Interested in building a multi-thread scheduling system for yourself? In this assignment, you will learn how to
11 use the three programming constructs provided by the posix pthread library:
12 1. threads
13 2. mutexes
14 3. condition variables (convars)
15 to do so. Your goal is to construct a simulator of an automated control system for the railway track shown in Figure 1
16 (i.e., to emulate the scheduling of multiple threads sharing a common resource in a real operating system).
17 As shown in Figure 1, there are two stations (for high and low priority trains) on each side of the main track. At
18 each station, one or more trains get loaded with commodities. Each train in the simulation commences its loading
19 process at a common start time 0 of the simulation program. Some trains take more time to load, some less. After
20 a train is loaded, it patiently awaits permission to cross the main track, subject to the requirements specified in
21 Section 2.2. Most importantly, only one train can be on the main track at any given time. After a train finishes
22 crossing, it magically disappears. You will use threads to simulate the trains approaching the main track from two
23 different directions, and your program will schedule between them to meet the requirements in Section 2.2.
24 You will use C or C++ and the Linux workstation in ECS242 to implement and test your work.
25 2 Trains
26 Each train, which will be simulated by a thread, has the following attributes:
27 1. Direction:
28 • If the direction of a train is Westward, it starts from the East station and travels to the West station.
29 • If the direction of a train is Eastward, it starts from the West station and travels to the East station.
east−bound West East
high−priority
low−priority
east−bound
east−bound
main track
west−bound
high−priority
west−bound
low−priority
west−bound
Figure 1: The railway system under consideration.
1
30 2. Priority: The priority of the station from which it departs.
31 3. Loading Time: The amount of time that it takes to load it (with goods) before it is ready to depart.
32 4. Crossing Time: The amount of time that the train takes to cross the main track.
33 Loading time and crossing time are measured in 10ths of a second. These durations will be simulated by having
34 your threads, which represent trains, usleep() for the required amount of time.
35 2.1 Step 1: Reading the input file
36 Your program (mts) will accept two parameters on the command line:
37 1. The first parameter is the name of the input file containing the definitions of the trains.
38 2. The second parameter, an integer > 0, is the number of trains specified in the input file.
39 2.1.1 Input file format
40 The input files have a simple format. Each line contains the information about a single train, such that:
41 1. The first character specifies the direction of the train. It is one of the following four characters:
42 e, E, w, or W
43 e or E specify a train headed East (East-Bound): e represents an east-bound low-priority train, and E represents
44 an east-bound high-priority train;
45 w or W specify a train headed West (West-Bound): w presents a west-bound low-priority train, and W represents
46 a west-bound high-priority train.
47 2. A colon(:) immediately follows the direction of the train.
48 3. Immediately following is an integer that indicates the loading time of the train.
49 4. A comma(,) immediately follows the previous number.
50 5. Immediately following is an integer that indicates the crossing time of the train.
51 6. A newline (\n) ends the line.
52 You may assume that the file always contains data for at least the number of trains specified in the second
53 parameter. During our testing, the file specified on the command line will exist, and it will contain valid data.
54 2.1.2 An Example
55 The following file specifies three trains, two headed East and one headed West.
56 e:10,6
57 W:6,7
58 E:3,10
59 It implies the following list of trains:
Train No. Priority Direction Loading Time Crossing Time
0 low East 1.0s 0.6s
1 high West 0.6s 0.7s
2 high East 0.3s 1.0s
60
61 Note: Observe that Train 2 is actually the first to finish the loading process.
2
62 2.2 Step 2: Simulation Rules
63 The rules enforced by the automated control system are:
64 1. Only one train is on the main track at any given time.
65 2. Only loaded trains can cross the main track.
66 3. If there are multiple loaded trains, the one with the high priority crosses.
67 4. If two loaded trains have the same priority, then:
68 (a) If they are both traveling in the same direction, the train which finished loading last gets the clearance
69 to cross first. If they finished loading at the same time, the one appeared last in the input file gets the
70 clearance to cross first.
71 (b) If they are traveling in opposite directions, pick the train which will travel in the direction opposite of
72 which the last train to cross the main track traveled.
73 2.3 Step 3: Output
74 For the example, shown in Section 2.1.2, the correct output is:
75 Train 2 is ready to go East
76 Train 2 is ON the main track going East
77 Train 1 is ready to go West
78 Train 0 is ready to go East
79 Train 2 is OFF the main track after going East
80 Train 1 is ON the main track going West
81 Train 1 is OFF the main track after going West
82 Train 0 is ON the main track going East
83 Train 0 is OFF the main track after going East
84 You must:
85 1. print the arrival of each train at its departure point (after loading) using the format string:
86 “Train %2d is ready to go %4s”
87 2. print the crossing of each train using the format string:
88 “Train %2d is ON the main track going %4s”
89 3. print the arrival of each train (at its destination) using the format string:
90 “Train %2d is OFF the main track after going %4s”
91 where:
92 • there are only two possible values for direction: “East” and “West”
93 • trains have identifying numbers in the range of [0, 99]. The ID number of a train is specified implicitly in the
94 input file. The train specified in the first line of the input file has ID number 0.
95 • trains have loading and crossing times in the range of [1, 99].
96 2.4 Manual Pages
97 Be sure to study the man pages for the various functions to be used in the assignment. For example, the man page
98 for pthread create can be found by typing the command:
99 $ man pthread create
100 At the end of this assignment you should be familiar with the following functions:
101 1. File access functions:
102 (a) atoi 3
103 (b) fopen
104 (c) feof
105 (d) fgetc and fgets
106 (e) fclose
107 2. Thread creation functions:
108 (a) pthread create
109 (b) pthread exit
110 (c) pthread join
111 3. Mutex manipulation functions:
112 (a) pthread mutex init
113 (b) pthread mutex lock
114 (c) pthread mutex unlock
115 4. Condition variable manipulation functions:
116 (a) pthread cond init
117 (b) pthread cond wait
118 (c) pthread cond broadcast
119 (d) pthread cond signal
120 It is absolutely critical that you read the man pages, and attend the tutorials.
121 Your best source of information, as always, is the man pages.
122 For help with the posix interface (in general):
123 https://www.opengroup.org/onlinepubs/007908799/
124 For help with posix threads:
125 https://www.opengroup.org/onlinepubs/007908799/xsh/pthread.h.html
126 A good overview of pthread can be found at: https://www.llnl.gov/computing/tutorials/pthreads/
127 3 Submission: Deliverable 1 (Design Due: October 27, 2010)
128 You will write a design document which answers the following questions. It is recommended that you think through
129 the following questions very carefully before answering them.
130 Unlike P0 and P1, no amount of debugging will help after the basic design has been coded. Therefore, it is very
131 important to ensure that the basic design is correct. Answering the following questions haphazardly will basically
132 ensure that Deliverable 2 does not work.
133 So think about the following for a few days and then write down the answers.
134 1. How many threads are you going to use? Specify the work that you intend each thread to perform.
135 2. Do the threads work independently? Or, is there an overall “controller” thread?
136 3. How many mutexes are you going to use? Specify the operation that each mutex will guard.
137 4. Will the main thread be idle? If not, what will it be doing?
138 5. How are you going to represent stations (which are collections of loaded trains ready to depart)? That is, what
139 type of data structure will you use?
140 6. How are you going to ensure that data structures in your program will not be modified concurrently?
141 7. How many convars are you going to use? For each convar:
4
142 (a) Describe the condition that the convar will represent.
143 (b) Which mutex is associated with the convar? Why?
144 (c) What operation should be performed once pthread cond wait() has been unblocked and re-acquired the
145 mutex?
146 8. In 15 lines or less, briefly sketch the overall algorithm you will use. You may use sentences such as:
147 If train is loaded, get station mutex, put into queue, release station mutex.
148 The marker will not read beyond 15 lines.
Note: Please submit answers to the above on 8.5′′×11′′
149 paper in 10pt font, single spaced with 1” margins left,
150 right, top, and bottom. 2 pages maximum (cover page excluded), on October 27 at the lecture.
151 4 Bonus Features
152 Only a simple control system simulator with limited scheduling and synchronization features is required in this assign153 ment. However, students have the option to propose bonus features to include more scheduling and synchronization
154 functions (e.g., to address scheduling fairness issues).
155 If you want to design and implement a bonus feature, you should contact the course instructor for permission
156 before the due date of Deliverable 1, and clearly indicate the feature in the submission of Deliverable 2. The credit
157 for the correctly implemented bonus feature will not exceed 25% of the full marks for this assignment.
158 5 Submission: Deliverable 2 (Code Due: November 12, 2010)
159 The code is submitted through connex. The tutorial instructor will give the detailed instruction in the tutorial.
160 5.1 Submission Requirements
161 Your submission will be marked by an automated script. The script (which isn’t very smart) makes certain assump162 tions about how you have packaged your assignment submission. We list these assumptions so that your submission
163 can be marked thus, in a timely, convenient, and hassle-free manner.
164 1. The name of the submission file must be p2.tar.gz
165 2. p2.tar.gz must contain all your files in a directory named p2
166 3. Inside the directory p2, there must be a Makefile.
167 4. Invoking make on it must result in an executable named mts being built, without user intervention.
168 5. You may not submit the assignment with a compiled executable and/or object (.o) files; the script will delete
169 them before invoking make.
170 Note:
171 1. The script will give a time quota of 1 minute for your program to run on a given input. This time quota is
172 given so that non-terminating programs can be killed.
173 Since your program simulates train crossing delays in 10ths of a second, this should not be an issue, at all.
174 2. Follow the output rules specified in the assignment specification, so that the script can tally the output produced
175 by your program against text files containing the correct answer.
176 3. The markers will read your C code after the script has run to ensure that the pthread library is used as
177 required.
178 6 Plagiarism
179 This assignment is to be done individually. You are encouraged to discuss the design of the solution with your
180 classmates, but each student must implement their own assignment. The markers will submit your code to an
181 automated plagiarism detection service.

