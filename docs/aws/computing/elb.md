# Elastic Load Balancer (ELB)

## Overview
Elastic Load Balancer is a service that forwards traffic to multiple servers (such as EC2 instances) downstream. It acts as a single point of contact for all incoming web traffic to your applications.

## Benefits of Load Balancing

* Spreads load across multiple downstream instances
* Provides a single point of access (DNS) to your application
* Handles failures of downstream instances automatically
* Performs regular health checks on instances
* Provides SSL termination (HTTPS) for websites
* Enforces stickiness with cookies
* Ensures high availability across zones
* Separates public traffic from private traffic

## Why Choose Elastic Load Balancer?

* Fully managed by AWS with guaranteed uptime
* AWS handles upgrades, maintenance, and high availability
* Simplified configuration options
* Cost-effective compared to setting up your own load balancer
* Integrated with various AWS services:
  * EC2 and EC2 Auto Scaling Groups
  * Amazon ECS
  * AWS Certificate Manager (ACM)
  * CloudWatch
  * Route 53
  * AWS WAF
  * AWS Global Accelerator

## Types of Load Balancers

### Classic Load Balancer (CLB) - 2009
* Supports HTTP, HTTPS, TCP, and SSL (secure TCP)
* Legacy load balancer (first generation)

### Application Load Balancer (ALB) - 2016
* Operates at Layer 7 (HTTP)
* Supports HTTP, HTTPS, WebSocket
* Features:
  * Path-based routing
  * Host-based routing
  * Query string/header-based routing
  * Support for HTTP/2 and WebSocket
  * Support for redirects
  * Container-friendly with dynamic port mapping

### Network Load Balancer (NLB) - 2017
* Operates at Layer 4 (TCP/UDP)
* Features:
  * Handles millions of requests per second
  * Ultra-low latency
  * Static IP per AZ with Elastic IP support
  * Ideal for extreme performance needs

### Gateway Load Balancer (GWLB) - 2020
* Operates at Layer 3 (Network layer)
* Used for deploying and managing third-party virtual appliances

Combines:

  * Transparent Network Gateway
  * Load Balancer functionality

## Health Checks
* Essential for load balancer operation
* Monitors instance availability

Configured with:

  * Protocol
  * Port
  * Endpoint path
  * Response code expectations

## Security Groups
* Load Balancer Security Group:
  
  - Allows inbound HTTPS/HTTP from anywhere
  
* Application Security Group:
  
  - Allows traffic only from Load Balancer

## Target Groups
Support various target types depending on the load balancer:

  * EC2 instances
  * IP addresses (must be private IPs)
  * Lambda functions (ALB only)
  * Application Load Balancers (NLB only)
  * Container instances

## Sticky Sessions
* Ensures client requests route to the same instance
* Supported by all load balancer types
* Cookie types:

  * Application-based Cookies:

    * Custom cookies (generated by target)
    * Application cookies (generated by load balancer - AWSALBAPP)

  * Duration-based Cookies:

    * Generated by load balancer
    * AWSALB for ALB, AWSELB for CLB

## Cross-Zone Load Balancing
* ALB: Enabled by default, no inter-AZ charges
* NLB and GWLB: Disabled by default, charges apply for inter-AZ data if enabled
* CLB: Disabled by default, no inter-AZ charges if enabled

## SSL/TLS Support
* Provides in-flight encryption
* Certificate management through AWS Certificate Manager (ACM)
Features:

  * Support for multiple certificates (ALB and NLB)
  * Server Name Indication (SNI) support
  * Customizable security policies

Certificate handling varies by load balancer type:

  * CLB: Single SSL certificate only
  * ALB/NLB: Multiple listeners with multiple SSL certificates via SNI

## Connection Draining
* Named "Connection Draining" for CLB
* Named "Deregistration Delay" for ALB & NLB
* Allows completion of in-flight requests during instance deregistration
Configuration options:

  * Duration: 1-3600 seconds (default 300)
  * Can be disabled (set to 0)
  * Recommended to set lower values for short-lived requests
