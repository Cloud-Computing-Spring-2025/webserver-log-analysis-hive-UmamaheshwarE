# Web-Server-Log-Analysis
# Problem Statement:
You need to process a web server log file where each entry is represented as a row in a CSV file, formatted as:

ip,timestamp,url,status,user_agent 192.168.1.1,2024-02-01 10:15:00,/home,200,Mozilla/5.0 192.168.1.2,2024-02-01 10:16:00,/products,200,Chrome/90.0 192.168.1.3,2024-02-01 10:17:00,/checkout,404,Safari/13.1 192.168.1.10,2024-02-01 10:18:00,/home,500,Mozilla/5.0 192.168.1.15,2024-02-01 10:19:00,/products,404,Chrome/90.0
Your task is to analyze the web server logs using Apache Hive and perform the following tasks:

Count Total Web Requests: Calculate the total number of requests processed.

Analyze Status Codes: Determine the frequency of HTTP status codes (e.g., 200, 404, 500).

Identify Most Visited Pages: Extract the top 3 most visited URLs.

Traffic Source Analysis: Identify the most common user agents (browsers).

Detect Suspicious Activity: Identify IP addresses with more than 3 failed requests (status 404 or 500).

Analyze Traffic Trends: Calculate the number of requests per minute to observe traffic patterns.

Implement Partitioning: Use partitioning by status code to optimize query performance.
# Question-1
### To count Total no of web-requests
# Query 
```sh
INSERT OVERWRITE DIRECTORY '/user/hue/output/total_requests'
SELECT COUNT(*) FROM hue__tmp_web_server_logs
```
# To retrive the query-1 output in bash
```sh
    hdfs dfs -getmerge /user/hue/output/total_requests total_requests.txt
```
# Question-2
### To Analyze the frequncy of status codes
# Query
```sh
INSERT OVERWRITE DIRECTORY '/user/hue/output/status_codes'
SELECT status, COUNT(*) 
FROM hue__tmp_web_server_logs
GROUP BY status 
ORDER BY status;
```
# To retrive the query-2 output in bash
```sh
    hdfs dfs -getmerge /user/hue/output/status_codes status_codes.txt
```
# Question-3
### To Get Most visited pages
# query
```sh
INSERT OVERWRITE DIRECTORY '/user/hue/output/most_visited_pages'
SELECT url, COUNT(*) AS visit_count
FROM hue__tmp_web_server_logs
GROUP BY url
ORDER BY visit_count DESC
LIMIT 3;
```
# To retrive the query-3 output in bash
```sh
    hdfs dfs -getmerge /user/hue/output/most_visited_pages most_visited_pages.txt   
```
# question-4 
### To do the Traffic Source Analysis
# Query
```sh
INSERT OVERWRITE DIRECTORY '/user/hue/output/user_agents'
SELECT user_agent, COUNT(*) AS user_count
FROM hue__tmp_web_server_logs
GROUP BY user_agent
ORDER BY user_count DESC;
```
# To retrive the query-4 output in bash
```sh
 hdfs dfs -getmerge /user/hue/output/user_agents user_agents.txt   
```
# question-5 
### To Export Suspicious IP Addresses
# query
```sh
INSERT OVERWRITE DIRECTORY '/user/hue/output/suspicious_ips'
SELECT ip, COUNT(*) AS failed_requests
FROM hue__tmp_web_server_logs
WHERE status IN (404, 500)
GROUP BY ip
HAVING COUNT(*) > 3
ORDER BY failed_requests DESC;
```
# To retrive the query-5 output in bash
```sh
 hdfs dfs -getmerge /user/hue/output/suspicious_ips suspicious_ips.txt   
```
# question-6 
### To get the traffic trend Overtime
# query
```sh
INSERT OVERWRITE DIRECTORY '/user/hue/output/traffic_trends'
SELECT DATE_FORMAT(`timestamp`, 'yyyy-MM-dd HH:mm') AS request_minute, COUNT(*) AS request_count
FROM hue__tmp_web_server_logs
GROUP BY DATE_FORMAT(`timestamp`, 'yyyy-MM-dd HH:mm')
ORDER BY request_minute;
```
# To retrive the query-6 output in bash
```sh
 hdfs dfs -getmerge /user/hue/output/traffic_trends traffic_trends.txt   
```





# To up the Docker
```sh
 docker compose up -d  
```
# To up the container
```sh
docker exec -it resourcemanager /bin/bash 
```

# To get the output  of the individual query from container to the output folder
```sh
docker cp resourcemanager:/total_requests.txt output/
```
```sh
docker cp resourcemanager:/status_codes.txt output/
```
```sh
docker cp resourcemanager:/most_visited_pages.txt output/
```
```sh
docker cp resourcemanager:/user_agents.txt output/
```
```sh
docker cp resourcemanager:/suspicious_ips.txt output/
```
```sh
docker cp resourcemanager:/traffic_trends.txt output/
```
# challenges Faced:
  ### 1
  Faced an issue with the ports so have to delete repo and done it from the start
  ### 2
  Faced an issue with overwriting Directory 

#   Sample Inputs

 ### ip,timestamp,url,status,user_agent 192.168.1.1,2024-02-01 10:15:00,/home,200,Mozilla/5.0 192.168.1.2,2024-02-01 10:16:00,/products,200,Chrome/90.0 192.168.1.3,2024-02-01 10:17:00,/checkout,404,Safari/13.1 192.168.1.10,2024-02-01 10:18:00,/home,500,Mozilla/5.0 192.168.1.15,2024-02-01 10:19:00,/products,404,Chrome/90.0

## Sample output
#### Total Web Requests:

Total Requests: 100

#### Status Code Analysis:

200: 80

404: 10

500: 10

#### Most Visited Pages:

/home: 50

/products: 30

/checkout: 20

#### Traffic Source Analysis:

Mozilla/5.0: 60

Chrome/90.0: 30

Safari/13.1: 10

#### Suspicious IP Addresses:

192.168.1.10: 5 failed requests

192.168.1.15: 4 failed requests

#### Traffic Trend Over Time:

2024-02-01 10:15: 5 requests

2024-02-01 10:16: 7 requests


