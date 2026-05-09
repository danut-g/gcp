# Associate Cloud Engineer v21.5 - Current Valid Question Set

Source PDF: `/Users/danutgornicioiu/learning/gcp/gcp/q&a/associate-cloud-engineerv21-5_compress.pdf`

This file keeps only questions that were not flagged as incorrect, ambiguous, extraction-broken, or tied to deprecated/legacy Google Cloud services, tools, or product names. Original PDF question numbers are preserved for traceability.

- Original questions: 269
- Kept questions: 161
- Removed questions: 108
- Removed original question numbers: 7-9, 13, 15, 24, 29, 32, 34, 37-38, 40, 42-43, 45, 47, 54-55, 60-61, 64-65, 69-72, 74-75, 79, 81, 84, 87-89, 92, 94, 96, 107, 109, 111, 113, 115, 119-121, 123, 126, 129-131, 137, 139, 141-142, 147, 149-152, 157, 160, 168, 171, 173-174, 176, 180-183, 190-192, 198, 201-205, 209, 212, 215, 219-220, 222, 224, 229-230, 233-234, 237, 241-243, 246-249, 254-257, 259-260, 264, 267-269

---

## Question 1 (PDF Q1)

You are working with a Cloud SQL MySQL database at your company. You need to retain a month-end copy of the database for three years for audit purposes. What should you do?

A. Save file automatic first-of-the- month backup for three years Store the backup file in an Archive class Cloud Storage bucket

B. Convert the automatic first-of-the-month backup to an export file Write the export file to a Coldline class Cloud Storage bucket

C. Set up an export job for the first of the month Write the export file to an Archive class Cloud Storage bucket

D. Set up an on-demand backup tor the first of the month Write the backup to an Archive class Cloud Storage bucket

---

## Question 2 (PDF Q2)

You have created a code snippet that should be triggered whenever a new file is uploaded to a Cloud Storage bucket. You want to deploy this code snippet. What should you do?

A. Use App Engine and configure Cloud Scheduler to trigger the application using Pub/Sub.

B. Use Cloud Functions and configure the bucket as a trigger resource.

C. Use Google Kubernetes Engine and configure a CronJob to trigger the application using Pub/Sub.

D. Use Dataflow as a batch job, and configure the bucket as a data source.

---

## Question 3 (PDF Q3)

You need to create a Compute Engine instance in a new project that doesn't exist yet. What should you do?

A. Using the Cloud SDK, create a new project, enable the Compute Engine API in that project, and then create the instance specifying your new project.

B. Enable the Compute Engine API in the Cloud Console, use the Cloud SDK to create the instance, and then use the --project flag to specify a new project.

C. Using the Cloud SDK, create the new instance, and use the --project flag to specify the new project. Answer yes when prompted by Cloud SDK to enable the Compute Engine API.

D. Enable the Compute Engine API in the Cloud Console. Go to the Compute Engine section of the Console to create a new instance, and look for the Create In A New Project option in the creation form.

---

## Question 4 (PDF Q4)

You want to configure 10 Compute Engine instances for availability when maintenance occurs. Your requirements state that these instances should attempt to automatically restart if they crash. Also, the instances should be highly available including during system maintenance. What should you do?

A. Create an instance template for the instances. Set the 'Automatic Restart' to on. Set the 'On-host maintenance' to Migrate VM instance. Add the instance template to an instance group.

B. Create an instance template for the instances. Set 'Automatic Restart' to off. Set 'On- host maintenance' to Terminate VM instances. Add the instance template to an instance group.

C. Create an instance group for the instances. Set the 'Autohealing' health check to healthy (HTTP).

D. Create an instance group for the instance. Verify that the 'Advanced creation options' setting for 'do not retry machine creation' is set to off.

---

## Question 5 (PDF Q5)

You are running an application on multiple virtual machines within a managed instance group and have autoscaling enabled. The autoscaling policy is configured so that additional instances are added to the group if the CPU utilization of instances goes above 80%. VMs are added until the instance group reaches its maximum limit of five VMs or until CPU utilization of instances lowers to 80%. The initial delay for HTTP health checks against the instances is set to 30 seconds. The virtual machine instances take around three minutes to become available for users. You observe that when the instance group autoscales, it adds more instances then necessary to support the levels of end-user traffic. You want to properly maintain instance group sizes when autoscaling. What should you do?

A. Set the maximum number of instances to 1.

B. Decrease the maximum number of instances to 3.

C. Use a TCP health check instead of an HTTP health check.

D. Increase the initial delay of the HTTP health check to 200 seconds.

---

## Question 6 (PDF Q6)

You want to host your video encoding software on Compute Engine. Your user base is growing rapidly, and users need to be able 3 to encode their videos at any time without interruption or CPU limitations. You must ensure that your encoding solution is highly available, and you want to follow Google-recommended practices to automate operations. What should you do?

A. Deploy your solution on multiple standalone Compute Engine instances, and increase the number of existing instances wnen CPU utilization on Cloud Monitoring reaches a certain threshold.

B. Deploy your solution on multiple standalone Compute Engine instances, and replace existing instances with high-CPU instances when CPU utilization on Cloud Monitoring reaches a certain threshold.

C. Deploy your solution to an instance group, and increase the number of available instances whenever you see high CPU utilization in Cloud Monitoring.

D. Deploy your solution to an instance group, and set the autoscaling based on CPU utilization.

---

## Question 7 (PDF Q10)

You are building an archival solution for your data warehouse and have selected Cloud Storage to archive your data. Your users need to be able to access this archived data once a quarter for some regulatory requirements. You want to select a cost-efficient option. Which storage option should you use?

A. Coldline Storage

B. Nearline Storage

C. Regional Storage

D. Multi-Regional Storage

---

## Question 8 (PDF Q11)

You recently received a new Google Cloud project with an attached billing account where you will work. You need to create instances, set firewalls, and store data in Cloud Storage. You want to follow Google-recommended practices. What should you do?

A. Use the gcloud CLI services enable cloudresourcemanager.googleapis.com command to enable all resources.

B. Use the gcloud services enable compute.googleapis.com command to enable Compute Engine and the gcloud services enable storage-api.googleapis.com command to enable the Cloud Storage APIs.

C. Open the Google Cloud console and enable all Google Cloud APIs from the API dashboard.

D. Open the Google Cloud console and run gcloud init --project <project-id> in a Cloud Shell.

---

## Question 9 (PDF Q12)

You are storing sensitive information in a Cloud Storage bucket. For legal reasons, you need to be able to record all requests that read any of the stored data. You want to make sure you comply with these requirements. What should you do?

A. Enable the Identity Aware Proxy API on the project.

B. Scan the bucker using the Data Loss Prevention API.

C. Allow only a single Service Account access to read the data.

D. Enable Data Access audit logs for the Cloud Storage API.

---

## Question 10 (PDF Q14)

Several employees at your company have been creating projects with Cloud Platform and paying for it with their personal credit cards, which the company reimburses. The company wants to centralize all these projects under a single, new billing account. What should you do?

A. Contact cloud-billing@google.com with your bank account details and request a corporate billing account for your company.

B. Create a ticket with Google Support and wait for their call to share your credit card details over the phone.

C. In the Google Platform Console, go to the Resource Manage and move all projects to the root Organization.

D. In the Google Cloud Platform Console, create a new billing account and set up a payment method.

---

## Question 11 (PDF Q16)

Your company has workloads running on Compute Engine and on-premises. The Google Cloud Virtual Private Cloud (VPC) is connected to your WAN over a Virtual Private Network (VPN). You need to deploy a new Compute Engine instance and ensure that no public Internet traffic can be routed to it. What should you do?

A. Create the instance without a public IP address.

B. Create the instance with Private Google Access enabled.

C. Create a deny-all egress firewall rule on the VPC network.

D. Create a route on the VPC to route all traffic to the instance over the VPN tunnel.

---

## Question 12 (PDF Q17)

Your team is building a website that handles votes from a large user population. The incoming votes will arrive at various rates. You want to optimize the storage and processing of the votes. What should you do?

A. Save the incoming votes to Firestore. Use Cloud Scheduler to trigger a Cloud Functions instance to periodically process the votes.

B. Use a dedicated instance to process the incoming votes. Send the votes directly to this instance.

C. Save the incoming votes to a JSON file on Cloud Storage. Process the votes in a batch at the end of the day.

D. Save the incoming votes to Pub/Sub. Use the Pub/Sub topic to trigger a Cloud Functions instance to process the votes.

---

## Question 13 (PDF Q18)

You want to add a new auditor to a Google Cloud Platform project. The auditor should be allowed to read, but not modify, all project items. How should you configure the auditor's permissions?

A. Create a custom role with view-only project permissions. Add the user's account to the custom role.

B. Create a custom role with view-only service permissions. Add the user's account to the custom role.

C. Select the built-in IAM project Viewer role. Add the user's account to this role.

D. Select the built-in IAM service Viewer role. Add the user's account to this role.

---

## Question 14 (PDF Q19)

You want to run a single caching HTTP reverse proxy on GCP for a latency-sensitive website. This specific reverse proxy consumes almost no CPU. You want to have a 30-GB in-memory cache, and need an additional 2 GB of memory for the rest of the processes. You want to minimize cost. How should you run this reverse proxy?

A. Create a Cloud Memorystore for Redis instance with 32-GB capacity.

B. Run it on Compute Engine, and choose a custom instance type with 6 vCPUs and 32 GB of memory.

C. Package it in a container image, and run it on Kubernetes Engine, using n1-standard-32 instances as nodes.

D. Run it on Compute Engine, choose the instance type n1-standard-1, and add an SSD persistent disk of 32 GB.


---

## Question 15 (PDF Q20)

You have an application that receives SSL-encrypted TCP traffic on port 443. Clients for this application are located all over the world. You want to minimize latency for the clients. Which load balancing option should you use?

A. HTTPS Load Balancer

B. Network Load Balancer

C. SSL Proxy Load Balancer

D. Internal TCP/UDP Load Balancer. Add a firewall rule allowing ingress traffic from 0.0.0.0/0 on the target instances.

---

## Question 16 (PDF Q21)

Your organization has a dedicated person who creates and manages all service accounts for Google Cloud projects. You need to assign this person the minimum role for projects. What should you do?

A. Add the user to roles/iam.roleAdmin role.

B. Add the user to roles/iam.securityAdmin role.

C. Add the user to roles/iam.serviceAccountUser role.

D. Add the user to roles/iam.serviceAccountAdmin role.

---

## Question 17 (PDF Q22)

Users of your application are complaining of slowness when loading the application. You realize the slowness is because the App Engine deployment serving the application is deployed in us-central whereas all users of this application are closest to europe-west3. You want to change the region of the App Engine application to europe-west3 to minimize latency. What's the best way to change the App Engine region?

A. Create a new project and create an App Engine instance in europe-west3

B. Use the gcloud app region set command and supply the name of the new region.

C. From the console, under the App Engine page, click edit, and change the region drop- down.

D. Contact Google Cloud Support and request the change.

---

## Question 18 (PDF Q23)

Your team has developed a stateless application which requires it to be run directly on virtual machines. The application is expected to receive a fluctuating amount of traffic and needs to scale automatically. You need to deploy the application. What should you do?

A. Deploy the application on a managed instance group and configure autoscaling.

B. Deploy the application on a Kubernetes Engine cluster and configure node pool autoscaling.

C. Deploy the application on Cloud Functions and configure the maximum number instances.

D. Deploy the application on Cloud Run and configure autoscaling.

---

## Question 19 (PDF Q25)

You are working for a startup that was officially registered as a business 6 months ago. As your customer base grows, your use of Google Cloud increases. You want to allow all engineers to create new projects without asking them for their credit card information. What should you do?

A. Create a Billing account, associate a payment method with it, and provide all project creators with permission to associate that billing account with their projects.

B. Grant all engineer's permission to create their own billing accounts for each new project.

C. Apply for monthly invoiced billing, and have a single invoice tor the project paid by the finance team.

D. Create a billing account, associate it with a monthly purchase order (PO), and send the PO to Google Cloud.

---

## Question 20 (PDF Q26)

Your VMs are running in a subnet that has a subnet mask of 255.255.255.240. The current subnet has no more free IP addresses and you require an additional 10 IP addresses for new VMs. The existing and new VMs should all be able to reach each other without additional routes. What should you do?

A. Use gcloud to expand the IP range of the current subnet.

B. Delete the subnet, and recreate it using a wider range of IP addresses.

C. Create a new project. Use Shared VPC to share the current network with the new project.

D. Create a new subnet with the same starting IP but a wider range to overwrite the current subnet.

---

## Question 21 (PDF Q27)

You have an application that looks for its licensing server on the IP 10.0.3.21. You need to deploy the licensing server on Compute Engine. You do not want to change the configuration of the application and want the application to be able to reach the licensing server. What should you do?

A. Reserve the IP 10.0.3.21 as a static internal IP address using gcloud and assign it to the licensing server.

B. Reserve the IP 10.0.3.21 as a static public IP address using gcloud and assign it to the licensing server.

C. Use the IP 10.0.3.21 as a custom ephemeral IP address and assign it to the licensing server.

D. Start the licensing server with an automatic ephemeral IP address, and then promote it to a static internal IP address.

---

## Question 22 (PDF Q28)

You need to set up permissions for a set of Compute Engine instances to enable them to write data into a particular Cloud Storage bucket. You want to follow Google-recommended practices. What should you do?

