# Environmental Impact Data-Logger – Node-RED Flows  
*A reference implementation for real-time environmental monitoring and automatic data submission to the **Bangkok Metropolitan Administration – Building Control Office (BMA BCO)** platform.*

<p align="center">
  <img src="docs/architecture.svg" width="620" alt="System-Level Overview"/>
</p>

---

## Table of Contents
1. [Overview](#1-overview)  
2. [Architecture](#2-architecture)  
3. [Repository Structure](#3-repository-structure)  
4. [Prerequisites](#4-prerequisites)  
5. [Quick Start](#5-quick-start)  
6. [Configuration](#6-configuration)  
7. [Security Considerations](#7-security-considerations)  
8. [Development & Contribution](#8-development--contribution)  
9. [License](#9-license)  
10. [References](#10-references)  

---

## 1. Overview
This repository hosts the complete **Node-RED** source code and helper services that power the *Environmental Impact Data-Logger* prototype.  
The system **polls multi-sensor nodes via RS-485 / Modbus RTU**, validates and normalises measurements, converts them to the official **BMA E-Permit JSON schema**, and pushes the payload to the **BCO API Gateway** over an **HTTPS + JWT** secured channel.

| Design Goal | Rationale |
|-------------|-----------|
| **Standards-Based I/O** | Modbus RTU ensures vendor neutrality and future sensor expansion. |
| **Edge Validation** | Guarantee schema compliance before uplink; reduce bandwidth. |
| **Secure, Resilient Uplink** | TLS 1.3, short-lived JWT, automatic retry with exponential back-off. |
| **Minimal Footprint** | Runs on an EdgeBox RPi 200 with \< 5 % CPU at 10 s polling. |
| **Drag-and-Drop Extensibility** | Add new sensors or cloud endpoints directly in Node-RED. |

---

## 2. Architecture
```mermaid
graph LR
  subgraph Sensor_Layer
    A(Weather Station<br/>(Wind · Temp · RH · PM · Noise))
    B(Vibration Sensor<br/>(3-axis))
  end
  subgraph Edge_Layer
    C[EdgeBox RPi 200<br/>Modbus Agent · JSON Mapper · Watch-dog · LTE Link]
  end
  subgraph Cloud_Layer
    D[BMA BCO<br/>API Gateway<br/>(E-Permit Dashboard)]
  end
  A -- RS-485&nbsp;/&nbsp;Modbus --> C
  B -- RS-485&nbsp;/&nbsp;Modbus --> C
  C -- LTE&nbsp;/&nbsp;HTTPS&nbsp;+&nbsp;JWT --> D
