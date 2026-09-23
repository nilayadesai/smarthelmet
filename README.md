# Smart Helmet – Accident Detection & Emergency Alert System

An IoT-based smart helmet system using ESP32, MPU6050, GPS, Wi-Fi, and Twilio API to detect potential accidents and automatically send the rider's location to an emergency contact.

## Scenario

The Smart Helmet continuously monitors the motion and acceleration of the rider using an MPU6050 sensor.

If a sudden impact causes the resultant acceleration to exceed a predefined threshold, the system considers it a potential accident.

After detecting an accident:

- The ESP32 detects the sudden impact using MPU6050.
- GPS data is processed to obtain the rider's latitude and longitude.
- The ESP32 connects to a Wi-Fi network.
- The GPS coordinates are converted into a Google Maps location link.
- The Twilio API is used to send an emergency SMS.
- The emergency contact receives the accident alert along with the location.

## Initial Block Diagram

![Smart Helmet Block Diagram](images/block_diagram.png)

## System Architecture

![System Architecture](images/system_architecture.png)

## High-Level Data Flow

The Smart Helmet consists of the following major modules:

- MPU6050: Detects sudden acceleration and impact.
- ESP32: Acts as the main controller.
- GPS Module: Provides latitude and longitude.
- Wi-Fi: Provides internet connectivity.
- Twilio API: Sends the emergency SMS.
- Emergency Contact: Receives the accident alert and location.

The overall data flow is:

```text
MPU6050
   |
   v
ESP32
   |
   | Accident Detected
   v
GPS Module
   |
   | Latitude + Longitude
   v
Wi-Fi
   |
   v
Twilio API
   |
   v
Emergency Contact