A. Create a service account with an access scope. Use the access scope 'https://www.googleapis.com/auth/devstorage.write_only'.

B. Create a service account with an access scope. Use the access scope 'https://www.googleapis.com/auth/cloud-platform'.

C. Create a service account and add it to the IAM role 'storage.objectCreator' for that bucket.

D. Create a service account and add it to the IAM role 'storage.objectAdmin' for that bucket.

---

## Question 23 (PDF Q30)

Your existing application running in Google Kubernetes Engine (GKE) consists of multiple pods running on four GKE n1-standard-2 nodes. You need to deploy additional pods requiring n2-highmem-16 nodes without any downtime. What should you do?

A. Use gcloud container clusters upgrade. Deploy the new services.

B. Create a new Node Pool and specify machine type n2-highmem-16. Deploy the new pods.

C. Create a new cluster with n2-highmem-16 nodes. Redeploy the pods and delete the old cluster.

D. Create a new cluster with both n1-standard-2 and n2-highmem-16 nodes. Redeploy the pods and delete the old cluster.

---

## Question 24 (PDF Q31)

Your company has multiple projects linked to a single billing account in Google Cloud. You need to visualize the costs with specific metrics that should be dynamically calculated based on company-specific criteria. You want to automate the process. What should you do?

A. In the Google Cloud console, visualize the costs related to the projects in the Reports section.

B. In the Google Cloud console, visualize the costs related to the projects in the Cost breakdown section.

C. In the Google Cloud console, use the export functionality of the Cost table. Create a Looker Studiodashboard on top of the CSV export.

D. Configure Cloud Billing data export to BigQuery for the billing account. Create a Looker Studio dashboard on top of the BigQuery export.

---

## Question 25 (PDF Q33)

You are deploying an application to a Compute Engine VM in a managed instance group. The application must be running at all times, but only a single instance of the VM should run per GCP project. How should you configure the instance group?

A. Set autoscaling to On, set the minimum number of instances to 1, and then set the maximum number of instances to 1.

B. Set autoscaling to Off, set the minimum number of instances to 1, and then set the maximum number of instances to 1.

C. Set autoscaling to On, set the minimum number of instances to 1, and then set the maximum number of instances to 2.

D. Set autoscaling to Off, set the minimum number of instances to 1, and then set the maximum number of instances to 2.

---

## Question 26 (PDF Q35)

You need to provide a cost estimate for a Kubernetes cluster using the GCP pricing calculator for Kubernetes. Your workload requires high IOPs, and you will also be using disk snapshots. You start by entering the number of nodes, average hours, and average days. What should you do next?

A. Fill in local SSD. Fill in persistent disk storage and snapshot storage.

B. Fill in local SSD. Add estimated cost for cluster management.

C. Select Add GPUs. Fill in persistent disk storage and snapshot storage.

D. Select Add GPUs. Add estimated cost for cluster management.

---

## Question 27 (PDF Q36)

You have an application that uses Cloud Spanner as a database backend to keep current state information about users. Cloud Bigtable logs all events triggered by users. You export Cloud Spanner data to Cloud Storage during daily backups. One of your analysts asks you to join data from Cloud Spanner and Cloud Bigtable for specific users. You want to complete this ad hoc request as efficiently as possible. What should you do?

A. Create a dataflow job that copies data from Cloud Bigtable and Cloud Storage for specific users.

B. Create a dataflow job that copies data from Cloud Bigtable and Cloud Spanner for specific users.

C. Create a Cloud Dataproc cluster that runs a Spark job to extract data from Cloud Bigtable and Cloud Storage for specific users.

D. Create two separate BigQuery external tables on Cloud Storage and Cloud Bigtable. Use the BigQuery console to join these tables through user fields, and apply appropriate filters.

---

## Question 28 (PDF Q39)

You want to deploy an application on Cloud Run that processes messages from a Cloud Pub/Sub topic. You want to follow Google-recommended practices. What should you do?

A. 1. Create a Cloud Function that uses a Cloud Pub/Sub trigger on that topic.2. Call your application on Cloud Run from the Cloud Function for every message.

B. 1. Grant the Pub/Sub Subscriber role to the service account used by Cloud Run.2. Create a Cloud Pub/Sub subscription for that topic.3. Make your application pull messages from that subscription.

C. 1. Create a service account.2. Give the Cloud Run Invoker role to that service account for your Cloud Run application.3. Create a Cloud Pub/Sub subscription that uses that service account and uses your Cloud Run application as the push endpoint.

D. 1. Deploy your application on Cloud Run on GKE with the connectivity set to Internal.2. Create a Cloud Pub/Sub subscription for that topic.3. In the same Google Kubernetes Engine cluster as your application, deploy a container that takes the messages and sends them to your application.

---

## Question 29 (PDF Q41)

You have a development project with appropriate IAM roles defined. You are creating a production project and want to have the same IAM roles on the new project, using the fewest possible steps. What should you do?

A. Use gcloud iam roles copy and specify the production project as the destination project.

B. Use gcloud iam roles copy and specify your organization as the destination organization.

C. In the Google Cloud Platform Console, use the 'create role from role' functionality.

D. In the Google Cloud Platform Console, use the 'create role' functionality and select all applicable permissions.

---

## Question 30 (PDF Q44)

You need to produce a list of the enabled Google Cloud Platform APIs for a GCP project using the gcloud command line in the Cloud Shell. The project name is my-project. What should you do?

A. Run gcloud projects list to get the project ID, and then run gcloud services list --project <project ID>.

B. Run gcloud init to set the current project to my-project, and then run gcloud services list --available.

C. Run gcloud info to view the account value, and then run gcloud services list --account <Account>.

D. Run gcloud projects describe <project ID> to verify the project value, and then run gcloud services list --available.

---

## Question 31 (PDF Q46)

You are hosting an application on bare-metal servers in your own data center. The application needs access to Cloud Storage. However, security policies prevent the servers hosting the application from having public IP addresses or access to the internet. You want to follow Google-recommended practices to provide the application with access to Cloud Storage. What should you do?

A. 1. Use nslookup to get the IP address for storage.googleapis.com.2. Negotiate with the security team to be able to give a public IP address to the servers.3. Only allow egress traffic from those servers to the IP addresses for storage.googleapis.com.

B. 1. Using Cloud VPN, create a VPN tunnel to a Virtual Private Cloud (VPC) in Google Cloud Platform (GCP).2. In this VPC, create a Compute Engine instance and install the Squid proxy server on this instance.3. Configure your servers to use that instance as a proxy to access Cloud Storage.

C. 1. Use Migrate for Compute Engine (formerly known as Velostrata) to migrate those servers to Compute Engine.2. Create an internal load balancer (ILB) that uses storage.googleapis.com as backend.3. Configure your new instances to use this ILB as proxy.

D. 1. Using Cloud VPN or Interconnect, create a tunnel to a VPC in GCP.2. Use Cloud Router to create a custom route advertisement for 199.36.153.4/30. Announce that network to your on-premises network through the VPN tunnel.3. In your on-premises network, configure your DNS server to resolve *.googleapis.com as a CNAME to restricted.googleapis.com.

---

## Question 32 (PDF Q48)

You are developing a new application and are looking for a Jenkins installation to build and deploy your source code. You want to automate the installation as quickly and easily as possible. What should you do?

A. Deploy Jenkins through the Google Cloud Marketplace.

B. Create a new Compute Engine instance. Run the Jenkins executable.

C. Create a new Kubernetes Engine cluster. Create a deployment for the Jenkins image.

D. Create an instance template with the Jenkins executable. Create a managed instance group with this template.

---

## Question 33 (PDF Q49)

You will have several applications running on different Compute Engine instances in the same project. You want to specify at a more granular level the service account each instance uses when calling Google Cloud APIs. What should you do?

A. When creating the instances, specify a Service Account for each instance

B. When creating the instances, assign the name of each Service Account as instance metadata

C. After starting the instances, use gcloud compute instances update to specify a Service Account for each instance

D. After starting the instances, use gcloud compute instances update to assign the name of the relevant Service Account as instance metadata

---

## Question 34 (PDF Q50)

You are using Google Kubernetes Engine with autoscaling enabled to host a new application. You want to expose this new application to the public, using HTTPS on a public IP address. What should you do?

A. Create a Kubernetes Service of type NodePort for your application, and a Kubernetes Ingress to expose this Service via a Cloud Load Balancer.

B. Create a Kubernetes Service of type ClusterIP for your application. Configure the public DNS name of your application using the IP of this Service.

C. Create a Kubernetes Service of type NodePort to expose the application on port 443 of each node of the Kubernetes cluster. Configure the public DNS name of your application with the IP of every node of the cluster to achieve load-balancing.

D. Create a HAProxy pod in the cluster to load-balance the traffic to all the pods of the application. Forward the public traffic to HAProxy with an iptable rule. Configure the DNS name of your application using the public IP of the node HAProxy is running on.

---

## Question 35 (PDF Q51)

You recently deployed a new version of an application to App Engine and then discovered a bug in the release. You need to immediately revert to the prior version of the application. What should you do?

A. Run gcloud app restore.

B. On the App Engine page of the GCP Console, select the application that needs to be reverted and click Revert.

C. On the App Engine Versions page of the GCP Console, route 100% of the traffic to the previous version.

D. Deploy the original version as a separate application. Then go to App Engine settings and split traffic between applications so that the original version serves 100% of the requests.

---

## Question 36 (PDF Q52)

You want to configure autohealing for network load balancing for a group of Compute Engine instances that run in multiple zones, using the fewest possible steps. You need to configure re-creation of VMs if they are unresponsive after 3 attempts of 10 seconds each. What should you do?

A. Create an HTTP load balancer with a backend configuration that references an existing instance group. Set the health check to healthy (HTTP).

B. Create an HTTP load balancer with a backend configuration that references an existing instance group. Define a balancing mode and set the maximum RPS to 10.

C. Create a managed instance group. Set the Autohealing health check to healthy (HTTP).

D. Create a managed instance group. Verify that the autoscaling setting is on.

---

## Question 37 (PDF Q53)

You created an instance of SQL Server 2017 on Compute Engine to test features in the new version. You want to connect to this instance using the fewest number of steps. What should you do?

A. Install a RDP client on your desktop. Verify that a firewall rule for port 3389 exists.

B. Install a RDP client in your desktop. Set a Windows username and password in the GCP Console. Use the credentials to log in to the instance.

C. Set a Windows password in the GCP Console. Verify that a firewall rule for port 22 exists. Click the RDP button in the GCP Console and supply the credentials to log in.

D. Set a Windows username and password in the GCP Console. Verify that a firewall rule for port 3389 exists. Click the RDP button in the GCP Console, and supply the credentials to log in.

---

## Question 38 (PDF Q56)

You are configuring Cloud DNS. You want !to create DNS records to point home.mydomain.com, mydomain.com. and www.mydomain.com to the IP address of your Google Cloud load balancer. What should you do?

A. Create one CNAME record to point mydomain.com to the load balancer, and create two A records to point WWW and HOME lo mydomain.com respectively.

B. Create one CNAME record to point mydomain.com to the load balancer, and create two AAAA records to point WWW and HOME to mydomain.com respectively.

C. Create one A record to point mydomain.com to the load balancer, and create two CNAME records to point WWW and HOME to mydomain.com respectively.

D. Create one A record to point mydomain.com lo the load balancer, and create two NS records to point WWW and HOME to mydomain.com respectively.

---

## Question 39 (PDF Q57)

During a recent audit of your existing Google Cloud resources, you discovered several users with email addresses outside of your Google Workspace domain. You want to ensure that your resources are only shared with users whose email addresses match your domain. You need to remove any mismatched users, and you want to avoid having to audit your resources to identify mismatched users. What should you do?

A. Create a Cloud Scheduler task to regularly scan your projects and delete mismatched users.

B. Create a Cloud Scheduler task to regularly scan your resources and delete mismatched users.

C. Set an organizational policy constraint to limit identities by domain to automatically remove mismatched users.

D. Set an organizational policy constraint to limit identities by domain, and then retroactively remove the existing mismatched users.

---

## Question 40 (PDF Q58)

Your company's security vulnerability management policy wonts 3 member of the security team to have visibility into vulnerabilities and other OS metadata for a specific Compute Engine instance This Compute Engine instance hosts a critical application in your Goggle Cloud project. You need to implement your company's security vulnerability management policy. What should you dc?

A. - Ensure that the Ops Agent Is Installed on the Compute Engine instance. - Create a custom metric in the Cloud Monitoring dashboard. - Provide the security team member with access to this dashboard.

B. - Ensure that the Ops Agent is installed on tie Compute Engine instance. - Provide the security team member roles/configure.inventoryViewer permission.

C. - Ensure that the OS Config agent Is Installed on the Compute Engine instance. - Provide the security team member roles/configure.vulnerabilityViewer permission.

D. - Ensure that the OS Config agent is installed on the Compute Engine instance - Create a log sink Co a BigQuery dataset. - Provide the security team member with access to this dataset.

---

## Question 41 (PDF Q59)

You want to select and configure a cost-effective solution for relational data on Google Cloud Platform. You are working with a small set of operational data in one geographic location. You need to support point-in-time recovery. What should you do?

A. Select Cloud SQL (MySQL). Verify that the enable binary logging option is selected.

