# Spatial-Temporal Anomaly Detection : LB2A

A general-purpose intelligent monitoring framework designed to identify unusual or impossible events by analyzing the relationship between **where something happens (spatial data)** and **when it happens (temporal data)**.

---

## About The Project

The Spatial-Temporal Anomaly Detection System is based on the idea that physical events follow natural limitations. Objects, people, devices, or identities cannot instantly move between distant locations. By combining location information with timestamps, the system can detect patterns that are physically impossible or highly unusual.

This concept can be applied across many fields, including security systems, IoT networks, transportation, access control, asset tracking, and identity verification.

---

## Core Concept

Every event has two important dimensions:

**Spatial Information** describes *where* an event occurs. Examples include GPS coordinates, building locations, network locations, sensor positions, and device locations.

**Temporal Information** describes *when* an event occurs. Examples include timestamp, duration, frequency, and sequence of events.

By analyzing both dimensions together, the system can determine whether an event is realistic.

---

## Problem Statement

Many systems verify only whether an object, device, or identity is valid. However, even valid entities can exhibit suspicious behavior. For example:

- A valid access card used at two locations 500km apart within 10 minutes
- A valid IoT device sending data from impossible physical locations

This system addresses these gaps by analyzing spatial-temporal consistency.

---

## Mathematical Framework

### Core Equation

The **Spatial-Temporal Anomaly Score** for an event *eᵢ* is defined as:

**𝒜(eᵢ) = α · 𝒮(eᵢ) + β · 𝒯(eᵢ) + γ · 𝒞(eᵢ)**

Where:
- **𝒜(eᵢ)** = Anomaly score (higher = more anomalous)
- **𝒮(eᵢ)** = Spatial anomaly component
- **𝒯(eᵢ)** = Temporal anomaly component  
- **𝒞(eᵢ)** = Spatio-temporal coupling component
- **α, β, γ** = Weighting coefficients (α + β + γ = 1)

---

### Component 1: Spatial Anomaly

**𝒮(eᵢ) = d(eᵢ, μ_spatial) / (σ_spatial + ε)**

Where:
- **d(eᵢ, μ_spatial)** = Spatial distance from expected location (e.g., Haversine distance for GPS)
- **μ_spatial** = Expected spatial centroid (historical mean)
- **σ_spatial** = Spatial standard deviation
- **ε** = Small constant to avoid division by zero

---

### Component 2: Temporal Anomaly

**𝒯(eᵢ) = |tᵢ - μ_temporal| / (σ_temporal + ε)**

Where:
- **tᵢ** = Timestamp of event *i*
- **μ_temporal** = Expected time (historical mean, or periodic mean for daily/weekly patterns)
- **σ_temporal** = Temporal standard deviation

For **periodic patterns** (daily cycles):
**μ_temporal = mod(tᵢ, 24 hours)** (circular mean)

---

### Component 3: Spatio-Temporal Coupling (Velocity Constraint)

This is the critical component that detects **physical impossibilities**:

**𝒞(eᵢ, eᵢ₋₁) = 0**, if v(eᵢ, eᵢ₋₁) ≤ v_max  
**𝒞(eᵢ, eᵢ₋₁) = (v(eᵢ, eᵢ₋₁) - v_max) / v_max**, if v(eᵢ, eᵢ₋₁) > v_max

Where:
**v(eᵢ, eᵢ₋₁) = d(eᵢ, eᵢ₋₁) / Δt**

- **d(eᵢ, eᵢ₋₁)** = Spatial distance between consecutive events
- **Δt = tᵢ - tᵢ₋₁** = Time difference between consecutive events
- **v_max** = Maximum physically possible speed (configurable per context)

---

### Complete Anomaly Classification

An event is classified as **anomalous** if:

**𝒜(eᵢ) > θ**

Where **θ** is a threshold determined by:
- Statistical methods (e.g., μ_𝒜 + 3σ_𝒜)
- Percentile-based (e.g., 99th percentile of historical scores)
- Domain-specific expert rules

---

### Extended Multi-Event Formulation

For analyzing a sequence of *n* events:

**𝒜_sequence = (1/n) · Σᵢ₌₁ⁿ 𝒜(eᵢ) + λ · maxᵢ₌₂..ₙ 𝒞(eᵢ, eᵢ₋₁)**

Where **λ** weights the maximum velocity violation (to catch "teleportation" events).

---

### Real-Time Update Equations

The system can adapt online using exponential moving averages:

