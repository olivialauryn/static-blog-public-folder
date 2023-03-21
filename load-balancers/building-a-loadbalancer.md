---
author: Olivia Snowden
comments: true
date: 2023
layout: post
slug: load-balancers
title: BUILDING A LOADBALANCER
tags: [code, networking, terraform, gcp]
---
**In this post:**

1. **What is a Load Balancer** 

2. **GCP Load Balancers** 

3. **Creating a Load Balancer using Terraform**

## What is a Load Balancer? 
Load balancers are networking resources that take in network traffic through their frontend and distribute that traffic to their backend(s). Load balancers have a few general attributes:
- Frontend 
  Where clients reach the load balancer, the "face" of the load balancer.
- Backend
  The server(s) where the load balancer directs traffic
- Forwarding Rules
  The logic the load balancer uses to distribute traffic. Also referred to as a load balancer's algorithm.
- Health Checks
  Checks the status of the backend(s) and whether they can recieve traffic. Load balancers will not send traffic to an unhealthy backend.

Below is a simple diagram that illustrates the flow of traffic through a load balancer:

![](/load-balancer.jpg)

Traffic enters the load balancer through its frontend IP, the forward rule determines how to route traffic to the backends, the health check confirms that the backend is healthy-and if so then the traffic is routed there.

### Rules
Deciding how to route traffic to a load balancer's backends depends on the use case. For example, if a network admin has 2 servers and server 1 can handle more requests than server 2, then the load balancer can be configured to use a weighted rule which will send more traffic to server 1.

However if a network admin has 2 servers with the same processing power they can confgure the load balancer to use a least connection rule which will send traffic to the server that is the most available at the time.

## GCP Load Balancers
Load balancers can exist in the cloud as well as on-premises networks. This post will primarily focus on internal/external TCP/HTTPS cloud-based load balancers in GCP, although similar load balancers exist in AWS and Azure cloud platforms. Load balancers in GCP work just like load balancers in any other network, with the exception that some or all of the traffic of GCP load balancers is betweek GCP cloud resources.

When creating a load balancer  in GCP, it is important to understand the different types of load balancers supported.

### Internal and External Load Balancers 
GCP supports 2 options for the IP address of their load balancers, internal and external.

Internal LBs only distribute traffic to resources within GCP. External LBs route traffic coming from the internet into GCP. It is important to note that external LBs often require additional security measures due to the LBs being public-facing.

### TCP/UDP and HTTP(S) Load Balancers
GCP supports 4 types of traffic on their load balancers: TCP, UDP, HTTP(S), and SSL. This post will primarily focus on TCP/UDP and HTTP(S) load balancing.

## Creating a Load Balancer using Terraform
Load balancers can be created in GCP using Terraform, however there is not a single Terraform resource that creates a load balancer. Instead, multiple Terraform resources together create a GCP load balancer. These resources include:
- "google_compute_global_address"    = The public IP address that is the frontend of the LB. (External LBs only)
- "google_compute_target_http_proxy" = Routes requests to the url map ("google_compute_global_forwarding_rule" for external LBs)
- "google_compute_url_map"           = Defines the rules to route traffic to the backend
- "google_compute_forwarding_rule"   = Routes traffic to the backends
- "google_compute_backend_service"   = Creates the LB's backend(s)
- "google_compute_health_check"      = The LB's health check

Note, that the global address and http proxy resources are different between internal and external LBs. External LBs use a global address resource to make a public IP address that clients from the internet can use to access the external LB. Internal LBs do not need a global address. In addiiton, internal LBs use the "google_compute_target_http_proxy" proxy resource, external LBs use the "google_compute_global_forwarding_rule" resource. 

Below is a diagram illustrating the flow of traffic through the Terraform resources that make an internal load balancer: 
![](/tf-ilb.jpg)

And this is the flow of traffic through the Terraform resources that make an external load balancer: 
![](/tf-xlb.jpg)