B. Select Cloud SQL (MySQL). Select the create failover replicas option.

C. Select Cloud Spanner. Set up your instance with 2 nodes.

D. Select Cloud Spanner. Set up your instance as multi-regional.

---

## Question 42 (PDF Q62)

Your development team needs a new Jenkins server for their project. You need to deploy the server using the fewest steps possible. What should you do?

A. Download and deploy the Jenkins Java WAR to App Engine Standard.

B. Create a new Compute Engine instance and install Jenkins through the command line interface.

C. Create a Kubernetes cluster on Compute Engine and create a deployment with the Jenkins Docker image.

D. Use GCP Marketplace to launch the Jenkins solution.

---

## Question 43 (PDF Q63)

You created a Google Cloud Platform project with an App Engine application inside the project. You initially configured the application to be served from the us-central region. Now you want the application to be served from the asia-northeast1 region. What should you do?

A. Change the default region property setting in the existing GCP project to asia- northeast1.

B. Change the region property setting in the existing App Engine application from us- central to asia-northeast1.

C. Create a second App Engine application in the existing GCP project and specify asia- northeast1 as the region to serve your application.

D. Create a new GCP project and create an App Engine application inside this new project. Specify asia-northeast1 as the region to serve your application.

---

## Question 44 (PDF Q66)

You are the organization and billing administrator for your company. The engineering team has the Project Creator role on the organization. You do not want the engineering team to be able to link projects to the billing account. Only the finance team should be able to link a project to a billing account, but they should not be able to make any other changes to projects. What should you do?

A. Assign the finance team only the Billing Account User role on the billing account.

B. Assign the engineering team only the Billing Account User role on the billing account.

C. Assign the finance team the Billing Account User role on the billing account and the Project Billing Manager role on the organization.

D. Assign the engineering team the Billing Account User role on the billing account and the Project Billing Manager role on the organization.

---

## Question 45 (PDF Q67)

You just installed the Google Cloud CLI on your new corporate laptop. You need to list the existing instances of your company on Google Cloud. What must you do before you run the gcloud compute instances list command? Choose 2 answers

A. Run gcloud auth login, enter your login credentials in the dialog window, and paste the received login token to gcloud CLI.

B. Create a Google Cloud service account, and download the service account key. Place the key file in a folder on your machine where gcloud CLI can find it.

C. Download your Cloud Identity user account key. Place the key file in a folder on your machine where gcloud CLI can find it.

D. Run gcloud config set compute/zone $my_zone to set the default zone for gcloud CLI.

E. Run gcloud config set project $my_project to set the default project for gcloud CLI.

---

## Question 46 (PDF Q68)

You create a new Google Kubernetes Engine (GKE) cluster and want to make sure that it always runs a supported and stable version of Kubernetes. What should you do?

A. Enable the Node Auto-Repair feature for your GKE cluster.

B. Enable the Node Auto-Upgrades feature for your GKE cluster.

C. Select the latest available cluster version for your GKE cluster.

D. Select "Container-Optimized OS (cos)" as a node image for your GKE cluster.

---

## Question 47 (PDF Q73)

You need to create an autoscaling managed instance group for an HTTPS web application. You want to make sure that unhealthy VMs are recreated. What should you do?

A. Create a health check on port 443 and use that when creating the Managed Instance Group.

B. Select Multi-Zone instead of Single-Zone when creating the Managed Instance Group.

C. In the Instance Template, add the label 'health-check'.

D. In the Instance Template, add a startup script that sends a heartbeat to the metadata server.

---

## Question 48 (PDF Q76)

You are operating a Google Kubernetes Engine (GKE) cluster for your company where different teams can run non-production workloads. Your Machine Learning (ML) team needs access to Nvidia Tesla P100 GPUs to train their models. You want to minimize effort and cost. What should you do?

A. Ask your ML team to add the "accelerator: gpu" annotation to their pod specification.

B. Recreate all the nodes of the GKE cluster to enable GPUs on all of them.

C. Create your own Kubernetes cluster on top of Compute Engine with nodes that have GPUs. Dedicate this cluster to your ML team.

D. Add a new, GPU-enabled, node pool to the GKE cluster. Ask your ML team to add the cloud.google.com/gke -accelerator: nvidia-tesla-p100 nodeSelector to their pod specification.

---

## Question 49 (PDF Q77)

You need to create a copy of a custom Compute Engine virtual machine (VM) to facilitate an expected increase in application traffic due to a business acquisition. What should you do?

A. Create a Compute Engine snapshot of your base VM. Create your images from that snapshot.

B. Create a Compute Engine snapshot of your base VM. Create your instances from that snapshot.

C. Create a custom Compute Engine image from a snapshot. Create your images from that image.

D. Create a custom Compute Engine image from a snapshot. Create your instances from that image.

---

## Question 50 (PDF Q78)

You have downloaded and installed the gcloud command line interface (CLI) and have authenticated with your Google Account. Most of your Compute Engine instances in your project run in the europe-west1-d zone. You want to avoid having to specify this zone with each CLI command when managing these instances. What should you do?

A. Set the europe-west1-d zone as the default zone using the gcloud config subcommand.

B. In the Settings page for Compute Engine under Default location, set the zone to europe-west1-d.

C. In the CLI installation directory, create a file called default.conf containing zone=europe-west1-d.

D. Create a Metadata entry on the Compute Engine page with key compute/zone and value europe-west1-d.

---

## Question 51 (PDF Q80)

You need to assign a Cloud Identity and Access Management (Cloud IAM) role to an external auditor. The auditor needs to have permissions to review your Google Cloud Platform (GCP) Audit Logs and also to review your Data Access logs. What should you do?

A. Assign the auditor the IAM role roles/logging.privateLogViewer. Perform the export of logs to Cloud Storage.

B. Assign the auditor the IAM role roles/logging.privateLogViewer. Direct the auditor to also review the logs for changes to Cloud IAM policy.

C. Assign the auditor's IAM user to a custom role that has logging.privateLogEntries.list permission. Perform the export of logs to Cloud Storage.

D. Assign the auditor's IAM user to a custom role that has logging.privateLogEntries.list permission. Direct the auditor to also review the logs for changes to Cloud IAM policy.

---

## Question 52 (PDF Q82)

Your management has asked an external auditor to review all the resources in a specific project. The security team has enabled the Organization Policy called Domain Restricted Sharing on the organization node by specifying only your Cloud Identity domain. You want the auditor to only be able to view, but not modify, the resources in that project. What should you do?

A. Ask the auditor for their Google account, and give them the Viewer role on the project.

B. Ask the auditor for their Google account, and give them the Security Reviewer role on the project.

C. Create a temporary account for the auditor in Cloud Identity, and give that account the Viewer role on the project.

D. Create a temporary account for the auditor in Cloud Identity, and give that account the Security Reviewer role on the project.

---

## Question 53 (PDF Q83)

You created a Kubernetes deployment by running kubectl run nginx image=nginx labels=app=prod. Your Kubernetes cluster is also used by a number of other deployments. How can you find the identifier of the pods for this nginx deployment?

A. kubectl get deployments -output=pods

B. gcloud get pods -selector="app=prod"

C. kubectl get pods -I "app=prod"

D. gcloud list gke-deployments -filter={pod }

---

## Question 54 (PDF Q85)

You need to configure optimal data storage for files stored in Cloud Storage for minimal cost. The files are used in a mission-critical analytics pipeline that is used continually. The users are in Boston, MA (United States). What should you do?

A. Configure regional storage for the region closest to the users Configure a Nearline storage class

B. Configure regional storage for the region closest to the users Configure a Standard storage class

C. Configure dual-regional storage for the dual region closest to the users Configure a Nearline storage class

D. Configure dual-regional storage for the dual region closest to the users Configure a Standard storage class

---

## Question 55 (PDF Q86)

You are performing a monthly security check of your Google Cloud environment and want to know who has access to view data stored in your Google Cloud Project. What should you do?

A. Enable Audit Logs for all APIs that are related to data storage.

B. Review the IAM permissions for any role that allows for data access.

C. Review the Identity-Aware Proxy settings for each resource.

D. Create a Data Loss Prevention job.

---

## Question 56 (PDF Q90)

Your company has developed a new application that consists of multiple microservices. You want to deploy the application to Google Kubernetes Engine (GKE), and you want to ensure that the cluster can scale as more applications are deployed in the future. You want to avoid manual intervention when each new application is deployed. What should you do?

A. Deploy the application on GKE, and add a HorizontalPodAutoscaler to the deployment.

B. Deploy the application on GKE, and add a VerticalPodAutoscaler to the deployment.

C. Create a GKE cluster with autoscaling enabled on the node pool. Set a minimum and maximum for the size of the node pool.

D. Create a separate node pool for each application, and deploy each application to its dedicated node pool.

---

## Question 57 (PDF Q91)

You have successfully created a development environment in a project for an application. This application uses Compute Engine and Cloud SQL. Now, you need to create a production environment for this application. The security team has forbidden the existence of network routes between these 2 environments, and asks you to follow Google-recommended practices. What should you do?

A. Create a new project, enable the Compute Engine and Cloud SQL APIs in that project, and replicate the setup you have created in the development environment.

B. Create a new production subnet in the existing VPC and a new production Cloud SQL instance in your existing project, and deploy your application using those resources.

C. Create a new project, modify your existing VPC to be a Shared VPC, share that VPC with your new project, and replicate the setup you have in the development environment in that new project, in the Shared VPC.

D. Ask the security team to grant you the Project Editor role in an existing production project used by another division of your company. Once they grant you that role, replicate the setup you have in the development environment in that project.

---

## Question 58 (PDF Q93)

Your company implemented BigQuery as an enterprise data warehouse. Users from multiple business units run queries on this data warehouse. However, you notice that query costs for BigQuery are very high, and you need to control costs. Which two methods should you use? (Choose two.)

A. Split the users from business units to multiple projects.

B. Apply a user- or project-level custom query quota for BigQuery data warehouse.

C. Create separate copies of your BigQuery data warehouse for each business unit.

D. Split your BigQuery data warehouse into multiple data warehouses for each business unit.

E. Change your BigQuery query model from on-demand to flat rate. Apply the appropriate number of slots to each Project.

---

## Question 59 (PDF Q95)

Your organization has strict requirements to control access to Google Cloud projects. You need to enable your Site Reliability Engineers (SREs) to approve requests from the Google Cloud support team when an SRE opens a support case. You want to follow Google- recommended practices. What should you do?

A. Add your SREs to roles/iam.roleAdmin role.

B. Add your SREs to roles/accessapproval approver role.

C. Add your SREs to a group and then add this group to roles/iam roleAdmin role.

D. Add your SREs to a group and then add this group to roles/accessapproval approver role.

---

## Question 60 (PDF Q97)

You deployed a new application inside your Google Kubernetes Engine cluster using the YAML file specified below. You check the status of the deployed pods and notice that one of them is still in PENDING status: You want to find out why the pod is stuck in pending status. What should you do?

A. Review details of the myapp-service Service object and check for error messages.

B. Review details of the myapp-deployment Deployment object and check for error messages.

C. Review details of myapp-deployment-58ddbbb995-lp86m Pod and check for warning messages.

D. View logs of the container in myapp-deployment-58ddbbb995-lp86m pod and check for warning messages.

---

## Question 61 (PDF Q98)

You have an instance group that you want to load balance. You want the load balancer to terminate the client SSL session. The instance group is used to serve a public web application over HTTPS. You want to follow Google-recommended practices. What should you do?

A. Configure an HTTP(S) load balancer.

B. Configure an internal TCP load balancer.

C. Configure an external SSL proxy load balancer.

D. Configure an external TCP proxy load balancer.

---

## Question 62 (PDF Q99)

Your company has a 3-tier solution running on Compute Engine. The configuration of the current infrastructure is shown below. Each tier has a service account that is associated with all instances within it. You need to enable communication on TCP port 8080 between tiers as follows: - Instances in tier #1 must communicate with tier #2. - Instances in tier #2 must communicate with tier #3. What should you do?

A. 1. Create an ingress firewall rule with the following settings:- Targets: all instances- Source filter: IP ranges (with the range set to 10.0.2.0/24)- Protocols: allow all2. Create an ingress firewall rule with the following settings:- Targets: all instances- Source filter: IP ranges (with the range set to 10.0.1.0/24)- Protocols: allow all

B. 1. Create an ingress firewall rule with the following settings:- Targets: all instances with tier #2 service account- Source filter: all instances with tier #1 service account- Protocols: allow TCP:80802. Create an ingress firewall rule with the following settings:- Targets: all instances with tier #3 service account- Source filter: all instances with tier #2 service account- Protocols: allow TCP: 8080

C. 1. Create an ingress firewall rule with the following settings:- Targets: all instances with tier #2 service account- Source filter: all instances with tier #1 service account- Protocols: allow all2. Create an ingress firewall rule with the following settings:- Targets: all instances with tier #3 service account- Source filter: all instances with tier #2 service account- Protocols: allow all

D. 1. Create an egress firewall rule with the following settings:- Targets: all instances- Source filter: IP ranges (with the range set to 10.0.2.0/24)- Protocols: allow TCP: 80802. Create an egress firewall rule with the following settings:- Targets: all instances- Source filter: IP ranges (with the range set to 10.0.1.0/24)- Protocols: allow TCP: 8080

---

## Question 63 (PDF Q100)

