# Project Structure

## Project
MGM Resorts Cybersecurity Attack Simulation

## Team Machines

### Red Team
Machine: red-kali
Role: Attack simulation and evidence collection

### Employee Endpoint
Machine: enterprises1-rocky
Role: Simulated employee workstation / initial attack entry point

### Enterprise/File Server
Machine: enterprises2-rocky
Role: Enterprise network, file server, and simulated business data

### Blue Team
Machine: blue-rocky
Role: Application server, monitoring, detection, containment, and recovery

## Network
- Cisco Packet Tracer: Enterprise network architecture
- Tailscale: VPN connectivity between the lab machines
- All attacks are restricted to the team's controlled lab environment.

## Attack Flow

Employee Endpoint
→ Initial Access Simulation
→ Enterprise/File Server
→ Detection by Blue Team
→ Containment
→ Recovery