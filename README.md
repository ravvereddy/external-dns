# What is External-dns

- ExternalDNS is a Kubernetes tool that automates the management of DNS records for services and ingresses running in a Kubernetes cluster. When using ExternalDNS with AWS Route53, the following steps occur behind the scenes:

 - ExternalDNS is deployed as a Kubernetes deployment and service within the cluster. When deployed, ExternalDNS queries the Kubernetes API server for services and ingresses that have certain annotations indicating they should be managed by ExternalDNS.

 - When ExternalDNS finds a service or ingress with the appropriate annotations, it checks if the service or ingress has an associated hostname or DNS name specified. If a hostname is present, ExternalDNS then queries the AWS Route53 API to check if the hostname already exists in the specified Route53 hosted zone.

 - If the hostname does not exist in the Route53 hosted zone, ExternalDNS creates a new DNS record in Route53 using the specified hostname and the IP address of the Kubernetes service. If the hostname already exists, ExternalDNS updates the existing record with the new IP address.

 - In some cases, ExternalDNS may need to verify ownership of the domain before it can create DNS records. For example, when creating a new hosted zone in Route53 or when using certain DNS providers. To do this, ExternalDNS creates a temporary TXT record in the DNS zone with a specific value, and then checks that the record exists to verify ownership.

 - ExternalDNS also supports various customizations to DNS record behavior through the use of TXT records. For example, you can use a TXT record to specify a custom TTL for a DNS record, or to define additional options for the DNS provider.

Overall, ExternalDNS simplifies the process of managing DNS records for Kubernetes services and ingresses by automating the creation and management of DNS records in AWS Route53. By using ExternalDNS, you can ensure that your Kubernetes services and ingresses are always accessible via a custom domain name, without having to manually manage DNS records.

# Explain how external dns works

- ExternalDNS is a tool that automates the management of DNS records for Kubernetes services by creating or updating DNS records in a DNS provider such as AWS Route53. Here's how ExternalDNS works with AWS Route53:

 - ExternalDNS watches Kubernetes services: ExternalDNS monitors Kubernetes services and ingresses for changes to their endpoints (i.e., IP addresses and ports).

 - ExternalDNS updates DNS records: When a change is detected, ExternalDNS updates the corresponding DNS record in AWS Route53 to point to the new endpoint. If the DNS record doesn't exist, ExternalDNS creates a new record.

 - Optional TTL customization: You can customize the TTL (time-to-live) of DNS records created by ExternalDNS. By default, ExternalDNS uses a TTL of 300 seconds, but you can configure it to use a different value if desired.

 - IAM Role configuration: In order to access and modify Route53 DNS records, ExternalDNS needs an IAM Role with the necessary permissions. You can create an IAM Role with a policy that allows ExternalDNS to manage DNS records in Route53.

 - Annotation configuration: Finally, you need to configure ExternalDNS to use AWS Route53 as the DNS provider for your Kubernetes cluster by adding the appropriate annotations to your Kubernetes resources (services, ingresses, etc.). Here is an example annotation for a service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  annotations:
    external-dns.alpha.kubernetes.io/hostname: my-service.example.com
    external-dns.alpha.kubernetes.io/target: my-route53-zone-id
```

In this example, the external-dns.alpha.kubernetes.io/hostname annotation specifies the DNS name to use for the service, and the external-dns.alpha.kubernetes.io/target annotation specifies the Route53 zone ID to use for managing the DNS record.


# Here's an example of how ExternalDNS works with AWS Route53:

- First, you'll need to set up ExternalDNS in your Kubernetes cluster. This typically involves creating a Kubernetes deployment and service for ExternalDNS, along with any necessary RBAC (role-based access control) rules to grant ExternalDNS permission to modify DNS records in your Route53 zone.

- Once ExternalDNS is up and running, you can create Kubernetes services and ingresses for your applications as usual. For example, you might create a service like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - name: http
    port: 80
    targetPort: http
```
This service defines a selector that matches pods with the label app=my-app, and exposes port 80 on a cluster-internal IP address.

To tell ExternalDNS to create a DNS record for this service in Route53, you can add some annotations to the service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
  annotations:
    external-dns.alpha.kubernetes.io/hostname: my-app.example.com
    external-dns.alpha.kubernetes.io/target: my-route53-zone-id
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - name: http
    port: 80
    targetPort: http
```
- Here, the external-dns.alpha.kubernetes.io/hostname annotation tells ExternalDNS to create a DNS record for the service with the hostname my-app.example.com, while the external-dns.alpha.kubernetes.io/target annotation specifies the Route53 zone ID where the record should be created.

- When ExternalDNS sees the updated service with these annotations, it will create a DNS record in Route53 for the specified hostname, pointing to the IP address of the service. If the service's IP address changes, ExternalDNS will automatically update the DNS record accordingly.

- By using ExternalDNS to automate the creation and management of DNS records in AWS Route53, you can simplify the process of making your Kubernetes applications available on the internet with custom domain names.

# Note

- TXT records are used by ExternalDNS to perform various actions when managing DNS records. Here's how it works in more detail:

- Verifying ownership of a domain: When using ExternalDNS with some DNS providers, such as AWS Route53, you need to verify that you own the domain before you can create DNS records for it. To do this, ExternalDNS will create a TXT record with a specific value in the domain's DNS zone, and then check to make sure the record exists. Once the record is verified, ExternalDNS can create or modify DNS records for the domain.

- Defining custom settings for DNS records: TXT records can also be used to specify custom settings for DNS records created by ExternalDNS. For example, you can use a TXT record to set the TTL (time-to-live) for a DNS record, or to specify additional options for the DNS provider.

# AWS Route53 Limitations
- While TXT records can be used with AWS Route53, there are some limitations to be aware of. For example, Route53 has a maximum limit on the size of TXT records, which can limit the amount of custom data that can be stored in a TXT record. Additionally, Route53 may cache TXT records for longer than the specified TTL, which can cause delays in updating DNS records.

- Overall, while TXT records are an important part of ExternalDNS, their use may be limited depending on the DNS provider being used, and care should be taken to ensure that TXT records are used appropriately to avoid unexpected behavior.