You are running a data warehouse on BigQuery. A partner company is offering a recommendation engine based on the data in your data warehouse. The partner company is also running their application on Google Cloud. They manage the resources in their own project, but they need access to the BigQuery dataset in your project. You want to provide the partner company with access to the dataset What should you do?

A. Create a Service Account in your own project, and grant this Service Account access to BigQuery in your project

B. Create a Service Account in your own project, and ask the partner to grant this Service Account access to BigQuery in their project

C. Ask the partner to create a Service Account in their project, and have them give the Service Account access to BigQuery in their project

D. Ask the partner to create a Service Account in their project, and grant their Service Account access to the BigQuery dataset in your project

---

## Question 64 (PDF Q101)

You want to configure a solution for archiving data in a Cloud Storage bucket. The solution must be cost-effective. Data with multiple versions should be archived after 30 days. Previous versions are accessed once a month for reporting. This archive data is also occasionally updated at month-end. What should you do?

A. Add a bucket lifecycle rule that archives data with newer versions after 30 days to Coldline Storage.

B. Add a bucket lifecycle rule that archives data with newer versions after 30 days to Nearline Storage.

C. Add a bucket lifecycle rule that archives data from regional storage after 30 days to Coldline Storage.

D. Add a bucket lifecycle rule that archives data from regional storage after 30 days to Nearline Storage.

---

## Question 65 (PDF Q102)

Your company developed a mobile game that is deployed on Google Cloud. Gamers are connecting to the game with their personal phones over the Internet. The game sends UDP packets to update the servers about the gamers' actions while they are playing in multiplayer mode. Your game backend can scale over multiple virtual machines (VMs), and you want to expose the VMs over a single IP address. What should you do?

A. Configure an SSL Proxy load balancer in front of the application servers.

B. Configure an Internal UDP load balancer in front of the application servers.

C. Configure an External HTTP(s) load balancer in front of the application servers.

D. Configure an External Network load balancer in front of the application servers.

---

## Question 66 (PDF Q103)

You need to create a custom IAM role for use with a GCP service. All permissions in the role must be suitable for production use. You also want to clearly share with your organization the status of the custom role. This will be the first version of the custom role. What should you do?

A. Use permissions in your role that use the 'supported' support level for role permissions. Set the role stage to ALPHA while testing the role permissions.

B. Use permissions in your role that use the 'supported' support level for role permissions. Set the role stage to BETA while testing the role permissions.

C. Use permissions in your role that use the 'testing' support level for role permissions. Set the role stage to ALPHA while testing the role permissions.

D. Use permissions in your role that use the 'testing' support level for role permissions. Set the role stage to BETA while testing the role permissions.

---

## Question 67 (PDF Q104)

You are working with a user to set up an application in a new VPC behind a firewall. The user is concerned about data egress. You want to configure the fewest open egress ports. What should you do?

A. Set up a low-priority (65534) rule that blocks all egress and a high-priority rule (1000) that allows only the appropriate ports.

B. Set up a high-priority (1000) rule that pairs both ingress and egress ports.

C. Set up a high-priority (1000) rule that blocks all egress and a low-priority (65534) rule that allows only the appropriate ports.

D. Set up a high-priority (1000) rule to allow the appropriate ports.

---

## Question 68 (PDF Q105)

A colleague handed over a Google Cloud Platform project for you to maintain. As part of a security checkup, you want to review who has been granted the Project Owner role. What should you do?

A. In the console, validate which SSH keys have been stored as project-wide keys.

B. Navigate to Identity-Aware Proxy and check the permissions for these resources.

C. Enable Audit Logs on the IAM & admin page for all resources, and validate the results.

D. Use the command gcloud projects get-iam-policy to view the current role assignments.

---

## Question 69 (PDF Q106)

You are deploying a web application using Compute Engine. You created a managed instance group (MIG) to host the application. You want to follow Google-recommended practices to implement a secure and highly available solution. What should you do?

A. Use SSL proxy load balancing for the MIG and an A record in your DNS private zone with the load balancer's IP address.

B. Use SSL proxy load balancing for the MIG and a CNAME record in your DNS public zone with the load balancer's IP address.

C. Use HTTP(S) load balancing for the MIG and a CNAME record in your DNS private zone with the load balancer's IP address.

D. Use HTTP(S) load balancing for the MIG and an A record in your DNS public zone with the load balancer's IP address.

---

## Question 70 (PDF Q108)

Your organization uses Active Directory (AD) to manage user identities. Each user uses this identity for federated access to various on-premises systems. Your security team has adopted a policy that requires users to log into Google Cloud with their AD identity instead of their own login. You want to follow the Google-recommended practices to implement this policy. What should you do?

A. Sync Identities with Cloud Directory Sync, and then enable SAML for single sign-on

B. Sync Identities in the Google Admin console, and then enable Oauth for single sign-on

