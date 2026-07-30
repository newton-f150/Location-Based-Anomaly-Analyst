# Spatial-Temporal Anomaly Detection System

A general-purpose intelligent monitoring framework designed to identify unusual or impossible events by analyzing the relationship between **where something happens (spatial data)** and **when it happens (temporal data)**.

## 📋 Table of Contents

- [About The Project](#about-the-project)
- [Core Concept](#core-concept)
- [Problem Statement](#problem-statement)
- [Mathematical Framework](#mathematical-framework)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Applications](#applications)
- [API Reference](#api-reference)
- [Contributing](#contributing)
- [License](#license)

---

## About The Project

The Spatial-Temporal Anomaly Detection System is based on the idea that physical events follow natural limitations. Objects, people, devices, or identities cannot instantly move between distant locations. By combining location information with timestamps, the system can detect patterns that are physically impossible or highly unusual.

This concept can be applied across many fields, including:
- Security systems
- IoT networks
- Transportation
- Access control
- Asset tracking
- Identity verification

---

## Core Concept

Every event has two important dimensions:

### Spatial Information
Describes **where** an event occurs.

**Examples:**
- GPS coordinates
- Building locations
- Network locations
- Sensor positions
- Device locations

### Temporal Information
Describes **when** an event occurs.

**Examples:**
- Timestamp
- Duration
- Frequency
- Sequence of events

By analyzing both dimensions together, the system can determine whether an event is realistic.

---

## Problem Statement

Many systems verify only whether an object, device, or identity is valid. However, even valid entities can exhibit suspicious behavior. For example:
- A valid access card used at two locations 500km apart within 10 minutes
- A valid user logging in from New York and Tokyo within 1 hour
- A valid IoT device sending data from impossible physical locations

This system addresses these gaps by analyzing **spatial-temporal consistency**.

---

## Mathematical Framework

### Core Equation

The **Spatial-Temporal Anomaly Score** for an event \( e_i \) is defined as:

\[
\mathcal{A}(e_i) = \alpha \cdot \mathcal{S}(e_i) + \beta \cdot \mathcal{T}(e_i) + \gamma \cdot \mathcal{C}(e_i)
\]

Where:
- \( \mathcal{A}(e_i) \) = Anomaly score (higher = more anomalous)
- \( \mathcal{S}(e_i) \) = Spatial anomaly component
- \( \mathcal{T}(e_i) \) = Temporal anomaly component  
- \( \mathcal{C}(e_i) \) = Spatio-temporal coupling component
- \( \alpha, \beta, \gamma \) = Weighting coefficients (\( \alpha + \beta + \gamma = 1 \))

---

### Component 1: Spatial Anomaly

\[
\mathcal{S}(e_i) = \frac{d(e_i, \mu_{spatial})}{\sigma_{spatial} + \epsilon}
\]

Where:
- \( d(e_i, \mu_{spatial}) \) = Spatial distance from expected location (e.g., Haversine distance for GPS)
- \( \mu_{spatial} \) = Expected spatial centroid (historical mean)
- \( \sigma_{spatial} \) = Spatial standard deviation
- \( \epsilon \) = Small constant to avoid division by zero

---

### Component 2: Temporal Anomaly

\[
\mathcal{T}(e_i) = \frac{|t_i - \mu_{temporal}|}{\sigma_{temporal} + \epsilon}
\]

Where:
- \( t_i \) = Timestamp of event \( i \)
- \( \mu_{temporal} \) = Expected time (historical mean, or periodic mean for daily/weekly patterns)
- \( \sigma_{temporal} \) = Temporal standard deviation

For **periodic patterns** (daily cycles):
\[
\mu_{temporal} = \text{mod}(t_i, 24 \text{ hours}) \quad \text{(circular mean)}
\]

---

### Component 3: Spatio-Temporal Coupling (Velocity Constraint)

This is the critical component that detects **physical impossibilities**:

\[
\mathcal{C}(e_i, e_{i-1}) = 
\begin{cases} 
0, & \text{if } v(e_i, e_{i-1}) \leq v_{max} \\
\frac{v(e_i, e_{i-1}) - v_{max}}{v_{max}}, & \text{if } v(e_i, e_{i-1}) > v_{max}
\end{cases}
\]

Where:
\[
v(e_i, e_{i-1}) = \frac{d(e_i, e_{i-1})}{\Delta t}
\]

- \( d(e_i, e_{i-1}) \) = Spatial distance between consecutive events
- \( \Delta t = t_i - t_{i-1} \) = Time difference between consecutive events
- \( v_{max} \) = Maximum physically possible speed (configurable per context)

---

### Complete Anomaly Classification

An event is classified as **anomalous** if:

\[
\mathcal{A}(e_i) > \theta
\]

Where \( \theta \) is a threshold determined by:
- Statistical methods (e.g., \( \mu_{\mathcal{A}} + 3\sigma_{\mathcal{A}} \))
- Percentile-based (e.g., 99th percentile of historical scores)
- Domain-specific expert rules

---

### Extended Multi-Event Formulation

For analyzing a sequence of \( n \) events:

\[
\mathcal{A}_{sequence} = \frac{1}{n} \sum_{i=1}^n \mathcal{A}(e_i) + \lambda \cdot \max_{i=2..n} \mathcal{C}(e_i, e_{i-1})
\]

Where \( \lambda \) weights the maximum velocity violation (to catch "teleportation" events).

---

### Real-Time Update Equations

The system can adapt online using exponential moving averages:

\[
\mu_{spatial}^{(t)} = \rho \cdot \mu_{spatial}^{(t-1)} + (1-\rho) \cdot \text{location}_t
\]

\[
\mu_{temporal}^{(t)} = \rho \cdot \mu_{temporal}^{(t-1)} + (1-\rho) \cdot t
\]

\[
\sigma_{spatial}^{(t)} = \rho \cdot \sigma_{spatial}^{(t-1)} + (1-\rho) \cdot |\text{location}_t - \mu_{spatial}^{(t)}|
\]

Where \( \rho \) is the forgetting factor (typically 0.9-0.99).

---

### Multi-Dimensional Extensions

For systems with additional metadata (user ID, device type, etc.), extend to:

\[
\mathcal{A}_{total}(e_i) = \mathcal{A}(e_i) + \sum_{k=1}^m w_k \cdot \mathcal{D}_k(e_i)
\]

Where \( \mathcal{D}_k \) are domain-specific anomaly scores for other dimensions.

---

## Installation

### Prerequisites
- Python 3.8+
- pip package manager

### Install from PyPI
```bash
pip install spatial-temporal-anomaly
