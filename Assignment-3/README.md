Assignment 3 - Local Hadoop Streaming and AWS
---------------------------------------

All the tasks should be done using Hadoop Streaming (Hadoop version 2.x) and Python.

You will execute all the tasks TWICE: first, you will run them locally
(Local Hadoop or Cloudera VM) to test whether the logic of your job is
correct, and then, you will run them on AWS over a bigger data set
(using the command line interface).

1) Local Hadoop:
- You will use the following input data:
    Taxi Data for two days: https://www.mediafire.com/folder/3m9a39g8n0jyd/Taxi_Data_-_Assignment_3
    Vehicle Data: http://www.mediafire.com/view/6wziuzg5983q9oq/licenses.csv

- Use a single reducer

- Submit a .zip file with the following structure:
    * One directory per task, named "taskX", where X is the number of the task.
    * If a task has a sub-task, use "taskX-Y", where Y identifies the sub-task.
      E.g.: "task2-a" refers to sub-task "a" of Task 2.
    * Inside each directory, include:
      **The Python scripts used by your map-reduce job
        - Name your map script as "map.py", and your reduce script as "reduce.py"
        - If you need a combiner, use "combine.py" as the name of the script
      ** The output directory generated by Hadoop (use the naming
      convention specified below)
    * DO NOT submit the input data

2) AWS:
- You will use the following input data:
    Taxi Data for one week: https://www.mediafire.com/folder/1zaqzcf722loc/taxi_data_aug_week1
    Vehicle Data: http://www.mediafire.com/view/6wziuzg5983q9oq/licenses.csv

- Use two reducers, unless noted otherwise
- Your output files should reside in an S3 PUBLIC bucket (see
https://aws.amazon.com/articles/5050).  You should include links to
all output files in a text file named aws.txt with the following structure:
    
Task 1
link 1
link 2

Task 2 (a)
link 1
...

- DO NOT submit links to the input data
- DO NOT submit links to your Python scripts -- they should be the same as the ones used for the Local Hadoop execution

===================================================================
Task 1
  - Write a map-reduce job that joins the 'trips' and 'fare' data (taxi data).
  - Note that the 'fares' and 'trips' data share 4 attributes: medallion, hack_license, vendor_id, pickup_datetime.
  - The join MUST BE a reduce-side inner join.
  - Output: A key-value pair per line, where
    
    key: medallion, hack_license, vendor_id, pickup_datetime
    value: remaining attributes of 'trips' data in their original order, and  the remaining attributes of 'fare' data in their original order

    You should respect this ordering!

  - The output directory produced by Hadoop should be named TripFareJoin.

Task 2
Write map-reduce jobs for each of the following sub-tasks, using the output of Task 1 (joined data) as input for all these sub-tasks:

    a) Find the distribution of fare amounts (fare_amount), i.e., for each amount A, the number of trips that cost A.
       Output: A key-value pair per line, where the key is the amount, and the value is the number of trips.
       The output directory produced by Hadoop should be named FareAmounts.
    b) Find the number of trips that cost less or equal than $10 (total_amount).
       Output: The number of trips.
       The output directory produced by Hadoop should be named TripAmount.
    c) Find the distribution of the number of passengers, i.e., for each number of passengers A, the number of trips that had A passengers.
       Output: A key-value pair per line, where the key is the number of passengers, and the value is the number of trips.
       The output directory produced by Hadoop should be named NumberPassengers.
    d) Find the total revenue (for all taxis) per day (from pickup_datetime). The revenue should include the fare amount, tips, tolls, surcharges.
       Output: A key-value pair per line, where the key is the day (YYYY-MM-DD), and the value is the fare amount, surcharges, tips and tolls, in this order.
       The values in the output must have a precision of two decimal digits, e.g., 3.02245 should be represented as 3.02.
       The output directory produced by Hadoop should be named TotalRevenue.
    e) Find the total number of trips for each taxi (medallion).
       Output: A key-value pair per line, where the key is the medallion, and the value is the number of trips.
       The output directory produced by Hadoop should be named MedallionTrips.
    f) Find the number of different taxis (medallion) used by each driver (license).
       Output: A key-value pair per line, where the key is the driver, and the value is the number of different taxis used by that driver.
       The output directory produced by Hadoop should be named UniqueTaxis.

Task 3
  - Write a map-reduce job that joins the output from Task 1 with the vehicle data.
  - Note that they both share the medallion attribute.
  - The join MUST BE a reduce-side inner join.
  - Output: A key-value pair per line, where
    
    key: medallion
    value: remaining attributes of Task 1 output (including the remaining keys) in their original order + remaining attributes of vehicle data in their original order

    You should respect this ordering!

  - The output directory produced by Hadoop should be named VehicleJoin.

Task 4
Create map-reduce jobs for the following sub-tasks, using the output from Task 3 as input.
    
Note: In the vehicle data, you may find attributes with commas in the value, so splitting the line by the comma character may not work when reading the attributes (you may end up slitting one attribute in two or more). You can use the csv module (https://docs.python.org/2/library/csv.html) to parse each line; since this module assumes a file as input (not a string), you will need to use StringIO (https://docs.python.org/2/library/stringio.html) as well. Example:

    csv_file = StringIO.StringIO(line)
    csv_reader = csv.reader(csv_file)
    for record in csv_reader:
        # record is a list containing all the attributes

    a) Compare trips based on vehicle_type (WAV, HYB, CNG, LV1, DSE, NRML).
       Output: A key-value pair per line, where the key is the vehicle type, and the value contains the total number of trips, the total revenue, and the average tip percentage (based on the total revenue), in this order.
       All the values in the output must have a precision of two decimal digits.
       The output directory produced by Hadoop should be named VehicleType.
       Note: if total revenue is zero, then tip percentage is zero as well.
    b) List the top 10 agents by total revenue.
       Output: A key-value pair per line, where the key is the agent name, and the value contains the total revenue.
       All the values in the output must have a precision of two decimal digits.
       The output directory produced by Hadoop should be named Top10Revenue.
       Note: This is a Top K task, so specifically for this task, remember to use a single reducer (otherwise you may have one Top K for each reducer).