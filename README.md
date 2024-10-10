# BSC Analytics TacoCon 2024 Data Bakeoff

## AWS Credentials Setup and EKS Cluster Access

### Adding a New Profile to the AWS Credentials File

To add a new profile to your AWS credentials file:
* **Open**
```bash
vim ~/.aws/credentials
```
* **Append**
```bash
[eks-gameday]
aws_access_key_id = AK...
aws_secret_access_key = za...
```

### AWS CLI Command to Connect to EKS Cluster

Once the profile is added, you can use the AWS CLI to connect to our EKS cluster:

```bash
aws eks update-kubeconfig --name dreamcanvas_dev --region us-east-1 --profile eks-gameday
```

Verify EKS Cluster Connectivity
After connecting to the EKS cluster, check if the connectivity is working by running the following command:

```bash
k get po -n kafka
```

If the connection is successful, you should see a single node in the EKS cluster.

![pods.png](images/pods.png)

### Consuming the Kafka Topic

To consume messages from the correct Kafka topic:

Here are two ways to consume the logs
1. Run Kafka CLI in cluster
OR
2. Simply use `k logs`

* **Run Kafka CLI in cluster**

```bash
k run <YOUR_TEAM_NAME> -l=kafka=consumer -n kafka --rm -i --tty --image=confluentinc/cp-kafka -- \
kafka-console-consumer --bootstrap-server kafka:9092 --topic gameday --from-beginning
```

This command runs a Kafka consumer in the kafka namespace to consume messages from the gameday topic starting from the beginning.

![consumer.png](images/consumer.png)

* **Simply use `k logs`**

```bash
todd@MacBookPro: Downloads » k logs -n kafka kafka-python-producer -f
{"start": "LOG_START", "log_message": {"log_identifier": "LOG-000000", "timestamp": 1728565962.2251318, "log_level": "DEBUG", "ip_address": "152.142.140.53", "user_id": "e6158ec1-f8c5-4517-b4af-a3954637fcce", "method": "DELETE", "path": "wp-content/list", "status_code": 502, "response_time": 0.146, "message": "Seat west value able major."}, "stop": "LOG_END"}
{"start": "LOG_START", "log_message": {"log_identifier": "LOG-000001", "timestamp": 1728565965.3893456, "log_level": "INFO", "ip_address": "31.36.210.183", "user_id": "99f56822-ee5e-45e4-81c6-1eb4bffd734e", "method": "DELETE", "path": "app/wp-content/main", "status_code": 404, "response_time": 2.622, "message": "Few its light shoulder play."}, "stop": "LOG_END"}
{"start": "LOG_START", "log_message": {"log_identifier": "LOG-000002", "timestamp": 1728565986.2584338, "log_level": "ERROR", "ip_address": "116.206.152.219", "user_id": "b14234b4-a2c8-487f-935c-6f307f78d913", "method": "GET", "path": "search/blog", "status_code": 201, "response_time": 1.208, "message": "Available next stuff my network picture simple young."}, "stop": "LOG_END"}
```

```bash
todd@MacBookPro: temp » k logs -n kafka kafka-python-producer -f | python3 forward.py
LOG-000000 forwarded to next step...
LOG-000001 forwarded to next step...
LOG-000002 forwarded to next step...
LOG-000003 forwarded to next step...
LOG-000004 forwarded to next step...
LOG-000005 forwarded to next step...
LOG-000006 forwarded to next step...
LOG-000007 forwarded to next step...
LOG-000008 forwarded to next step...
LOG-000009 forwarded to next step...
LOG-000010 forwarded to next step...
LOG-000011 forwarded to next step...
LOG-000012 forwarded to next step...
LOG-000013 forwarded to next step...
LOG-000014 forwarded to next step...
LOG-000015 forwarded to next step...
LOG-000016 forwarded to next step...
LOG-000017 forwarded to next step...
LOG-000018 forwarded to next step... 
```

### Solve this problem

Use a ML model to find outliers in the following areas:

* Log level
* Response time

You can use a home-grown model or any cloud managed models.

* AWS Sagemaker
* Azure Anomaly Detector
* DBSCAN
* Google Vertex
* Any other

### What we did

1. **Get access to the stream locally**  
   Access the Kafka stream by consuming the logs from the `gameday` topic as described in the previous section.

2. **Find a way to clean and store logs**  
   Use appropriate tools or scripts to clean the logs, ensuring the data is structured and ready for analysis. You can store the logs in a temporary database or cloud storage (e.g., S3), depending on your team's setup.

3. **Train a model** (if necessary - we used RCF)  
   If your solution requires training a model, decide which ML model to use. We used **RCF (Random Cut Forest)**, but you can choose any model or cloud-based service. Ensure the model is trained on clean log data to detect outliers in log level and response time.

4. **Find a way to append that outlier score to the data** (if necessary)  
   After detecting outliers, append the outlier score to the logs for further analysis or reporting.

5. **Visualize the data and display outlier logs**  
   Visualize the outlier logs using a tool of your choice (e.g., Grafana, Kibana, or any cloud-native solution).
![quicksight.png](images/quicksight.png)

### Recommendation to Get with Your Team

- **Decide what cloud and solution**  
   Collaborate with your team to choose the cloud provider (AWS, Azure, Google) and solution approach that works best for you.

- **How you are building it**  
   Plan how you will develop and integrate the solution, from consuming logs to detecting outliers.

- **Confirm you all have access to that cloud sandbox**  
   Make sure everyone on your team has access to the necessary cloud resources and sandbox environments.

- **Build whatever you want ahead of time**  
   Prepare your solution components in advance, such as scripts, models, and visualizations, to streamline the process during the competition.

## Teams
| **Team 1**      | **Team 2**        | **Team 3**      | **Team 4**        |
|-----------------|-------------------|-----------------|-------------------|
| Ronda Peoples   | Steffan Williams  | Khane Mitchell  | Cody Schuyler     |
| Mahmood Rahimi  | Max Rahimi        | Kevin Riste     | Michael Olivieri  |
| Gabriel Tizatto | Gus Zanotto       | Will Ahlborn    | Andrew Deweeveer  |
| Robbie Douglas  | Patrick Agboola   | Mehmet Güngören | Kenzie Graham     |
| Caleb Cohen     | Jay Hollingsworth | Jean Pimmel     | Kelechi Nwosu     |

| **Team 5**           | **Team 6**        | **Team 7**      | **Team 8**         |
|----------------------|-------------------|-----------------|--------------------|
| Ashley Ellis         | Michael Berrigan  | Anthony Coccaro | Alexis Perez       |
| Dallin Rasmuson      | Jeremy Elam       | JR Bullington   | David Van Ginneken |
| Thomas Behnken       | Steve Opilo       | CMack           | Noor Alsabahi      |
| Jon Ide              | Nikita Usenko     | Beshr Alamir    | Ryan Young         |
| John Pratt           | Dastan Rahimi     | Brian Riste     | Nolan Johnson      |
|                      | Bola Abiodun      | Nate O’Connell  |                    |
