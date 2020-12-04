Note that a EKS Control Plane is not managed by us (customer)
It is managed by AWS and exists in a VPC managed by AWS.
Our subnets/security groups need to allow traffic from the external EKS control plane


1. Create VPC with three private subnets and three public subnets (or two public and two private subnets)
2. The private subnets will be used for the data plane.
3. the public subnets should have a tag with following properties 
-  Key: kubernetes.io/role/elb, 
  Value: 1  
  The private subnets should have a tag as follows:         
         - Key: kubernetes.io/role/internal-elb
          Value: 1
Also create a NATGateway which is in one of the public subnets.     The private route table should have a route to this NATGateway , so that our worker nodes in data plane can connect to Internet via the NATGateway 

4. A VPC endpoint is recommended to be created to allow communication between our worker nodes and EKS control plane via VPC endpoint (rather than over the Internet). This makes the communication between control plane and worker nodes faster 
5. Then create a AWSServiceRoleForAmazonEKS which has following managed policies:
AmazonEKSServicePolicy, AmazonEKSClusterPolicy, AmazonRoute53FullAccess
6. Then create a ControlPlaneSecurityGroup which EKSCluster can assume (actually external ControlPlane will need this security group)
7. Then create a NodeInstanceRole , this IAM role will have some policies which nodes in DataPlane can assume
Also create a NodeInstanceProfile based on the NodeInstanceRole
8. Then create a NodeSecurityGroup , a security group for nodes in dataplane
9. Then create NodeSecurityGroupIngress, for allowing nodes in dataplane to communicate with each other.
10. Then create a ClusterControlPlaneSecurityGroupIngress, which allows nodes in data plane to communicate with cluster control plane
11. Then create a ControlPlaneEgressToNodeSecurityGroup, which allows outbound from control plane to data plane nodes
12. Then create ControlPlaneEgressToNodeSecurityGroupOn443, which allows outbound from control plane to data plane nodes over port 443
13. Then create NodeSecurityGroupFromControlPlaneIngress, which allows data plane to get inbound traffic from control plane 
14. Then create a NodeSecurityGroupFromControlPlaneOn443Ingress, which allows data plane to get inbound traffic from control plane .
15. Then create a NodeLaunchConfig with UserData that bootstraps EKS agent (kubelet) at startup. This will make it possible for all nodes being launched in EC2 autoscaling group to be part of the EKS cluster
16. Then create a NodeGroup called auto scaling group. This will use the NodeLaunchconfig created in prev. step.
You also need to give some tags specific to Kubernetes
You also need to specify the three private subnets to be used for the data plane
17. Then create a scaling policy for the above NodeGroup
18. Then create any other remaining custom policies and attach them to the NodeInstanceRole
