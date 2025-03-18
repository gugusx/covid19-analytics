# **COVID-19 Data Analysis with ClickHouse & Superset**

This repository provides a complete setup for storing, processing, and visualizing COVID-19 data using **ClickHouse** (as a database) and **Apache Superset** (as a visualization tool). The data includes COVID-19 cases, hospitalizations, vaccinations, testing, and historical weather data to analyze external factors affecting the pandemic.

---

## **Table of Contents**
- [Prerequisites](#prerequisites)
- [Setting Up ClickHouse](#setting-up-clickhouse)
- [Source Data](#source-data)
- [Preparing Data](#preparing-data)
- [Installing and Configuring Superset](#installing-and-configuring-superset)
- [Connecting Superset to ClickHouse](#connecting-superset-to-clickhouse)
- [Creating Charts and Dashboards](#creating-charts-and-dashboards)

---

## **A. Prerequisites**
Ensure the following dependencies are installed before proceeding:
1. **Python** (Using Jupyter Notebook)
2. **ClickHouse Docker Image** (To store raw data and results)
3. **Apache Superset Docker Image** (To visualize the results)

---

## **B. Setting Up ClickHouse**
### **1. Pull the ClickHouse Docker Image**
```bash
docker pull clickhouse/clickhouse-server:25.1
```
![ClickHouse Pull](<assets/Screenshot from 2025-03-17 15-44-32.png>)

### **2. Run ClickHouse Container**
```bash
docker run -d --name clickhouse-server --ulimit nofile=262144:262144 \
-p 8123:8123 -p 9000:9000 \
-e CLICKHOUSE_USER=covid_user \
-e CLICKHOUSE_PASSWORD=qwerty@123 \
-e CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT=1 \
-v clickhouse_data:/var/lib/clickhouse \
clickhouse/clickhouse-server:25.1
```

### **3. Connect to ClickHouse**
```bash
docker exec -it clickhouse-server clickhouse-client --user covid_user --password qwerty@123
```
![ClickHouse Connection](<assets/Screenshot from 2025-03-17 15-54-54.png>)

### **4. Create a New Database (`raw_data`)**
```sql
CREATE DATABASE raw_data;
```

---

## **C. Source Data**
1. **Main Data for COVID-19 Case and Impact Analysis**
   - ✅ **Cases and Deaths** – Daily counts of cases, deaths, and recoveries across different countries.
   - ✅ **Hospitalizations** – Number of COVID-19 patients admitted to hospitals.

2. **Supporting Data for External Factor Analysis**
   - ✅ **Vaccinations** – Global vaccination totals to analyze the impact on COVID-19 cases.
   - ✅ **Testing** – Number of COVID-19 tests conducted to assess spread and public health response.

3. **Historical Weather Data**
   - ✅ **January - February 2021** – Data from the initial Alpha variant wave.
   - ✅ **January - February 2022** – Data from the Omicron variant surge.

---

## **D. Preparing Data**
1. Fetch data from **CSV files and APIs**.
2. Create a **schema** for each dataset.
3. Handle missing values by replacing **null or NaN values**.
4. Insert processed data into **ClickHouse tables**.
5. (Optional) Create **views or tables** for analytical queries.

---

## **E. Installing and Configuring Superset**
### **1. Pull the Superset Docker Image**
```bash
docker pull apache/superset
```

### **2. Set a Secure `SECRET_KEY`**
- Generate a secure key:
  ```bash
  openssl rand -base64 42
  ```
- Create a `superset_config.py` file:
  ```bash
  mkdir -p ~/superset && nano ~/superset/superset_config.py
  ```
- Add the generated key:
  ```python
  SECRET_KEY = "your_secure_key_here"
  ```

### **3. Run the Superset Container (Port 9191)**
```bash
docker run -d --name superset -p 9191:8088 \
-v ~/superset/superset_config.py:/app/pythonpath/superset_config.py \
apache/superset
```

### **4. Create an Admin User**
```bash
docker exec -it superset superset fab create-admin
```
![Superset Admin](<assets/Screenshot from 2025-03-18 07-05-17.png>)

### **5. Initialize the Superset Database**
```bash
docker exec -it superset superset db upgrade
docker exec -it superset superset init
```

---

## **F. Connecting Superset to ClickHouse**
To connect Superset with ClickHouse, ensure both containers are on the same Docker network.

### **1. Create a Docker Network**
```bash
docker network create superset-network
```
### **2. Connect Superset and ClickHouse**
```bash
docker network connect superset-network superset
docker network connect superset-network clickhouse-server
```
### **3. Enter the Superset Container**
```bash
docker exec -it superset /bin/sh
```
### **4. Install the ClickHouse Driver**
```bash
pip install clickhouse-connect
```
### **5. Restart Superset**
```bash
docker restart superset
```

---

## **G. Creating Charts and Dashboards**
1. Open Superset at: [http://localhost:9191](http://localhost:9191)
2. Log in with:
   - **Username:** `admin_gugus`
   - **Password:** `qwerty123@`
   ![Superset Login](<assets/Screenshot from 2025-03-18 07-10-13.png>)

3. Navigate to **Settings → Database Connection → Add Database**  
   ![Superset Database](<assets/Screenshot from 2025-03-18 08-03-37.png>)

4. Create charts and dashboards  
   ![Superset Chart 1](<assets/Screenshot from 2025-03-18 13-09-00.png>)
   ![Superset Chart 2](<assets/Screenshot from 2025-03-18 12-15-10.png>)
   ![Superset Chart 3](<assets/Screenshot from 2025-03-18 12-31-18.png>)
   ![Superset Chart 4](<assets/Screenshot from 2025-03-18 13-07-04.png>)

---

## **I. License**
This project is open-source and free to use under the **MIT License**.

---
