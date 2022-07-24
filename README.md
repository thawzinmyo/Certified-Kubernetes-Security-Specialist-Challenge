# Certified Kubernetes Security Specialist Challanges Walkthrough!!!

KodeKloud announced CKS Challanges last two months that I saw on LinkedIn. When they challange fot that, I want to participate immediately! But on that time, I faced some lost in life. I can't concentrate for that. So, I can't take time for that challenges. After that time passed, I start for that **Challenges** which challenging on Kubernetes Security to solve using third party tools or native ways. Yeah, let's check the overview.

![The CKS Challenges](https://github.com/thawzinmyo/Certified-Kubernetes-Security-Specialist-Challenge/blob/master/image/CKS%20Intro/CKS_Intro.PNG)

- It's Free and here is the link for "[CKS Challenges](https://kodekloud.com/courses/cks-challenges/)"
- Participate challenges for CKS Exam preparation or Hands-on. Be enjoy learning and practing, Good Luck!

Need to complete the following challenges.

1. [Challange_1](https://github.com/thawzinmyo/Certified-Kubernetes-Security-Specialist-Challenge/blob/master/Challenge1.md)

   - Challenge 1: There are 6 images listed in the diagram on the right. Using Aquasec Trivy (which is already installed on the controlplane node), identify the image that has the least number of critical vulnerabilities and use it to deploy the alpha-xyz deployment. Secure this deployment by enforcing the AppArmor profile called custom-nginx. Expose this deployment with a NodePort type service and make sure that only incomings connections from the pod called middleware is accepted and everything else is rejected. 


2. [Challange_2](https://github.com/thawzinmyo/Certified-Kubernetes-Security-Specialist-Challenge/blob/master/Challenge2.md)

   - Challenge 2: A number of applications have been deployed in the dev, staging and prod namespaces. There are a few security issues with these applications. Inspect the issues in detail by clicking on the icons of the interactive architecture diagram on the right and complete the tasks to secure the applications.


3. [Challenge_3](https://github.com/thawzinmyo/Certified-Kubernetes-Security-Specialist-Challenge/blob/master/Challenge3.md)
   
   - Challenge 3: This is a two node kubernetes cluster. Using the kube-bench utility, identify and fix all the issues that were reported as failed for the controlplane and the worker node components.


4. [Challenge_4](https://github.com/thawzinmyo/Certified-Kubernetes-Security-Specialist-Challenge/blob/master/Challenge4.md)
   - Challenge 4: There are a number of Kubernetes objects created inside the omega, citadel and eden-prime namespaces. However, several suspicious/abnormal operations have been observed in these namespaces!. For example, in the citadel namespace, the application called webapp-color is constantly changing! You can see this for yourself by clicking on the citadel-webapp link and refreshing the page every 30 seconds. Similarly there are other issues with several other objects in other namespaces. To understand what's causing these anomalies, you would be required to configure auditing in Kubernetes and make use of the Falco tool.
   

5. [Challenge_5](https://github.com/thawzinmyo/Certified-Kubernetes-Security-Specialist-Challenge/blob/master/image/C5/Challenge5.PNG)
   - Challenge 5: Coming Soon

Click above each link to see my walkthrough.

I got a lot of hands-on experience and theory from these CKS challenges. I have learned new tools such as "[Auqasec Trivy](https://www.aquasec.com/products/trivy/) - [GitHub Source for trivy](https://github.com/aquasecurity/trivy)", "[Kubesec](https://kubesec.io/)", "[Falco](https://sysdig.com/opensource/falco/)", "[Kube Bench](https://github.com/aquasecurity/kube-bench)", "[CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)" and "[AppArmor](https://apparmor.net/)". I solved network policy tricks, RBAC, seccomp, Kube-APISERVER's health, Static YAML tricks, scanning the YAML File and image vulnerability, "[Auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)" etc. Click each link for more details walkthrough ...
