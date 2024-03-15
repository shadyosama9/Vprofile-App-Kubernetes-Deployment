## Overview:
This project deploys a multi-tier application on a Kubernetes cluster using kops. The application consists of the following components:

- `vproapp`: Backend service
- `vprodb`: MySQL database
- `vpromc`: Memcached service
- `vprormq`: RabbitMQ service

## Prerequisites:
Before you begin, ensure you have the following installed:

- `kubectl`: Kubernetes command-line tool
- `kops`: Kubernetes Operations tool
- AWS CLI configured with appropriate permissions
- Access to an AWS S3 bucket for storing Kubernetes cluster state


## Steps:

1. **Create Kubernetes Cluster:**

   Use kops to create a Kubernetes cluster on AWS:

   ```sh
   kops create cluster --name=mycluster.k8s.local --state=s3://k8s-bucket --zones=us-east-1a,us-east-1b --node-count=2 --node-size=t3.small --master-size=t3.medium --dns-zone=mycluster.k8s.local --node-volume-size=8 --master-volume-size=8

   kops update cluster --name mycluster.k8s.local --state=s3://k8s-buket --yes --admin

   ```
2. **Validate The Cluster**

   Make Sure To Check For The Cluster's Health

   ```sh
   kops validate cluster --name mycluster.k8s.local --state=s3://kops-state-2024
   ```

3. **Create an EBS Volume On AWS For The Database:**

   Using AWS CLi

   ```sh
   aws ec2 create-vloume --availability-zone=us-east-1a --size=3 --volume-type=gp2
   ```

4. **Tag The Volume With The Following Tag:**

   KubernetesCluster = mycluster.k8s.local

4. **Label The Node In The Same Zone As The Volume:**
  
   Using kubectl

   ```sh
   kubectl label node <node-name> zone=us-east-1a
   ```

5. **Change The Volume ID**

   Update The Vloume ID in DB/vprodb-deploy.yml With Your Volume ID


6. **Deploy The App:**
  
   Using kubectl

   ```sh
   kubectl apply --recursive -f .
   ```

7. **Clean Up**

   When You're Done Make Sure To Destroy The Cluster To Avoid Extra Charges

   ```sh
   kops delete cluster --name  mycluster.k8s.local--state=s3://kops-bucket --yes
   ```


## Notes

- Replace mycluster.k8s.local with your cluster's DNS name.
- Replace k8s-bucket with S3 bucket name.
- Ensure your AWS credentials are configured correctly.