C. Sync identities with 3rd party LDAP sync, and then copy passwords to allow simplified login with (he same credentials

D. Sync identities with Cloud Directory Sync, and then copy passwords to allow simplified login with the same credentials.

---

## Question 71 (PDF Q110)

You need to deploy an application, which is packaged in a container image, in a new project. The application exposes an HTTP endpoint and receives very few requests per day. You want to minimize costs. What should you do?

A. Deploy the container on Cloud Run.

B. Deploy the container on Cloud Run on GKE.

C. Deploy the container on App Engine Flexible.

D. Deploy the container on Google Kubernetes Engine, with cluster autoscaling and horizontal pod autoscaling enabled.

---

## Question 72 (PDF Q112)

You are developing a financial trading application that will be used globally. Data is stored and queried using a relational structure, and clients from all over the world should get the exact identical state of the data. The application will be deployed in multiple regions to provide the lowest latency to end users. You need to select a storage option for the application data while minimizing latency. What should you do?

A. Use Cloud Bigtable for data storage.

B. Use Cloud SQL for data storage.

C. Use Cloud Spanner for data storage.

D. Use Firestore for data storage.

---

## Question 73 (PDF Q114)

You want to verify the IAM users and roles assigned within a GCP project named my- project. What should you do?

A. Run gcloud iam roles list. Review the output section.

B. Run gcloud iam service-accounts list. Review the output section.

C. Navigate to the project and then to the IAM section in the GCP Console. Review the members and roles.

D. Navigate to the project and then to the Roles section in the GCP Console. Review the roles and status.

---

## Question 74 (PDF Q116)

You have a number of compute instances belonging to an unmanaged instances group. You need to SSH to one of the Compute Engine instances to run an ad hoc script. You've already authenticated gcloud, however, you don't have an SSH key deployed yet. In the fewest steps possible, what's the easiest way to SSH to the instance?

A. Run gcloud compute instances list to get the IP address of the instance, then use the ssh command.

B. Use the gcloud compute ssh command.

C. Create a key with the ssh-keygen command. Then use the gcloud compute ssh command.

D. Create a key with the ssh-keygen command. Upload the key to the instance. Run gcloud compute instances list to get the IP address of the instance, then use the ssh command.

---

## Question 75 (PDF Q117)

You are building a product on top of Google Kubernetes Engine (GKE). You have a single GKE cluster. For each of your customers, a Pod is running in that cluster, and your customers can run arbitrary code inside their Pod. You want to maximize the isolation between your customers' Pods. What should you do?

A. Use Binary Authorization and whitelist only the container images used by your customers' Pods.

B. Use the Container Analysis API to detect vulnerabilities in the containers used by your customers' Pods.

C. Create a GKE node pool with a sandbox type configured to gvisor. Add the parameter runtimeClassName: gvisor to the specification of your customers' Pods.

D. Use the cos_containerd image for your GKE nodes. Add a nodeSelector with the value cloud.google.com/gke-os-distribution: cos_containerd to the specification of your customers' Pods.

---

## Question 76 (PDF Q118)

You have been asked to set up Object Lifecycle Management for objects stored in storage buckets. The objects are written once and accessed frequently for 30 days. After 30 days, the objects are not read again unless there is a special need. The object should be kept for three years, and you need to minimize cost. What should you do?

A. Set up a policy that uses Nearline storage for 30 days and then moves to Archive storage for three years.

B. Set up a policy that uses Standard storage for 30 days and then moves to Archive storage for three years.

C. Set up a policy that uses Nearline storage for 30 days, then moves the Coldline for one year, and then moves to Archive storage for two years.

D. Set up a policy that uses Standard storage for 30 days, then moves to Coldline for one year, and then moves to Archive storage for two years.

---

## Question 77 (PDF Q122)

You are building an application that processes data files uploaded from thousands of suppliers. Your primary goals for the application are data security and the expiration of aged data. You need to design the application to: -Restrict access so that suppliers can access only their own data. -Give suppliers write access to data only for 30 minutes. -Delete data that is over 45 days old. You have a very short development cycle, and you need to make sure that the application requires minimal maintenance. Which two strategies should you use? (Choose two.)

A. Build a lifecycle policy to delete Cloud Storage objects after 45 days.

B. Use signed URLs to allow suppliers limited time access to store their objects.

C. Set up an SFTP server for your application, and create a separate user for each supplier.

D. Build a Cloud function that triggers a timer of 45 days to delete objects that have expired.

E. Develop a script that loops through all Cloud Storage buckets and deletes any buckets that are older than 45 days.

---

## Question 78 (PDF Q124)

Your company has an existing GCP organization with hundreds of projects and a billing account. Your company recently acquired another company that also has hundreds of projects and its own billing account. You would like to consolidate all GCP costs of both GCP organizations onto a single invoice. You would like to consolidate all costs as of tomorrow. What should you do?

A. Link the acquired company's projects to your company's billing account.

B. Configure the acquired company's billing account and your company's billing account to export the billing data into the same BigQuery dataset.

C. Migrate the acquired company's projects into your company's GCP organization. Link the migrated projects to your company's billing account.

D. Create a new GCP organization and a new billing account. Migrate the acquired company's projects and your company's projects into the new GCP organization and link the projects to the new billing account.

---

## Question 79 (PDF Q125)

Your team is running an on-premises ecommerce application. The application contains a complex set of microservices written in Python, and each microservice is running on Docker containers. Configurations are injected by using environment variables. You need to deploy your current application to a serverless Google Cloud cloud solution. What should you do?

A. Use your existing CI/CD pipeline Use the generated Docker images and deploy them to Cloud Run. Update the configurations and the required endpoints.

B. Use your existing continuous integration and delivery (CI/CD) pipeline. Use the generated Docker images and deploy them to Cloud Function. Use the same configuration as on-premises.

C. Use the existing codebase and deploy each service as a separate Cloud Function Update the configurations and the required endpoints.

D. Use your existing codebase and deploy each service as a separate Cloud Run Use the same configurations as on-premises.

---

## Question 80 (PDF Q127)

Your company is using Google Workspace to manage employee accounts. Anticipated growth will increase the number of personnel from 100 employees to 1.000 employees within 2 years. Most employees will need access to your company's Google Cloud account. The systems and processes will need to support 10x growth without performance degradation, unnecessary complexity, or security issues. What should you do?

A. Migrate the users to Active Directory. Connect the Human Resources system to Active Directory. Turn on Google Cloud Directory Sync (GCDS) for Cloud Identity. Turn on Identity Federation from Cloud Identity to Active Directory.

B. Organize the users in Cloud Identity into groups. Enforce multi-factor authentication in Cloud Identity.

C. Turn on identity federation between Cloud Identity and Google Workspace. Enforce multi-factor authentication for domain wide delegation.

D. Use a third-party identity provider service through federation. Synchronize the users from Google Workplace to the third-party provider in real time.

---

## Question 81 (PDF Q128)

You are creating a Google Kubernetes Engine (GKE) cluster with a cluster autoscaler feature enabled. You need to make sure that each node of the cluster will run a monitoring pod that sends container metrics to a third-party monitoring solution. What should you do?

A. Deploy the monitoring pod in a StatefulSet object.

B. Deploy the monitoring pod in a DaemonSet object.

C. Reference the monitoring pod in a Deployment object.

D. Reference the monitoring pod in a cluster initializer at the GKE cluster creation time.

---

## Question 82 (PDF Q132)

The DevOps group in your organization needs full control of Compute Engine resources in your development project. However, they should not have permission to create or update any other resources in the project. You want to follow Google's recommendations for setting permissions for the DevOps group. What should you do?

A. Grant the basic role roles/viewer and the predefined role roles/compute.admin to the DevOps group.

B. Create an IAM policy and grant all compute. instanceAdmln." permissions to the policy Attach the policy to the DevOps group.

C. Create a custom role at the folder level and grant all compute. instanceAdmln. * permissions to the role Grant the custom role to the DevOps group.

D. Grant the basic role roles/editor to the DevOps group.

---

## Question 83 (PDF Q133)

You have a Linux VM that must connect to Cloud SQL. You created a service account with the appropriate access rights. You want to make sure that the VM uses this service account instead of the default Compute Engine service account. What should you do?

A. When creating the VM via the web console, specify the service account under the 'Identity and API Access' section.

B. Download a JSON Private Key for the service account. On the Project Metadata, add that JSON as the value for the key compute-engine-service-account.

C. Download a JSON Private Key for the service account. On the Custom Metadata of the VM, add that JSON as the value for the key compute-engine-service-account.

D. Download a JSON Private Key for the service account. After creating the VM, ssh into the VM and save the JSON under ~/.gcloud/compute-engine-service-account.json.

---

## Question 84 (PDF Q134)

Your organization has user identities in Active Directory. Your organization wants to use Active Directory as their source of truth for identities. Your organization wants to have full control over the Google accounts used by employees for all Google services, including your Google Cloud Platform (GCP) organization. What should you do?

A. Use Google Cloud Directory Sync (GCDS) to synchronize users into Cloud Identity.

B. Use the cloud Identity APIs and write a script to synchronize users to Cloud Identity.

C. Export users from Active Directory as a CSV and import them to Cloud Identity via the Admin Console.

D. Ask each employee to create a Google account using self signup. Require that each employee use their company email address and password.

---

## Question 85 (PDF Q135)

You received a JSON file that contained a private key of a Service Account in order to get access to several resources in a Google Cloud project. You downloaded and installed the Cloud SDK and want to use this private key for authentication and authorization when performing gcloud commands. What should you do?

A. Use the command gcloud auth login and point it to the private key

B. Use the command gcloud auth activate-service-account and point it to the private key

C. Place the private key file in the installation directory of the Cloud SDK and rename it to "credentials ison"

D. Place the private key file in your home directory and rename it to ''GOOGLE_APPUCATION_CREDENTiALS".

---

## Question 86 (PDF Q136)

Your application development team has created Docker images for an application that will be deployed on Google Cloud. Your team does not want to manage the infrastructure associated with this application. You need to ensure that the application can scale automatically as it gains popularity. What should you do?

A. Create an Instance template with the container image, and deploy a Managed Instance Group with Autoscaling.

B. Upload Docker images to Artifact Registry, and deploy the application on Google Kubernetes Engine using Standard mode.

C. Upload Docker images to the Cloud Storage, and deploy the application on Google Kubernetes Engine using Standard mode.

D. Upload Docker images to Artifact Registry, and deploy the application on Cloud Run.

---

## Question 87 (PDF Q138)

You are running a web application on Cloud Run for a few hundred users. Some of your users complain that the initial web page of the application takes much longer to load than the following pages. You want to follow Google's recommendations to mitigate the issue. What should you do?

A. Update your web application to use the protocol HTTP/2 instead of HTTP/1.1

B. Set the concurrency number to 1 for your Cloud Run service.

C. Set the maximum number of instances for your Cloud Run service to 100.

D. Set the minimum number of instances for your Cloud Run service to 3.

---

## Question 88 (PDF Q140)

You have an on-premises data analytics set of binaries that processes data files in memory for about 45 minutes every midnight. The sizes of those data files range from 1 gigabyte to 16 gigabytes. You want to migrate this application to Google Cloud with minimal effort and cost. What should you do?

A. Upload the code to Cloud Functions. Use Cloud Scheduler to start the application.

B. Create a container for the set of binaries. Use Cloud Scheduler to start a Cloud Run job for the container.

C. Create a container for the set of binaries Deploy the container to Google Kubernetes Engine (GKE) and use the Kubernetes scheduler to start the application.

D. Lift and shift to a VM on Compute Engine. Use an instance schedule to start and stop the instance.

---

## Question 89 (PDF Q143)

You are assigned to maintain a Google Kubernetes Engine (GKE) cluster named dev that was deployed on Google Cloud. You want to manage the GKE configuration using the command line interface (CLI). You have just downloaded and installed the Cloud SDK. You want to ensure that future CLI commands by default address this specific cluster. What should you do?

A. Use the command gcloud config set container/cluster dev.

B. Use the command gcloud container clusters update dev.

C. Create a file called gke.default in the ~/.gcloud folder that contains the cluster name.

D. Create a file called defaults.json in the ~/.gcloud folder that contains the cluster name.

---

## Question 90 (PDF Q144)

Your company has embraced a hybrid cloud strategy where some of the applications are deployed on Google Cloud. A Virtual Private Network (VPN) tunnel connects your Virtual Private Cloud (VPC) in Google Cloud with your company's on-premises network. Multiple applications in Google Cloud need to connect to an on-premises database server, and you want to avoid having to change the IP configuration in all of your applications when the IP of the database changes. What should you do?

A. Configure Cloud NAT for all subnets of your VPC to be used when egressing from the VM instances.

B. Create a private zone on Cloud DNS, and configure the applications with the DNS name.

C. Configure the IP of the database as custom metadata for each instance, and query the metadata server.

D. Query the Compute Engine internal DNS from the applications to retrieve the IP of the database.

---

## Question 91 (PDF Q145)

You need to grant access for three users so that they can view and edit table data on a Cloud Spanner instance. What should you do?

A. Run gcloud iam roles describe roles/spanner.databaseUser. Add the users to the role.

B. Run gcloud iam roles describe roles/spanner.databaseUser. Add the users to a new group. Add the group to the role.

C. Run gcloud iam roles describe roles/spanner.viewer --project my-project. Add the users to the role.

D. Run gcloud iam roles describe roles/spanner.viewer --project my-project. Add the users to a new group. Add the group to the role.

---

## Question 92 (PDF Q146)

Your managed instance group raised an alert stating that new instance creation has failed to create new instances. You need to maintain the number of running instances specified by the template to be able to process expected application traffic. What should you do?

A. Create an instance template that contains valid syntax which will be used by the instance group. Delete any persistent disks with the same name as instance names.

B. Create an instance template that contains valid syntax that will be used by the instance group. Verify that the instance name and persistent disk name values are not the same in the template.

C. Verify that the instance template being used by the instance group contains valid syntax. Delete any persistent disks with the same name as instance names. Set the disks.autoDelete property to true in the instance template.

D. Delete the current instance template and replace it with a new instance template. Verify that the instance name and persistent disk name values are not the same in the template. Set the disks.autoDelete property to true in the instance template.

---

## Question 93 (PDF Q148)

A colleague handed over a Google Cloud project for you to maintain. As part of a security checkup, you want to review who has been granted the Project Owner role. What should you do?

A. In the Google Cloud console, validate which SSH keys have been stored as project-wide keys.

B. Navigate to Identity-Aware Proxy and check the permissions for these resources.

C. Enable Audit logs on the 1AM & admin page for all resources, and validate the results.

D. Use the gcloud projects get-iam-policy command to view the current role assignments.

---

## Question 94 (PDF Q153)

You have an object in a Cloud Storage bucket that you want to share with an external company. The object contains sensitive data. You want access to the content to be removed after four hours. The external company does not have a Google account to which you can grant specific user-based access privileges. You want to use the most secure method that requires the fewest steps. What should you do?

A. Create a signed URL with a four-hour expiration and share the URL with the company.

B. Set object access to 'public' and use object lifecycle management to remove the object after four hours.

C. Configure the storage bucket as a static website and furnish the object's URL to the company. Delete the object from the storage bucket after four hours.

D. Create a new Cloud Storage bucket specifically for the external company to access. Copy the object to that bucket. Delete the bucket after four hours have passed.

---

## Question 95 (PDF Q154)

Your company set up a complex organizational structure on Google Could Platform. The structure includes hundreds of folders and projects. Only a few team members should be able to view the hierarchical structure. You need to assign minimum permissions to these team members and you want to follow Google-recommended practices. What should you do?

A. Add the users to roles/browser role.

B. Add the users to roles/iam.roleViewer role.

C. Add the users to a group, and add this group to roles/browser role.

D. Add the users to a group, and add this group to roles/iam.roleViewer role.

---

## Question 96 (PDF Q155)

You have a Bigtable instance that consists of three nodes that store personally identifiable information (Pll) data. You need to log all read or write operations, including any metadata or configuration reads of this database table, in your company's Security Information and Event Management (SIEM) system. What should you do?

A. - Navigate to Cloud Mentioning in the Google Cloud console, and create a custom monitoring job for the Bigtable instance to track all changes. - Create an alert by using webhook endpoints. with the SIEM endpoint as a receiver

B. Navigate to the Audit Logs page in the Google Cloud console, and enable Data Read. Data Write and Admin Read logs for the Bigtable instance - Create a Pub/Sub topic as a Cloud Logging sink destination, and add your SIEM as a subscriber to the topic.

C. - Install the Ops Agent on the Bigtable instance during configuration. K - Create a service account with read permissions for the Bigtable instance. - Create a custom Dataflow job with this service account to export logs to the company's SIEM system.

D. - Navigate to the Audit Logs page in the Google Cloud console, and enable Admin Write logs for the Biglable instance. - Create a Cloud Functions instance to export logs from Cloud Logging to your SIEM.

---

## Question 97 (PDF Q156)

You are running multiple VPC-native Google Kubernetes Engine clusters in the same subnet. The IPs available for the nodes are exhausted, and you want to ensure that the clusters can grow in nodes when needed. What should you do?

A. Create a new subnet in the same region as the subnet being used.

B. Add an alias IP range to the subnet used by the GKE clusters.

C. Create a new VPC, and set up VPC peering with the existing VPC.

D. Expand the CIDR range of the relevant subnet for the cluster.

---

## Question 98 (PDF Q158)

Your company requires all developers to have the same permissions, regardless of the Google Cloud project they are working on. Your company's security policy also restricts developer permissions to Compute Engine. Cloud Functions, and Cloud SQL. You want to implement the security policy with minimal effort. What should you do?

A. - Create a custom role with Compute Engine, Cloud Functions, and Cloud SQL permissions in one project within the Google Cloud organization. - Copy the role across all projects created within the organization with the gcloud iam roles copy command. - Assign the role to developers in those projects.

B. - Add all developers to a Google group in Google Groups for Workspace. - Assign the predefined role of Compute Admin to the Google group at the Google Cloud organization level.

C. - Add all developers to a Google group in Cloud Identity. - Assign predefined roles for Compute Engine, Cloud Functions, and Cloud SQL permissions to the Google group for each project in the Google Cloud organization.

D. - Add all developers to a Google group in Cloud Identity. - Create a custom role with Compute Engine, Cloud Functions, and Cloud SQL permissions at the Google Cloud organization level. - Assign the custom role to the Google group.

---

## Question 99 (PDF Q159)

You recently discovered that your developers are using many service account keys during their development process. While you work on a long term improvement, you need to quickly implement a process to enforce short-lived service account credentials in your company. You have the following requirements: - All service accounts that require a key should be created in a centralized project called pj- sa. - Service account keys should only be valid for one day. You need a Google-recommended solution that minimizes cost. What should you do?

A. Implement a Cloud Run job to rotate all service account keys periodically in pj-sa. Enforce an org policy to deny service account key creation with an exception to pj-sa.

B. Implement a Kubernetes Cronjob to rotate all service account keys periodically. Disable attachment of service accounts to resources in all projects with an exception to pj-sa.

C. Enforce an org policy constraint allowing the lifetime of service account keys to be 24 hours. Enforce an org policy constraint denying service account key creation with an exception on pj-sa.

D. Enforce a DENY org policy constraint over the lifetime of service account keys for 24 hours. Disable attachment of service accounts to resources in all projects with an exception to pj-sa.

---

## Question 100 (PDF Q161)

You are analyzing Google Cloud Platform service costs from three separate projects. You want to use this information to create service cost estimates by service type, daily and monthly, for the next six months using standard query syntax. What should you do?

A. Export your bill to a Cloud Storage bucket, and then import into Cloud Bigtable for analysis.

B. Export your bill to a Cloud Storage bucket, and then import into Google Sheets for analysis.

C. Export your transactions to a local file, and perform analysis with a desktop tool.

D. Export your bill to a BigQuery dataset, and then write time window-based SQL queries for analysis.

---

## Question 101 (PDF Q162)

You are working in a team that has developed a new application that needs to be deployed on Kubernetes. The production application is business critical and should be optimized for reliability. You need to provision a Kubernetes cluster and want to follow Google- recommended practices. What should you do?

A. Create a GKE Autopilot cluster. Enroll the cluster in the rapid release channel.

B. Create a GKE Autopilot cluster. Enroll the cluster in the stable release channel.

C. Create a zonal GKE standard cluster. Enroll the cluster in the stable release channel.

D. Create a regional GKE standard cluster. Enroll the cluster in the rapid release channel.

---

## Question 102 (PDF Q163)

You are responsible for a web application on Compute Engine. You want your support team to be notified automatically if users experience high latency for at least 5 minutes. You need a Google-recommended solution with no development cost. What should you do?

A. Create an alert policy to send a notification when the HTTP response latency exceeds the specified threshold.

B. Implement an App Engine service which invokes the Cloud Monitoring API and sends a notification in case of anomalies.

C. Use the Cloud Monitoring dashboard to observe latency and take the necessary actions when the response latency exceeds the specified threshold.

D. Export Cloud Monitoring metrics to BigQuery and use a Looker Studio dashboard to monitor your web applications latency.

---

## Question 103 (PDF Q164)

You have production and test workloads that you want to deploy on Compute Engine. Production VMs need to be in a different subnet than the test VMs. All the VMs must be able to reach each other over internal IP without creating additional routes. You need to set up VPC and the 2 subnets. Which configuration meets these requirements?

A. Create a single custom VPC with 2 subnets. Create each subnet in a different region and with a different CIDR range.

B. Create a single custom VPC with 2 subnets. Create each subnet in the same region and with the same CIDR range.

C. Create 2 custom VPCs, each with a single subnet. Create each subnet is a different region and with a different CIDR range.

D. Create 2 custom VPCs, each with a single subnet. Create each subnet in the same region and with the same CIDR range.

---

## Question 104 (PDF Q165)

Your Dataproc cluster runs in a single Virtual Private Cloud (VPC) network in a single subnet with range 172.16.20.128/25. There are no private IP addresses available in the VPC network. You want to add new VMs to communicate with your cluster using the minimum number of steps. What should you do?

A. Modify the existing subnet range to 172.16.20.0/24.

B. Create a new Secondary IP Range in the VPC and configure the VMs to use that range.

C. Create a new VPC network for the VMs. Enable VPC Peering between the VMs' VPC network and the Dataproc cluster VPC network.

D. Create a new VPC network for the VMs with a subnet of 172.32.0.0/16. Enable VPC network Peering between the Dataproc VPC network and the VMs VPC network. Configure a custom Route exchange.

---

## Question 105 (PDF Q166)

You have been asked to create robust Virtual Private Network (VPN) connectivity between a new Virtual Private Cloud (VPC) and a remote site. Key requirements include dynamic routing, a shared address space of 10.19.0.1/22, and no overprovisioning of tunnels during a failover event. You want to follow Google-recommended practices to set up a high availability Cloud VPN. What should you do?

A. Use a custom mode VPC network, configure static routes, and use active/passive routing

B. Use an automatic mode VPC network, configure static routes, and use active/active routing

C. Use a custom mode VPC network use Cloud Router border gateway protocol (86P) routes, and use active/passive routing

D. Use an automatic mode VPC network, use Cloud Router border gateway protocol (BGP) routes and configure policy-based routing

---

## Question 106 (PDF Q167)

You have a web application deployed as a managed instance group. You have a new version of the application to gradually deploy. Your web application is currently receiving live web traffic. You want to ensure that the available capacity does not decrease during the deployment. What should you do?

A. Perform a rolling-action start-update with maxSurge set to 0 and maxUnavailable set to 1.

B. Perform a rolling-action start-update with maxSurge set to 1 and maxUnavailable set to 0.

C. Create a new managed instance group with an updated instance template. Add the group to the backend service for the load balancer. When all instances in the new managed instance group are healthy, delete the old managed instance group.

D. Create a new instance template with the new application version. Update the existing managed instance group with the new instance template. Delete the instances in the managed instance group to allow the managed instance group to recreate the instance using the new instance template.

---

## Question 107 (PDF Q169)

You have deployed multiple Linux instances on Compute Engine. You plan on adding more instances in the coming weeks. You want to be able to access all of these instances through your SSH client over me Internet without having to configure specific access on the existing and new instances. You do not want the Compute Engine instances to have a public IP. What should you do?

A. Configure Cloud Identity-Aware Proxy (or HTTPS resources

B. Configure Cloud Identity-Aware Proxy for SSH and TCP resources.

C. Create an SSH keypair and store the public key as a project-wide SSH Key

D. Create an SSH keypair and store the private key as a project-wide SSH Key

---

## Question 108 (PDF Q170)

You are the project owner of a GCP project and want to delegate control to colleagues to manage buckets and files in Cloud Storage. You want to follow Google-recommended practices. Which IAM roles should you grant your colleagues?

A. Project Editor

B. Storage Admin

C. Storage Object Admin

D. Storage Object Creator

---

## Question 109 (PDF Q172)

A company wants to build an application that stores images in a Cloud Storage bucket and wants to generate thumbnails as well as resize the images. They want to use a google managed service that can scale up and scale down to zero automatically with minimal effort. You have been asked to recommend a service. Which GCP service would you suggest?

A. Google Compute Engine

B. Google App Engine

C. Cloud Functions

D. Google Kubernetes Engine

---

## Question 110 (PDF Q175)

You installed the Google Cloud CLI on your workstation and set the proxy configuration. However, you are worried that your proxy credentials will be recorded in the gcloud CLI logs. You want to prevent your proxy credentials from being logged What should you do?

A. Configure username and password by using gcloud configure set proxy/username and gcloud configure set proxy/ proxy/password commands.

B. Encode username and password in sha256 encoding, and save it to a text file. Use filename as a value in the gcloud configure set core/custom_ca_certs_file command.

C. Provide values for CLOUDSDK_USERNAME and CLOUDSDK_PASSWORD in the gcloud CLI tool configure file.

D. Set the CLOUDSDK_PROXY_USERNAME and CLOUDSDK_PROXY PASSWORD properties by using environment variables in your command line tool.

---

## Question 111 (PDF Q177)

You are deploying an application to App Engine. You want the number of instances to scale based on request rate. You need at least 3 unoccupied instances at all times. Which scaling type should you use?

A. Manual Scaling with 3 instances.

B. Basic Scaling with min_instances set to 3.

C. Basic Scaling with max_instances set to 3.

D. Automatic Scaling with min_idle_instances set to 3.

---

## Question 112 (PDF Q178)

You are running multiple microservices in a Kubernetes Engine cluster. One microservice is rendering images. The microservice responsible for the image rendering requires a large amount of CPU time compared to the memory it requires. The other microservices are workloads that are optimized for n1-standard machine types. You need to optimize your cluster so that all workloads are using resources as efficiently as possible. What should you do?

A. Assign the pods of the image rendering microservice a higher pod priority than the older microservices

B. Create a node pool with compute-optimized machine type nodes for the image rendering microservice Use the node pool with general-purpose machine type nodes for the other microservices

C. Use the node pool with general-purpose machine type nodes for lite mage rendering microservice Create a nodepool with compute-optimized machine type nodes for the other microservices

D. Configure the required amount of CPU and memory in the resource requests specification of the image rendering microservice deployment Keep the resource requests for the other microservices at the default

---

## Question 113 (PDF Q179)

You are hosting an application from Compute Engine virtual machines (VMs) in us-central1-a. You want to adjust your design to support the failure of a single Compute Engine zone, eliminate downtime, and minimize cost. What should you do?

A. - Create Compute Engine resources in us-central1-b. -Balance the load across both us-central1-a and us-central1-b.

B. - Create a Managed Instance Group and specify us-central1-a as the zone. -Configure the Health Check with a short Health Interval.

C. - Create an HTTP(S) Load Balancer. -Create one or more global forwarding rules to direct traffic to your VMs.

D. - Perform regular backups of your application. -Create a Cloud Monitoring Alert and be notified if your application becomes unavailable. -Restore from backups when notified.

---

## Question 114 (PDF Q184)

Your continuous integration and delivery (CI/CD) server can't execute Google Cloud actions in a specific project because of permission issues. You need to validate whether the used service account has the appropriate roles in the specific project. What should you do?

A. Open the Google Cloud console, and run a query to determine which resources this service account can access.

B. Open the Google Cloud console, and run a query of the audit logs to find permission denied errors for this service account.

C. Open the Google Cloud console, and check the organization policies.

D. Open the Google Cloud console, and check the Identity and Access Management (IAM) roles assigned to the service account at the project or inherited from the folder or organization levels.

---

## Question 115 (PDF Q185)

You have an application on a general-purpose Compute Engine instance that is experiencing excessive disk read throttling on its Zonal SSD Persistent Disk. The application primarily reads large files from disk. The disk size is currently 350 GB. You want to provide the maximum amount of throughput while minimizing costs. What should you do?

A. Increase the size of the disk to 1 TB.

B. Increase the allocated CPU to the instance.

C. Migrate to use a Local SSD on the instance.

D. Migrate to use a Regional SSD on the instance.

---

## Question 116 (PDF Q186)

You created a Kubernetes deployment by running kubectl run nginx image=nginx replicas=1. After a few days, you decided you no longer want this deployment. You identified the pod and deleted it by running kubectl delete pod. You noticed the pod got recreated. - $ kubectl get pods - NAME READY STATUS RESTARTS AGE - nginx-84748895c4-nqqmt 1/1 Running 0 9m41s - $ kubectl delete pod nginx-84748895c4-nqqmt - pod nginx-84748895c4-nqqmt deleted - $ kubectl get pods - NAME READY STATUS RESTARTS AGE - nginx-84748895c4-k6bzl 1/1 Running 0 25s What should you do to delete the deployment and avoid pod getting recreated?

A. kubectl delete deployment nginx

B. kubectl delete -deployment=nginx

C. kubectl delete pod nginx-84748895c4-k6bzl -no-restart 2

D. kubectl delete inginx

---

## Question 117 (PDF Q187)

You have a virtual machine that is currently configured with 2 vCPUs and 4 GB of memory. It is running out of memory. You want to upgrade the virtual machine to have 8 GB of memory. What should you do?

A. Rely on live migration to move the workload to a machine with more memory.

B. Use gcloud to add metadata to the VM. Set the key to required-memory-size and the value to 8 GB.

C. Stop the VM, change the machine type to n1-standard-8, and start the VM.

D. Stop the VM, increase the memory to 8 GB, and start the VM.

---

## Question 118 (PDF Q188)

You have deployed an application on a Compute Engine instance. An external consultant needs to access the Linux-based instance. The consultant is connected to your corporate network through a VPN connection, but the consultant has no Google account. What should you do?

A. Instruct the external consultant to use the gcloud compute ssh command line tool by using Identity-Aware Proxy to access the instance.

B. Instruct the external consultant to use the gcloud compute ssh command line tool by using the public IP address of the instance to access it.

C. Instruct the external consultant to generate an SSH key pair, and request the public key from the consultant. Add the public key to the instance yourself, and have the consultant access the instance through SSH with their private key.

D. Instruct the external consultant to generate an SSH key pair, and request the private key from the consultant. Add the private key to the instance yourself, and have the consultant access the instance through SSH with their public key.

---

## Question 119 (PDF Q189)

You need to add a group of new users to Cloud Identity. Some of the users already have existing Google accounts. You want to follow one of Google's recommended practices and avoid conflicting accounts. What should you do?

A. Invite the user to transfer their existing account

B. Invite the user to use an email alias to resolve the conflict

C. Tell the user that they must delete their existing account

D. Tell the user to remove all personal email from the existing account

---

## Question 120 (PDF Q193)

You are building a multi-player gaming application that will store game information in a database. As the popularity of the application increases, you are concerned about delivering consistent performance. You need to ensure an optimal gaming performance for global users, without increasing the management complexity. What should you do?

A. Use Cloud SQL database with cross-region replication to store game statistics in the EU, US, and APAC regions.

B. Use Cloud Spanner to store user data mapped to the game statistics.

C. Use BigQuery to store game statistics with a Redis on Memorystore instance in the front to provide global consistency.

D. Store game statistics in a Bigtable database partitioned by username.

---

## Question 121 (PDF Q194)

You have a workload running on Compute Engine that is critical to your business. You want to ensure that the data on the boot disk of this workload is backed up regularly. You need to be able to restore a backup as quickly as possible in case of disaster. You also want older backups to be cleaned automatically to save on cost. You want to follow Google- recommended practices. What should you do?

A. Create a Cloud Function to create an instance template.

B. Create a snapshot schedule for the disk using the desired interval.

C. Create a cron job to create a new disk from the disk using gcloud.

D. Create a Cloud Task to create an image and export it to Cloud Storage.

---

## Question 122 (PDF Q195)

Your finance team wants to view the billing report for your projects. You want to make sure that the finance team does not get additional permissions to the project. What should you do?

A. Add the group for the finance team to roles/billing user role.

B. Add the group for the finance team to roles/billing admin role.

C. Add the group for the finance team to roles/billing viewer role.

D. Add the group for the finance team to roles/billing project/Manager role.

---

## Question 123 (PDF Q196)

You are setting up a Windows VM on Compute Engine and want to make sure you can log in to the VM via RDP. What should you do?

A. After the VM has been created, use your Google Account credentials to log in into the VM.

B. After the VM has been created, use gcloud compute reset-windows-password to retrieve the login credentials for the VM.

C. When creating the VM, add metadata to the instance using 'windows-password' as the key and a password as the value.

D. After the VM has been created, download the JSON private key for the default Compute Engine service account. Use the credentials in the JSON file to log in to the VM.

---

## Question 124 (PDF Q197)

You need to run an important query in BigQuery but expect it to return a lot of records. You want to find out how much it will cost to run the query. You are using on-demand pricing. What should you do?

A. Arrange to switch to Flat-Rate pricing for this query, then move back to on-demand.

B. Use the command line to run a dry run query to estimate the number of bytes read. Then convert that bytes estimate to dollars using the Pricing Calculator.

C. Use the command line to run a dry run query to estimate the number of bytes returned. Then convert that bytes estimate to dollars using the Pricing Calculator.

D. Run a select count (*) to get an idea of how many records your query will look through. Then convert that number of rows to dollars using the Pricing Calculator.

---

## Question 125 (PDF Q199)

You need to deploy an application in Google Cloud using savorless technology. You want to test a new version of the application with a small percentage of production traffic. What should you do?

A. Deploy the application lo Cloud. Run. Use gradual rollouts for traffic spelling.

B. Deploy the application lo Google Kubemetes Engine. Use Anthos Service Mesh for traffic splitting.

C. Deploy the application to Cloud functions. Saucily the version number in the functions name.

D. Deploy the application to App Engine. For each new version, create a new service.

---

## Question 126 (PDF Q200)

Your company is moving from an on-premises environment to Google Cloud Platform (GCP). You have multiple development teams that use Cassandra environments as backend databases. They all need a development environment that is isolated from other Cassandra instances. You want to move to GCP quickly and with minimal support effort. What should you do?

A. * 1. Build an instruction guide to install Cassandra on GCP. * 2. Make the instruction guide accessible to your developers.

B. * 1. Advise your developers to go to Cloud Marketplace. * 2. Ask the developers to launch a Cassandra image for their development work.

C. * 1. Build a Cassandra Compute Engine instance and take a snapshot of it. * 2. Use the snapshot to create instances for your developers.

D. * 1. Build a Cassandra Compute Engine instance and take a snapshot of it. * 2.Upload the snapshot to Cloud Storage and make it accessible to your developers. * 3.Build instructions to create a Compute Engine instance from the snapshot so that developers can do it themselves.

---

## Question 127 (PDF Q206)

The core business of your company is to rent out construction equipment at a large scale. All the equipment that is being rented out has been equipped with multiple sensors that send event information every few seconds. These signals can vary from engine status, distance traveled, fuel level, and more. Customers are billed based on the consumption monitored by these sensors. You expect high throughput - up to thousands of events per hour per device - and need to retrieve consistent data based on the time of the event. Storing and retrieving individual signals should be atomic. What should you do?

A. Create a file in Cloud Storage per device and append new data to that file.

B. Create a file in Cloud Filestore per device and append new data to that file.

C. Ingest the data into Datastore. Store data in an entity group based on the device.

D. Ingest the data into Cloud Bigtable. Create a row key based on the event timestamp.

---

## Question 128 (PDF Q207)

You are managing a Data Warehouse on BigQuery. An external auditor will review your company's processes, and multiple external consultants will need view access to the data. You need to provide them with view access while following Google-recommended practices. What should you do?

A. Grant each individual external consultant the role of BigQuery Editor

B. Grant each individual external consultant the role of BigQuery Viewer

C. Create a Google Group that contains the consultants and grant the group the role of BigQuery Editor

D. Create a Google Group that contains the consultants, and grant the group the role of BigQuery Viewer

---

## Question 129 (PDF Q208)

You have an application running in Google Kubernetes Engine (GKE) with cluster autoscaling enabled. The application exposes a TCP endpoint. There are several replicas of this application. You have a Compute Engine instance in the same region, but in another Virtual Private Cloud (VPC), called gce-network, that has no overlapping IP ranges with the first VPC. This instance needs to connect to the application on GKE. You want to minimize effort. What should you do?

A. 1. In GKE, create a Service of type LoadBalancer that uses the application's Pods as backend.2. Set the service's externalTrafficPolicy to Cluster.3. Configure the Compute Engine instance to use the address of the load balancer that has been created.

B. 1. In GKE, create a Service of type NodePort that uses the application's Pods as backend.2. Create a Compute Engine instance called proxy with 2 network interfaces, one in each VPC.3. Use iptables on this instance to forward traffic from gce-network to the GKE nodes.4. Configure the Compute Engine instance to use the address of proxy in gce- network as endpoint.

C. 1. In GKE, create a Service of type LoadBalancer that uses the application's Pods as backend.2. Add an annotation to this service: cloud.google.com/load-balancer-type: Internal3. Peer the two VPCs together.4. Configure the Compute Engine instance to use the address of the load balancer that has been created.

D. 1. In GKE, create a Service of type LoadBalancer that uses the application's Pods as backend.2. Add a Cloud Armor Security Policy to the load balancer that whitelists the internal IPs of the MIG's instances.3. Configure the Compute Engine instance to use the address of the load balancer that has been created.

---

## Question 130 (PDF Q210)

Your learn wants to deploy a specific content management system (CMS) solution lo Google Cloud. You need a quick and easy way to deploy and install the solution. What should you do?

A. Search for the CMS solution in Google Cloud Marketplace. Use gcloud CLI to deploy the solution.

B. Search for the CMS solution in Google Cloud Marketplace. Deploy the solution directly from Cloud Marketplace.

C. Search for the CMS solution in Google Cloud Marketplace. Use Terraform and the Cloud Marketplace ID to deploy the solution with the appropriate parameters.

D. Use the installation guide of the CMS provider. Perform the installation through your configuration management system.

---

## Question 131 (PDF Q211)

You are migrating a production-critical on-premises application that requires 96 vCPUs to perform its task. You want to make sure the application runs in a similar environment on GCP. What should you do?

A. When creating the VM, use machine type n1-standard-96.

B. When creating the VM, use Intel Skylake as the CPU platform.

C. Create the VM using Compute Engine default settings. Use gcloud to modify the running instance to have 96 vCPUs.

D. Start the VM using Compute Engine default settings, and adjust as you go based on Rightsizing Recommendations.

---

## Question 132 (PDF Q213)

Your company uses a large number of Google Cloud services centralized in a single project. All teams have specific projects for testing and development. The DevOps team needs access to all of the production services in order to perform their job. You want to prevent Google Cloud product changes from broadening their permissions in the future. You want to follow Google-recommended practices. What should you do?

A. Grant all members of the DevOps team the role of Project Editor on the organization level.

B. Grant all members of the DevOps team the role of Project Editor on the production project.

C. Create a custom role that combines the required permissions. Grant the DevOps team the custom role on the production project.

D. Create a custom role that combines the required permissions. Grant the DevOps team the custom role on the organization level.

---

## Question 133 (PDF Q214)

Every employee of your company has a Google account. Your operational team needs to manage a large number of instances on Compute Engine. Each member of this team needs only administrative access to the servers. Your security team wants to ensure that the deployment of credentials is operationally efficient and must be able to determine who accessed a given instance. What should you do?

A. Generate a new SSH key pair. Give the private key to each member of your team. Configure the public key in the metadata of each instance.

B. Ask each member of the team to generate a new SSH key pair and to send you their public key. Use a configuration management tool to deploy those keys on each instance.

C. Ask each member of the team to generate a new SSH key pair and to add the public key to their Google account. Grant the "compute.osAdminLogin" role to the Google group corresponding to this team.

D. Generate a new SSH key pair. Give the private key to each member of your team. Configure the public key as a project-wide public SSH key in your Cloud Platform project and allow project-wide public SSH keys on each instance.

---

## Question 134 (PDF Q216)

You need to create a custom VPC with a single subnet. The subnet's range must be as large as possible. Which range should you use?

A. .00.0.0/0

B. 10.0.0.0/8

C. 172.16.0.0/12

D. 192.168.0.0/16

---

## Question 135 (PDF Q217)

Your company uses Cloud Storage to store application backup files for disaster recovery purposes. You want to follow Google's recommended practices. Which storage option should you use?

A. Multi-Regional Storage

B. Regional Storage

C. Nearline Storage

D. Coldline Storage

---

## Question 136 (PDF Q218)

You manage an App Engine Service that aggregates and visualizes data from BigQuery. The application is deployed with the default App Engine Service account. The data that needs to be visualized resides in a different project managed by another team. You do not have access to this project, but you want your application to be able to read data from the BigQuery dataset. What should you do?

A. Ask the other team to grant your default App Engine Service account the role of BigQuery Job User.

B. Ask the other team to grant your default App Engine Service account the role of BigQuery Data Viewer.

C. In Cloud IAM of your project, ensure that the default App Engine service account has the role of BigQuery Data Viewer.

D. In Cloud IAM of your project, grant a newly created service account from the other team the role of BigQuery Job User in your project.

---

## Question 137 (PDF Q221)

You are building a new version of an application hosted in an App Engine environment. You want to test the new version with 1% of users before you completely switch your application over to the new version. What should you do?

A. Deploy a new version of your application in Google Kubernetes Engine instead of App Engine and then use GCP Console to split traffic.

B. Deploy a new version of your application in a Compute Engine instance instead of App Engine and then use GCP Console to split traffic.

C. Deploy a new version as a separate app in App Engine. Then configure App Engine using GCP Console to split traffic between the two apps.

D. Deploy a new version of your application in App Engine. Then go to App Engine settings in GCP Console and split traffic between the current version and newly deployed versions accordingly.

---

## Question 138 (PDF Q223)

Your coworker has helped you set up several configurations for gcloud. You've noticed that you're running commands against the wrong project. Being new to the company, you haven't yet memorized any of the projects. With the fewest steps possible, what's the fastest way to switch to the correct configuration?

A. Run gcloud configurations list followed by gcloud configurations activate .

B. Run gcloud config list followed by gcloud config activate.

C. Run gcloud config configurations list followed by gcloud config configurations activate.

D. Re-authenticate with the gcloud auth login command and select the correct configurations on login.

---

## Question 139 (PDF Q225)

You need to reduce GCP service costs for a division of your company using the fewest possible steps. You need to turn off all configured services in an existing GCP project. What should you do?

A. * 1. Verify that you are assigned the Project Owners IAM role for this project. * 2. Locate the project in the GCP console, click Shut down and then enter the project ID.

B. * 1. Verify that you are assigned the Project Owners IAM role for this project. * 2. Switch to the project in the GCP console, locate the resources and delete them.

C. * 1. Verify that you are assigned the Organizational Administrator IAM role for this project. * 2. Locate the project in the GCP console, enter the project ID and then click Shut down.

D. * 1. Verify that you are assigned the Organizational Administrators IAM role for this project. * 2. Switch to the project in the GCP console, locate the resources and delete them.

---

## Question 140 (PDF Q226)

An application generates daily reports in a Compute Engine virtual machine (VM). The VM is in the project corp-iot-insights. Your team operates only in the project corp-aggregate- reports and needs a copy of the daily exports in the bucket corp-aggregate-reports-storage. You want to configure access so that the daily reports from the VM are available in the bucket corp-aggregate-reports-storage and use as few steps as possible while following Google-recommended practices. What should you do?

A. Move both projects under the same folder.

B. Grant the VM Service Account the role Storage Object Creator on corp-aggregate- reports-storage.

C. Create a Shared VPC network between both projects. Grant the VM Service Account the role Storage Object Creator on corp-iot-insights.

D. Make corp-aggregate-reports-storage public and create a folder with a pseudo- randomized suffix name. Share the folder with the IoT team.

---

## Question 141 (PDF Q227)

You want to configure an SSH connection to a single Compute Engine instance for users in the dev1 group. This instance is the only resource in this particular Google Cloud Platform project that the dev1 users should be able to connect to. What should you do?

A. Set metadata to enable-oslogin=true for the instance. Grant the dev1 group the compute.osLogin role. Direct them to use the Cloud Shell to ssh to that instance.

B. Set metadata to enable-oslogin=true for the instance. Set the service account to no service account for that instance. Direct them to use the Cloud Shell to ssh to that instance.

C. Enable block project wide keys for the instance. Generate an SSH key for each user in the dev1 group. Distribute the keys to dev1 users and direct them to use their third-party tools to connect.

D. Enable block project wide keys for the instance. Generate an SSH key and associate the key with that instance. Distribute the key to dev1 users and direct them to use their third-party tools to connect.

---

## Question 142 (PDF Q228)

You've deployed a microservice called myapp1 to a Google Kubernetes Engine cluster using the YAML file specified below: You need to refactor this configuration so that the database password is not stored in plain text. You want to follow Google-recommended practices. What should you do?

A. Store the database password inside the Docker image of the container, not in the YAML file.

B. Store the database password inside a Secret object. Modify the YAML file to populate the DB_PASSWORD environment variable from the Secret.

C. Store the database password inside a ConfigMap object. Modify the YAML file to populate the DB_PASSWORD environment variable from the ConfigMap.

D. Store the database password in a file inside a Kubernetes persistent volume, and use a persistent volume claim to mount the volume to the container.

---

## Question 143 (PDF Q231)

Your company completed the acquisition of a startup and is now merging the IT systems of both companies. The startup had a production Google Cloud project in their organization. You need to move this project into your organization and ensure that the project is billed lo your organization. You want to accomplish this task with minimal effort. What should you do?

A. Use the projects. move method to move the project to your organization. Update the billing account of the project to that of your organization.

B. Ensure that you have an Organization Administrator Identity and Access Management (IAM) role assigned to you in both organizations. Navigate to the Resource Manager in the startup's Google Cloud organization, and drag the project to your company's organization.

C. Create a Private Catalog tor the Google Cloud Marketplace, and upload the resources of the startup's production project to the Catalog. Share the Catalog with your organization, and deploy the resources in your company's project.

D. Create an infrastructure-as-code template tor all resources in the project by using Terraform. and deploy that template to a new project in your organization. Delete the protect from the startup's Google Cloud organization.

---

## Question 144 (PDF Q232)

You have a website hosted on App Engine standard environment. You want 1% of your users to see a new test version of the website. You want to minimize complexity. What should you do?

A. Deploy the new version in the same application and use the --migrate option.

B. Deploy the new version in the same application and use the --splits option to give a weight of 99 to the current version and a weight of 1 to the new version.

C. Create a new App Engine application in the same project. Deploy the new version in that application. Use the App Engine library to proxy 1% of the requests to the new version.

D. Create a new App Engine application in the same project. Deploy the new version in that application. Configure your network load balancer to send 1% of the traffic to that new application.

---

## Question 145 (PDF Q235)

Your customer wants you to create a secure website with autoscaling based on the compute instance CPU load. You want to enhance performance by storing static content in Cloud Storage. Which resources are needed to distribute the user traffic?

A. An internal HTTP(S) load balancer together with Identity-Aware Proxy to allow only HTTPS traffic.

B. An external HTTP(S) load balancer to distribute the load and a URL map to target the requests for the static content to the Cloud Storage backend. Install the HTTPS certificates on the instance.

C. An external HTTP(S) load balancer with a managed SSL certificate to distribute the load and a URL map to target the requests for the static content to the Cloud Storage backend.

D. An external network load balancer pointing to the backend instances to distribute the load evenly. The web servers will forward the request to the Cloud Storage as needed.

---

## Question 146 (PDF Q236)

You host a static website on Cloud Storage. Recently, you began to include links to PDF files on this site. Currently, when users click on the links to these PDF files, their browsers prompt them to save the file onto their local system. Instead, you want the clicked PDF files to be displayed within the browser window directly, without prompting the user to save the file locally. What should you do?

A. Enable Cloud CDN on the website frontend.

B. Enable 'Share publicly' on the PDF file objects.

C. Set Content-Type metadata to application/pdf on the PDF file objects.

D. Add a label to the storage bucket with a key of Content-Type and value of application/pdf.

---

## Question 147 (PDF Q238)

You want to permanently delete a Pub/Sub topic managed by Config Connector in your Google Cloud project. What should you do?

A. Use kubectl to delete the topic resource.

B. Use gcloud CLI to delete the topic.

C. Use kubectl to create the label deleted-by-cnrm and to change its value to true for the topic resource.

D. Use gcloud CLI to update the topic label managed-by-cnrm to false.

---

## Question 148 (PDF Q239)

The sales team has a project named Sales Data Digest that has the ID acme-data-digest You need to set up similar Google Cloud resources for the marketing team but their resources must be organized independently of the sales team. What should you do?

A. Grant the Project Editor role to the Marketing learn for acme data digest

B. Create a Project Lien on acme-data digest and then grant the Project Editor role to the Marketing team

C. Create another protect with the ID acme-marketing-data-digest for the Marketing team and deploy the resources there

D. Create a new protect named Meeting Data Digest and use the ID acme-data-digest Grant the Project Editor role to the Marketing team.

---

## Question 149 (PDF Q240)

You need to manage multiple Google Cloud Platform (GCP) projects in the fewest steps possible. You want to configure the Google Cloud SDK command line interface (CLI) so that you can easily manage multiple GCP projects. What should you?

A. * 1. Create a configuration for each project you need to manage. * 2. Activate the appropriate configuration when you work with each of your assigned GCP projects.

B. * 1. Create a configuration for each project you need to manage. * 2. Use gcloud init to update the configuration values when you need to work with a non- default project

C. * 1. Use the default configuration for one project you need to manage. * 2. Activate the appropriate configuration when you work with each of your assigned GCP projects.

D. * 1. Use the default configuration for one project you need to manage. * 2. Use gcloud init to update the configuration values when you need to work with a non- default project.

---

## Question 150 (PDF Q244)

You have experimented with Google Cloud using your own credit card and expensed the costs to your company. Your company wants to streamline the billing process and charge the costs of your projects to their monthly invoice. What should you do?

A. Grant the financial team the IAM role of Billing Account User on the billing account linked to your credit card.

B. Set up BigQuery billing export and grant your financial department IAM access to query the data.

C. Create a ticket with Google Billing Support to ask them to send the invoice to your company.

D. Change the billing account of your projects to the billing account of your company.

---

## Question 151 (PDF Q245)

Your organization needs to grant users access to query datasets in BigQuery but prevent them from accidentally deleting the datasets. You want a solution that follows Google- recommended practices. What should you do?

A. Add users to roles/bigquery user role only, instead of roles/bigquery dataOwner.

B. Add users to roles/bigquery dataEditor role only, instead of roles/bigquery dataOwner.

C. Create a custom role by removing delete permissions, and add users to that role only.

D. Create a custom role by removing delete permissions. Add users to the group, and then add the group to the custom role.

---

## Question 152 (PDF Q250)

You have one project called proj-sa where you manage all your service accounts. You want to be able to use a service account from this project to take snapshots of VMs running in another project called proj-vm. What should you do?

A. Download the private key from the service account, and add it to each VMs custom metadata.

B. Download the private key from the service account, and add the private key to each VM's SSH keys.

C. Grant the service account the IAM Role of Compute Storage Admin in the project called proj-vm.

D. When creating the VMs, set the service account's API scope for Compute Engine to read/write.

---

## Question 153 (PDF Q251)

Your company runs its Linux workloads on Compute Engine instances. Your company will be working with a new operations partner that does not use Google Accounts. You need to grant access to the instances to your operations partner so they can maintain the installed tooling. What should you do?

A. Enable Cloud IAP for the Compute Engine instances, and add the operations partner as a Cloud IAP Tunnel User.

B. Tag all the instances with the same network tag. Create a firewall rule in the VPC to grant TCP access on port 22 for traffic from the operations partner to instances with the network tag.

C. Set up Cloud VPN between your Google Cloud VPC and the internal network of the operations partner.

D. Ask the operations partner to generate SSH key pairs, and add the public keys to the VM instances.

---

## Question 154 (PDF Q252)

Your company has a single sign-on (SSO) identity provider that supports Security Assertion Markup Language (SAML) integration with service providers. Your company has users in Cloud Identity. You would like users to authenticate using your company's SSO provider. What should you do?

A. In Cloud Identity, set up SSO with Google as an identity provider to access custom SAML apps.

B. In Cloud Identity, set up SSO with a third-party identity provider with Google as a service provider.

C. Obtain OAuth 2.0 credentials, configure the user consent screen, and set up OAuth 2.0 for Mobile & Desktop Apps.

D. Obtain OAuth 2.0 credentials, configure the user consent screen, and set up OAuth 2.0 for Web Server Applications.

---

## Question 155 (PDF Q253)

You have a Compute Engine instance hosting an application used between 9 AM and 6 PM on weekdays. You want to back up this instance daily for disaster recovery purposes. You want to keep the backups for 30 days. You want the Google-recommended solution with the least management overhead and the least number of services. What should you do?

A. * 1. Update your instances' metadata to add the following value: snapshot-schedule: 0 1 * ** * 2. Update your instances' metadata to add the following value: snapshot-retention: 30

B. * 1. In the Cloud Console, go to the Compute Engine Disks page and select your instance's disk. * 2. In the Snapshot Schedule section, select Create Schedule and configure the following parameters: -Schedule frequency: Daily -Start time: 1:00 AM - 2:00 AM -Autodelete snapshots after 30 days

C. * 1. Create a Cloud Function that creates a snapshot of your instance's disk. * 2.Create a Cloud Function that deletes snapshots that are older than 30 days. * 3.Use Cloud Scheduler to trigger both Cloud Functions daily at 1:00 AM.

D. * 1. Create a bash script in the instance that copies the content of the disk to Cloud Storage. * 2. Create a bash script in the instance that deletes data older than 30 days in the backup Cloud Storage bucket. * 3. Configure the instance's crontab to execute these scripts daily at 1:00 AM.

---

## Question 156 (PDF Q258)

Your company has a Google Cloud Platform project that uses BigQuery for data warehousing. Your data science team changes frequently and has few members. You need to allow members of this team to perform queries. You want to follow Google- recommended practices. What should you do?

A. 1. Create an IAM entry for each data scientist's user account.2. Assign the BigQuery jobUser role to the group.

B. 1. Create an IAM entry for each data scientist's user account.2. Assign the BigQuery dataViewer user role to the group.

C. 1. Create a dedicated Google group in Cloud Identity.2. Add each data scientist's user account to the group.3. Assign the BigQuery jobUser role to the group.

D. 1. Create a dedicated Google group in Cloud Identity.2. Add each data scientist's user account to the group.3. Assign the BigQuery dataViewer user role to the group.

---

## Question 157 (PDF Q261)

You have one GCP account running in your default region and zone and another account running in a non-default region and zone. You want to start a new Compute Engine instance in these two Google Cloud Platform accounts using the command line interface. What should you do?

A. Create two configurations using gcloud config configurations create [NAME]. Run gcloud config configurations activate [NAME] to switch between accounts when running the commands to start the Compute Engine instances.

B. Create two configurations using gcloud config configurations create [NAME]. Run gcloud configurations list to start the Compute Engine instances.

C. Activate two configurations using gcloud configurations activate [NAME]. Run gcloud config list to start the Compute Engine instances.

D. Activate two configurations using gcloud configurations activate [NAME]. Run gcloud configurations list to start the Compute Engine instances.

---

## Question 158 (PDF Q262)

Your application is running on Google Cloud in a managed instance group (MIG). You see errors in Cloud Logging for one VM that one of the processes is not responsive. You want to replace this VM in the MIG quickly. What should you do?

A. Select the MIG from the Compute Engine console and, in the menu, select Replace VMs.

B. Use the gcloud compute instance-groups managed recreate-instances command to recreate theVM.

C. Use the gcloud compute instances update command with a REFRESH action for the VM.

D. Update and apply the instance template of the MIG.

---

## Question 159 (PDF Q263)

You are creating an application that will run on Google Kubernetes Engine. You have identified MongoDB as the most suitable database system for your application and want to deploy a managed MongoDB environment that provides a support SLA. What should you do?

A. Create a Cloud Bigtable cluster and use the HBase API

B. Deploy MongoDB Alias from the Google Cloud Marketplace

C. Download a MongoDB installation package and run it on Compute Engine instances

D. Download a MongoDB installation package, and run it on a Managed Instance Group

---

## Question 160 (PDF Q265)

You have been asked to set up the billing configuration for a new Google Cloud customer. Your customer wants to group resources that share common IAM policies. What should you do?

A. Use labels to group resources that share common IAM policies

B. Use folders to group resources that share common IAM policies

C. Set up a proper billing account structure to group IAM policies

D. Set up a proper project naming structure to group IAM policies

---

## Question 161 (PDF Q266)

Your company's infrastructure is on-premises, but all machines are running at maximum capacity. You want to burst to Google Cloud. The workloads on Google Cloud must be able to directly communicate to the workloads on-premises using a private IP range. What should you do?

A. In Google Cloud, configure the VPC as a host for Shared VPC.

B. In Google Cloud, configure the VPC for VPC Network Peering.

C. Create bastion hosts both in your on-premises environment and on Google Cloud. Configure both as proxy servers using their public IP addresses.

D. Set up Cloud VPN between the infrastructure on-premises and Google Cloud.

---

## Answers

| Question | Answer |
|---|---|
| 1 | C |
| 2 | B |
| 3 | A |
| 4 | A |
| 5 | D |
| 6 | D |
| 7 | A |
| 8 | B |
| 9 | D |
| 10 | D |
| 11 | A |
| 12 | D |
| 13 | C |
| 14 | B |
| 15 | C |
| 16 | D |
| 17 | A |
| 18 | A |
| 19 | A |
| 20 | A |
| 21 | A |
| 22 | C |
| 23 | B |
| 24 | D |
| 25 | A |
| 26 | A |
| 27 | D |
| 28 | C |
| 29 | A |
| 30 | A |
| 31 | D |
| 32 | A |
| 33 | A |
| 34 | A |
| 35 | C |
| 36 | C |
| 37 | D |
| 38 | C |
| 39 | D |
| 40 | C |
| 41 | A |
| 42 | D |
| 43 | D |
| 44 | C |
| 45 | A, E |
| 46 | B |
| 47 | A |
| 48 | D |
| 49 | D |
| 50 | A |
| 51 | B |
| 52 | C |
| 53 | C |
| 54 | B |
| 55 | B |
| 56 | C |
| 57 | A |
| 58 | B, E |
| 59 | D |
| 60 | C |
| 61 | A |
| 62 | B |
| 63 | D |
| 64 | B |
| 65 | D |
| 66 | A |
| 67 | A |
| 68 | D |
| 69 | D |
| 70 | A |
| 71 | A |
| 72 | C |
| 73 | C |
| 74 | B |
| 75 | C |
| 76 | B |
| 77 | A, B |
| 78 | A |
| 79 | A |
| 80 | B |
| 81 | B |
| 82 | A |
| 83 | A |
| 84 | A |
| 85 | B |
| 86 | D |
| 87 | D |
| 88 | B |
| 89 | A |
| 90 | B |
| 91 | B |
| 92 | A |
| 93 | D |
| 94 | A |
| 95 | C |
| 96 | B |
| 97 | D |
| 98 | D |
| 99 | C |
| 100 | D |
| 101 | B |
| 102 | A |
| 103 | A |
| 104 | A |
| 105 | C |
| 106 | B |
| 107 | B |
| 108 | B |
| 109 | C |
| 110 | D |
| 111 | D |
| 112 | B |
| 113 | A |
| 114 | D |
| 115 | C |
| 116 | A |
| 117 | D |
| 118 | C |
| 119 | A |
| 120 | B |
| 121 | B |
| 122 | C |
| 123 | B |
| 124 | B |
| 125 | A |
| 126 | B |
| 127 | D |
| 128 | D |
| 129 | A |
| 130 | B |
| 131 | A |
| 132 | C |
| 133 | C |
| 134 | B |
| 135 | D |
| 136 | B |
| 137 | D |
| 138 | C |
| 139 | A |
| 140 | B |
| 141 | A |
| 142 | B |
| 143 | A |
| 144 | B |
| 145 | C |
| 146 | C |
| 147 | A |
| 148 | C |
| 149 | A |
| 150 | D |
| 151 | D |
| 152 | C |
| 153 | D |
| 154 | B |
| 155 | B |
| 156 | C |
| 157 | A |
| 158 | A |
| 159 | B |
| 160 | B |
| 161 | D |