**μ_spatial⁽ᵗ⁾ = ρ · μ_spatial⁽ᵗ⁻¹⁾ + (1-ρ) · locationₜ**

**μ_temporal⁽ᵗ⁾ = ρ · μ_temporal⁽ᵗ⁻¹⁾ + (1-ρ) · t**

**σ_spatial⁽ᵗ⁾ = ρ · σ_spatial⁽ᵗ⁻¹⁾ + (1-ρ) · |locationₜ - μ_spatial⁽ᵗ⁾|**

Where **ρ** is the forgetting factor (typically 0.9-0.99).

---

### Multi-Dimensional Extensions

For systems with additional metadata (user ID, device type, etc.), extend to:

**𝒜_total(eᵢ) = 𝒜(eᵢ) + Σₖ₌₁ᵐ wₖ · 𝒟ₖ(eᵢ)**

Where **𝒟ₖ** are domain-specific anomaly scores for other dimensions.

---

## Card Cloning Detection Use Case

### Overview

This system is particularly effective for detecting credit/debit card cloning and fraudulent transactions. Card cloning typically involves creating a duplicate card and using it at a different location simultaneously or in quick succession. The spatial-temporal analysis can instantly flag such impossible movements.

### Detection Scenario

**Scenario**: A legitimate cardholder is in London, but a cloned card is used in New York 30 minutes later.

**Detection Process**:
1. Transaction 1: London, UK (51.5074° N, 0.1278° W) at 14:00
2. Transaction 2: New York, USA (40.7128° N, 74.0060° W) at 14:30

**Analysis**:
- Distance: ~5,570 km (great-circle distance)
- Time difference: 30 minutes = 1800 seconds
- Required speed: 5,570,000 / 1800 ≈ 3,094 m/s (~11,140 km/h)
- This exceeds any physically possible travel speed for a cardholder

**Result**: Anomaly detected immediately

### Transaction Processing Example

```python
from spatial_temporal_anomaly import CardTransactionDetector
from datetime import datetime

# Initialize detector with card-specific configuration
detector = CardTransactionDetector(
    velocity_threshold=50.0,      # 180 km/h (reasonable travel speed)
    transaction_window=3600,       # 1 hour window for velocity checks
    fraud_threshold=0.85,          # 85% probability triggers alert
    use_location_history=True      # Learn cardholder's patterns
)

# Track transactions per card
card_transactions = {
    'card_1234': [
        {
            'card_id': '1234',
            'merchant': 'Coffee Shop London',
            'amount': 4.50,
            'latitude': 51.5074,
            'longitude': -0.1278,
            'timestamp': datetime(2024, 1, 15, 14, 0, 0)
        },
        {
            'card_id': '1234', 
            'merchant': 'Electronics Store NYC',
            'amount': 1299.99,
            'latitude': 40.7128,
            'longitude': -74.0060,
            'timestamp': datetime(2024, 1, 15, 14, 30, 0)  # 30 minutes later
        }
    ]
}

# Process transactions
for card_id, transactions in card_transactions.items():
    results = detector.analyze_transactions(transactions)
    
    for result in results:
        if result['fraud_probability'] > 0.8:
            print(f"🚨 FRAUD ALERT! Card {card_id}")
            print(f"   Score: {result['fraud_probability']:.2%}")
            print(f"   Reason: {result['reason']}")
            print(f"   Distance: {result['distance_km']:.0f} km")
            print(f"   Time: {result['time_minutes']:.0f} minutes")
            print(f"   Required Speed: {result['required_speed_kmh']:.0f} km/h")
```

```
from spatial_temporal_anomaly import CardCloningDetector

# Detect card cloning across multiple merchants
cloning_detector = CardCloningDetector(
    temporal_threshold=300,        # 5 minutes
    spatial_threshold=100000,      # 100 km
    velocity_threshold=100.0,      # 360 km/h (flight speed)
    duplicate_check_window=3600,   # 1 hour
    merchant_risk_scoring=True,
    amount_anomaly_detection=True
)

# Analyze batch of transactions
batch_results = cloning_detector.detect_cloning_batch(
    transactions=all_transactions,
    batch_size=1000,
    parallel_processing=True
)

# Generate fraud report
report = cloning_detector.generate_report(batch_results)
print(f"Potential cloning incidents: {report['cloning_incidents']}")
print(f"At-risk cards: {report['at_risk_cards']}")
print(f"High-risk merchants: {report['high_risk_merchants']}")
```
