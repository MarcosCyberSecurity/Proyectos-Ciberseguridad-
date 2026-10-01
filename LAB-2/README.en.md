
# 🌐 Lab 2: Deploying an Nginx Web Server and Analysing Audit Logs (Baseline)

## 🎯 Objective

The aim of this exercise is to deploy an Nginx web server locally in order to understand the structure of the access logs (`access.log`) and error logs (`error.log`). Log analysis is fundamental in cybersecurity for establishing a baseline of legitimate traffic and learning to identify anomalies, vulnerability scans or intrusions.

## 🧰 Tools and Commands Used

* **`Nginx`**: A lightweight web server used to simulate the production environment.
  
* **`tail -f /var/log/nginx/access.log`**: To monitor and analyse incoming HTTP requests in real time.
  
* **`ls -l /var/log/nginx/`**: To audit storage permissions and the location of audit log folders.

----

## 🚀 Execution and Investigation Process

### 1. Initialising the Web Service

We start the Nginx server locally on our machine using elevated privileges:

```bash
sudo /usr/sbin/nginx/
```


### 2. Log Directory Audit

We access and check the folders and files where the server logs its activity, ensuring that both `access.log` and `error.log` are collecting data correctly:

```bash
ls -l /var/log/nginx/
```


### 3. Simulating Legitimate Web Traffic

We connect to the web server locally using the loopback IP address `127.0.0.1` on the standard port `80`. The photorealistic interface confirms that the server is responding correctly:

![Lab 2 Screenshot](img/Captura%20de%20pantalla%202026-10-01%20180718.png)



### 4. Analysis and Interpretation of Real-Time Access Logs
To verify what happens behind the scenes every time a user interacts with the website, we run the interactive tracing command:


```bash
sudo tail -f /var/log/nginx/access.log
```

**Technical analysis of the lines captured in the log:**

* **HTTP status codes 200 / 304:** Client requests with the status `GET / HTTP/1.1` indicate that the server successfully returned the home page (`200 OK`) or served the resource from the browser’s cache without any changes (`304 Not Modified`).

* **HTTP status code 404:** Failed requests are logged as `GET /OO HTTP/1.1 404`. In a real-world Blue Team environment, an unusual spike in consecutive 404 codes would alert us to a possible automated directory scan (e.g. Dirbuster, Gobuster).
  
* Client Identification (User-Agent):** The log accurately records the origin of the request: `Mozilla/5.0 (X11; Ubuntu; Linux x86_64...)`, which is crucial information in forensic analysis for identifying the digital fingerprint (*fingerprinting*) of a client or attacker.

----

## 🛡️ Conclusions on Cybersecurity

Monitoring logs on web servers is the cornerstone for feeding SIEM systems (such as Splunk or Wazuh). Understanding how a legitimate request is logged in Nginx enables us to create detection rules capable of identifying malicious behaviour, such as SQL injection (SQLi) attacks, cross-site scripting (XSS) or brute-force attacks.
