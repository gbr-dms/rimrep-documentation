# Observability

This document provides an overview of how we leverage [New Relic](https://www.newrelic.com) for monitoring and observing our environments, ensuring the reliability and performance of our services.

## Summary

New Relic is an observability platform that enables us to monitor, debug, and optimize our applications and infrastructure in real-time. By using New Relic, we gain comprehensive insights into the health of our environments and can proactively address potential issues before they affect our users.

### Key Features

- **Real-time Analytics**: New Relic provides real-time insights into application performance, user experience, and business metrics, enabling us to make data-driven decisions quickly.
- **APM (Application Performance Monitoring)**: It offers deep visibility into application performance, helping us identify bottlenecks and optimize response times.
- **Infrastructure Monitoring**: We monitor our cloud and on-premises environments to ensure they are operating efficiently and securely.
- **Custom Dashboards**: Several custom dashboards have been created to monitor different parts of our Kubernetes cluster, offering tailored views and insights specific to our project needs.

## Kubernetes on AWS EKS

Our environments are hosted on a Kubernetes cluster, managed via Amazon Web Services (AWS) Elastic Kubernetes Service (EKS). This setup provides us with a scalable and secure infrastructure to deploy and manage our applications.

## Monitoring with New Relic

We utilize New Relic to monitor various aspects of our Kubernetes cluster on AWS EKS. This includes tracking the performance of individual pods, services, and the overall health of the cluster. New Relic's integration with Kubernetes and AWS EKS allows us to have a holistic view of our infrastructure, ensuring high availability and performance.

## Alerts and Notifications

Important alerts regarding the health and performance of our environments are configured to be sent directly to Slack. This ensures that our team is promptly informed of any issues, allowing for quick resolution and minimal disruption to our services.

## Monitoring Flux GitOps with Weave GitOps UI

In addition to New Relic, we also use Weave GitOps UI to monitor our Flux GitOps processes. Weave GitOps provides a user-friendly interface to visualize and manage the continuous delivery and deployment processes powered by Flux. This integration allows us to:

- **Visualize Workflows**: Easily see the state of deployments and the health of our Kubernetes resources in real-time.
- **Audit Changes**: Track and audit changes made to our infrastructure as code, ensuring traceability and compliance.
- **Troubleshoot Issues**: Quickly identify and resolve issues with deployments or configurations, minimizing downtime and impact on our services.

The use of Weave GitOps UI complements our observability stack, providing additional layers of visibility and control over our GitOps workflows. This ensures that our infrastructure and applications are not only monitored for performance but also continuously and reliably deployed through GitOps practices.
