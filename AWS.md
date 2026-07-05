1. Cloud Computing: The on-demand delivery of IT resources, particularly compute power, application hosting, database applications, networking, and more.
2. Cloud Computing Models: All-in-cloud, on-premises (not-in-cloud), Hybrid
3. All-in-cloud:
  1. Usually startups of all sizes. Most businesses started after 2011.
  2. Migrates existing projects to the cloud (if they exist)
  3. All new projects are in the cloud.
4. On-premises:
  1. Minimal to no cloud computing usage.
  2. All projects run in an owned data center or a rented data center
  3. Responsible for all security and operations.
  4. Legacy companies that don't have a reason to move to the cloud.
  5. Companies that need strict control and security over the infrastructure (example like government organizations).
5. Hybrid
   1. Some parts run in the cloud and others on-premises.
   2. Migrating existing applications to the cloud, leaving legacy applications on-premises.
   3. Most new applications are designed and built for the cloud.
   4. Usually, there is a fast connection between on-premises and cloud resources.
6. 3 ways to interact with AWS
   1. Amazon Console (The user interface)
   2. AWS CLI (Command Line Interface)
   3. AWS SDKs -> For developers and different types of coders
7. Advantages of AWS
   1. Upfront expenses (CAPEX) -> Large upfront investment in hardware before use
      is traded for variable expenses (OPEX) -> pay for usage and requests. Give back what you don't need
   2. Focus on the data center is replaced with focus on customers.
   3. Limited scalability is replaced by extreme scalability.
   4. The more you consume in AWS, the less you pay per unit price (economies of scale).
   5. Increased provisioning speed and business agility.
   6. Global deployment options (AWS has 31 large locations globally, and hundreds of edge locations) and can go up in minutes.
8. Types of payment:
   1. Free tier -> no need to pay as long as you are in the free tier period.
   2. On-demand -> only pay for what you used for how long.
   3. Reservations -> We commit in terms of years and get a deep discount on the usage. There is no cancellation midway if you have committed once.
   4. Volume Standards (S3 standard) -> The more you use, the lower it costs per unit.
   5. Usually, on the release of a new version, every time AWS cuts the prices.
9. Cloud Design principles:
    1. Design for failure -> Increase resilience and auto-recovery by adding redundancy.
    2. Decouple components -> In case of a surge in traffic, there should be no data loss. So, we loosely couple with a queue or scaling layer between components.
    3. Implement Elasticity -> Scale up and down as needed. Better costs and better performance. This can be done automatically.
    4. Parallel computing -> Increased concurrency (the ability of a system to execute multiple tasks simultaneously or in an overlapping manner. Rather than completing one task entirely before starting the next, a concurrent system processes multiple tasks at the same time, significantly improving program responsiveness and resource utilization)
