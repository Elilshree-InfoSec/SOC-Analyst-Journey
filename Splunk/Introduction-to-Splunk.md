# Cisco - Introduction To Splunk

## 1. What is Splunk?

Splunk is a unified platform that allows teams to work together or individually to ensure mission-critical digital systems stay secure and reliable.

### Core Capabilities

### 🔍 Searching with SPL
* **Cross-Source Analysis:** By entering a query into the Splunk search bar, you can find events that contain values across multiple data sources.
* **Statistical Analysis:** This allows you to analyze and run statistics on the events using **SPL (Search Processing Language)**.

### 📊 Reports & Dashboards
* **Saved Insights:** Search results can be saved to **Reports**, which provide ongoing insights into your data.
* **Visual Panels:** These saved reports can be directly used to power **Dashboard Panels** for real-time monitoring.

### 📐 Data Models & Pivot
* **Structured Datasets:** Knowledge can be organized into structured datasets called **Data Models**.
* **No-Code Visualization:** This allows users to quickly visualize data in **Pivot** without having to write custom searches.

### 🚨 Alerts
* **Custom Triggers:** Alerts allow you to set custom triggers so you are automatically notified of anomalies.
* **Incident Response:** This ensures you can respond to operational or security incidents as soon as they occur.

---

## 2. Splunk Web & Interface Notes

Comprehensive study notes covering Splunk Web components, user roles, application structure, and search fundamentals based on the Cisco Introduction to Splunk course material.

### 🧩 Splunk Apps Overview
* **Definition**: Apps are pre-configured environments sitting on top of a Splunk instance.
* **Purpose**: They extend built-in knowledge, visualizations, and capabilities.
* **Design**: Built specifically to solve a targeted use case or manage specific data sources.

### 👥 Splunk User Roles & Permissions
Roles determine exactly what a user can see, do, and interact with inside the platform.

| Role | Permissions & Capabilities |
| :--- | :--- |
| **Admin** | <ul><li>Most powerful system role</li><li>Can install apps and configurations</li><li>Can ingest and onboard new data</li><li>Can create global knowledge objects accessible to all users</li></ul> |
| **Power** | <ul><li>Can create and share knowledge objects with all users of a specific app</li><li>Can perform real-time data searches</li></ul> |
| **User** | <ul><li>Most restrictive default role</li><li>Can only see their own created knowledge objects</li><li>Can only see external knowledge objects explicitly shared with them</li></ul> |

> **Note**: Splunk also includes cloud-specific roles optimized for hosted environments.

### 🖥️ Splunk Web Home Interface
The Splunk Web Home screen serves as your primary launcher and management console.
* **Home Screen**: Used to launch, configure, and manage installed Splunk applications.
* **Documentation**: Built-in reference materials and guides accessible directly from the UI.
* **Customization**: Allows setting any custom dashboard as your default system landing page.
* **Data Ingestion**: Serve as a starting point to add new apps and browse/onboard new data sources.

### 🔍 The Search and Reporting App
The **Search and Reporting App** provides the default, core interface for querying, searching, and analyzing indexed data.

#### The 8 Main Components of the Interface
| Component | Description |
| :--- | :--- |
| **1. Splunk Bar** | Topmost bar to switch apps, edit account, check system messages, access settings, monitor jobs, and open help. |
| **2. App Bar** | Located below the Splunk bar. Used to navigate within the specific application currently open. |
| **3. Search Bar** | The central input field used to type and execute Search Processing Language (SPL) queries. |
| **4. Time Range Picker** | Dropdown menu used to constrain or retrieve events over a specific real-time or historical time period. |
| **5. Data Summary Button** | Provides an aggregate view of the data broken down by **Hosts**, **Sources**, and **Source Types**. |
| **6. Table Views** | A user-interface (UI)-driven way to explore, clean, and prepare data without needing to write SPL commands. |
| **7. Search History Menu** | Allows you to view, filter, and easily rerun past searches that you have previously executed. |
| **8. Search Terms / Elements** | Interactive elements within results to click and instantly narrow down active search events. |

