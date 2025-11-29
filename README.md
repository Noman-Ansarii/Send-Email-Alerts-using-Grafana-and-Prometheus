# 🚀 Monitor Express.js App with Prometheus + Grafana & Email Alerts

A simple and beginner-friendly guide to monitor your **Express.js application** with **Prometheus** and **Grafana**, and receive **real-time email alerts** when requests come in.

---

## ⚡ Step 1: Initialize Node.js Project

```bash
mkdir express-prometheus-grafana
cd express-prometheus-grafana
npm init -y
npm install express prom-client
```

---

## ⚡ Step 2: Create Express Server with Prometheus Metrics

Create a file **server.js**:

```js
const express = require("express");
const client = require("prom-client");

const app = express();
const PORT = 3001;

// Registry
const register = new client.Registry();
client.collectDefaultMetrics({ register });

// Custom metric: total requests counter
const httpRequestCount = new client.Counter({
  name: "http_request_count",
  help: "Total number of HTTP requests",
});
register.registerMetric(httpRequestCount);

// Middleware: increase counter on each request
app.use((req, res, next) => {
  httpRequestCount.inc();
  next();
});

// Example route
app.get("/", (req, res) => {
  res.send("Hello Prometheus + Grafana!");
});

// Metrics endpoint for Prometheus
app.get("/metrics", async (req, res) => {
  res.set("Content-Type", register.contentType);
  res.end(await register.metrics());
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

Run the server:

```bash
node server.js
```

👉 Open in browser: [http://localhost:3001/metrics](http://localhost:3001/metrics) (You should see Prometheus metrics ✅)

---

## ⚡ Step 3: Configure Prometheus

Open the Prometheus config file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add job:

```yaml
scrape_configs:
  - job_name: "express-app"
    static_configs:
      - targets: ["<your-system-ip>:3001"]
```

👉 Replace `<your-system-ip>` with your actual machine IP (example: `192.168.1.100:3001`).

Save → exit → restart Prometheus:

```bash
sudo systemctl restart prometheus
```

Check targets:  
👉 Open [http://localhost:9090/classic/targets](http://localhost:9090/classic/targets) → verify **express-app UP ✅**

---

## ⚡ Step 4: Configure SMTP in Grafana (Email Settings)

Edit Grafana config:

```bash
sudo nano /etc/grafana/grafana.ini
```

Uncomment + set:

```ini
[smtp]
enabled = true
host = smtp.gmail.com:587
user = ansariaya995@gmail.com
password = "bdof axxf khxi hzkq"   ; (App Password from Gmail)
from_address = ansariaya995@gmail.com
from_name = Grafana
startTLS_policy = OpportunisticStartTLS
```

Save + restart Grafana:

```bash
sudo systemctl restart grafana-server
sudo systemctl status grafana-server
```

---

## ⚡ Step 5: Create Contact Point in Grafana

1. Open Grafana UI → **Alerting → Contact Points → New Contact Point**  
2. Type: **Email**  
3. Name: `My Email`  
4. Recipients: `your@domain.com`  
5. Save  
6. (Optional) Send Test Email → check inbox ✅  

---

## ⚡ Step 6: Create Dashboard in Grafana

1. Open Grafana UI → **Dashboards → New Dashboard → Add new panel**  
2. In query box, write:

   ```promql
   rate(http_request_count[1m])
   ```

3. Panel will show **per-minute request rate**.  
4. Go to **Alert tab** → Enable Alert.  
5. Condition: `A > 0` → means request received.  
6. Notification → Select Contact Point: `My Email`.  
7. Save Dashboard ✅  

---

## ✅ Final Result

- Express app will expose metrics.
- Prometheus will scrape those metrics.
- Grafana dashboard will show request rate graph (`rate(http_request_count[1m])`).
- If a request comes in → condition `A > 0` triggers.
- Grafana will immediately send you an **email alert** 🚨📧

👉 **Result Example:**

- Open [http://localhost:3001/](http://localhost:3001/) → request count increases.
- Grafana dashboard shows a spike 📈
- Your email inbox receives:  
  *"Alert firing: Express App Request Alert – at least 1 request detected!"*
### 🎉 Congratulations!  
You just built a **full monitoring setup** for your Express.js app using **Prometheus + Grafana + Email Alerts**.
