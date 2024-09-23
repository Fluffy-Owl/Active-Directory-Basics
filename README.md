# Active Directory Basics

## Introduction

Active Directory (AD) is a directory service developed by Microsoft for managing permissions and access to network resources in a Windows domain environment. It allows centralized control of user accounts, devices, and network services within organizations. This Active Directory lab is simulated in a TryHackMe room which provides an introduction to AD concepts, components, configurations and AD's Forests and Domain Trust.

## Task Breakdown

### Task 1: Introduction to Active Directory

The first task introduced the core concepts of Active Directory, including the purpose of **Domains**, **Organizational Units (OUs)**, and **Domain Controllers (DCs)**. I learned how AD facilitates centralized management of users, computers, and other network resources. The main idea behind a domain is to centralise the administration of common components of a Windows computer network in a single repository called **Active Directory (AD)**. The server that runs the Active Directory services is known as a **Domain Controller (DC)**.

**The main advantages of having a configured Windows domain are:**
- Centralised identity management: All users across the network can be configured from Active Directory with minimum effort.
- Managing security policies: You can configure security policies directly from Active Directory and apply them to users and computers across the network as needed.

**Understanding between Security Groups vs Organisational Units (OUs):**

- **OUs** are handy for **applying policies** to users and computers, which include specific configurations that pertain to sets of users depending on their particular role in the enterprise. Remember, a user can only be a member of a single OU at a time, as it wouldn't make sense to try to apply two different sets of policies to a single user.
- **Security Groups**, on the other hand, are used to **grant permissions over resources**. For example, you will use groups if you want to allow some users to access a shared folder or network printer. A user can be a part of many groups, which is needed to grant access to multiple resources.


### Task 2: Delegation in Active Directory

Delegation is a powerful feature within Active Directory that allows specific users to control certain aspects of Organizational Units (OUs) without requiring full Domain Administrator privileges. This process is useful for delegating day-to-day administrative tasks, such as resetting user passwords or managing accounts, to trusted individuals or support staff.

In this example, **Phillip**, who is in charge of IT support, needs to be granted control to reset passwords for users in the **Sales**, **Marketing**, and **Management** OUs. We'll walk through how to delegate control over the **Sales OU** to Phillip, allowing him to manage password resets without needing Domain Admin intervention.

![Screenshot 2024-09-24 065137](https://github.com/user-attachments/assets/4cb2dfa8-2f18-4319-bf85-17da2ff3abc2)

#### Steps to Delegate Control

1. Open the **Active Directory Users and Computers** tool.
2. Right-click on the **Sales OU** and select **Delegate Control**.
   
3. In the **Delegation of Control Wizard**, add **Phillip** as the user to whom you want to delegate control. To avoid typing errors, type "phillip" and click the **Check Names** button to verify the user.
   
4. Select the task that you want to delegate, in this case, choose the option to **Reset user passwords** and click **Next** a few times to complete the delegation process.

At this point, Phillip should now have the ability to reset passwords for users in the **Sales OU**.
![Screenshot 2024-09-24 065351](https://github.com/user-attachments/assets/2928038b-7df7-416f-8210-b931cf3034c5)

#### Testing Phillip's Delegation

Phillip can now reset user passwords via PowerShell. We will log in using Phillip's credentials and attempt to reset **Sophie's** password using the following steps:

1. Log into the system using Phillip's credentials
   
2. Open PowerShell and run the following command to reset **Sophie's** password:
   ```powershell
   Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
   ```

3. To ensure that Sophie changes her password at the next logon, run:
   ```powershell
   Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
   ```
![image](https://github.com/user-attachments/assets/f79618da-28a4-4e1a-9e97-535e37d8ab38)

4. Now, log in as Sophie using the new password to verify the reset.

By delegating tasks like password resets to support staff such as Phillip, the workload on Domain Administrators is reduced, and IT teams can resolve user issues faster. This task demonstrates how to securely delegate control while maintaining proper security boundaries within the AD environment.


### Task 3: AD Group Policies

Group Policy Objects (GPOs) are used in Active Directory environments to manage and enforce configurations and security settings across users and computers. GPOs can be applied to Organizational Units (OUs) and linked to specific parts of the AD hierarchy to target certain users or machines, ensuring that policies are applied where needed. Each GPO can contain settings for users, computers, or both.

Group Policy Objects (GPOs) are a core component of Active Directory management. They allow administrators to define settings and enforce them across users and computers in the domain. This task covered the creation, application, and management of GPOs, including how to scope policies to specific OUs and filter them based on security requirements. The policies included:
- **Password Policies**: Enforcing minimum password lengths and complexity.
- **Screen Lock Policies**: Automatically locking workstations after inactivity.
- **Control Panel Access**: Restricting non-IT users from accessing system settings.

For example, GPOs like the **Default Domain Policy** or **RDP Policy** can apply to the entire domain, while others like the **Default Domain Controllers Policy** apply only to specific OUs, such as the **Domain Controllers** OU.

Key aspects of GPO management:
- **Scope**: GPOs can be linked to OUs, and they also apply to all child OUs. This allows an administrator to apply policies to large sections of the AD hierarchy.
- **Security Filtering**: By default, GPOs apply to all authenticated users. However, they can be filtered to apply only to specific users or groups, offering more granular control over who the policies affect.
  
In this task, I explored the configuration of GPOs, such as enforcing a minimum password length of 10 characters in the **Default Domain Policy**. Here’s the process I followed to modify the GPO:

1. Open the **Group Policy Management Tool**.
2. Select the **Default Domain Policy** and click "Edit."
3. Navigate to:
   ``` 
   Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies -> Password Policy
   ```
4. Adjust the **Minimum Password Length** policy to require 10 characters.

![Screenshot 2024-09-24 065137](https://github.com/user-attachments/assets/59745629-acd7-4416-b1c1-d0297d7d2bcb)


Additionally, I created new GPOs to address specific organizational needs:
- **Restrict Control Panel Access**: This policy was applied to prevent non-IT users from accessing the Control Panel by linking it to the Marketing, Management, and Sales OUs. I enabled the "Prohibit Access to Control Panel and PC settings" policy under the **User Configuration**.
- **Auto Lock Screen GPO**: I configured a new GPO to automatically lock workstations and servers after 5 minutes of inactivity. This GPO was applied at the root domain level to affect all computers across the network, ensuring that the policy applied to all child OUs, including Workstations, Servers, and Domain Controllers.

![image](https://github.com/user-attachments/assets/e4f6e8b3-9154-42b6-a99f-dd579842cc19)


To enforce GPO changes immediately, I used the following command:
```bash
gpupdate /force
```

This task demonstrated how powerful GPOs are in controlling and securing an Active Directory environment, and how they can be tailored to the specific needs of different organizational units.


### Task 4: Authentication Methods

In Windows domains, credentials are stored on Domain Controllers (DC). When a user tries to authenticate to a service using domain credentials, the service contacts the DC for verification. Two protocols are used for network authentication in Windows domains:

- **Kerberos**: Default protocol in recent Windows versions.
- **NetNTLM**: Legacy protocol, kept for compatibility.

Most networks have both protocols enabled, although NetNTLM is considered obsolete.

#### Kerberos Authentication Overview

Kerberos is the default authentication protocol for modern Windows. Users logging into services are assigned tickets, which serve as proof of authentication. The process works as follows:

1. **Ticket Granting Ticket (TGT) Request**:  
   The user sends their username and a timestamp encrypted with a key derived from their password to the Key Distribution Center (KDC) on the Domain Controller. The KDC returns a TGT, allowing the user to request additional tickets for specific services, along with a Session Key for further requests.
   
![Screenshot 2024-09-24 070916](https://github.com/user-attachments/assets/0e125b26-8aa2-45c8-b712-291d09f1c326)

3. **Ticket Granting Service (TGS) Request**:  
   When accessing a specific service (e.g., a website or database), the user sends the TGT, a Service Principal Name (SPN), and an encrypted timestamp to the KDC. The KDC then provides a TGS and a Service Session Key, which are used to authenticate to the desired service.
   
![Screenshot 2024-09-24 070949](https://github.com/user-attachments/assets/20f242ca-2e3b-437d-be1d-8038a7112200)

4. **Service Authentication**:  
   The TGS is sent to the target service, which decrypts it using its account's password hash to verify the Service Session Key, establishing a connection.
   
![Screenshot 2024-09-24 071018](https://github.com/user-attachments/assets/932a0bc1-0050-4764-a52b-bdcf96606dbe)

Kerberos ensures users do not repeatedly pass their credentials when connecting to services across the network.

### NetNTLM Authentication

NetNTLM uses a challenge-response mechanism for authentication. The process works as follows:

![Screenshot 2024-09-24 071302](https://github.com/user-attachments/assets/7bb1c390-21fa-4cc7-9e25-a148d83cec42)

1. The client sends an authentication request to the server.
2. The server generates a random number (challenge) and sends it to the client.
3. The client combines their NTLM password hash with the challenge to create a response, then sends it back to the server.
4. The server forwards the challenge and response to the Domain Controller (DC) for verification.
5. The DC recalculates the response using the challenge and compares it with the client's response. If they match, the client is authenticated. If not, access is denied.
6. The server forwards the authentication result to the client.

Note that the user's password or password hash is never transmitted over the network, ensuring security.

### Task 5: Trees, Forests, and Trusts

#### Trees
![image](https://github.com/user-attachments/assets/81b61d47-cda2-4015-a990-70ab151db267)

When expanding to new locations with different laws and IT teams, managing resources for each can become complex. Instead of creating a large OU structure, Active Directory allows multiple domains to be integrated into a **Tree**. Domains that share the same namespace can be grouped in a tree, making management easier.

For example, a company with a domain `thm.local` could split into two subdomains `uk.thm.local` and `us.thm.local`. Each subdomain has its own Domain Controller (DC), users, and resources:

- **Subdomains**: UK IT admins control the UK domain, and US admins control the US domain.
- **Independent Management**: Domain admins for each subdomain have control over their own resources, and policies can be set independently.
  
An important group in tree structures is **Enterprise Admins**, which grants administrative privileges across all domains. However, each domain still has its own **Domain Admins** with control limited to their respective domain.

#### Forests

As a company grows, it may acquire another organization with its own domain tree. A **Forest** is the union of multiple domain trees with different namespaces in the same network.

For example, after acquiring `MHT Inc.`, the company may manage two separate domain trees: `thm.local` and `mht.local`. This structure allows each tree to operate independently, yet still be part of the same network.

#### Trust Relationships

Trust relationships link domains within trees and forests, allowing cross-domain access to resources:

- **One-Way Trust**: Domain A trusts Domain B, so users from Domain B can access resources in Domain A.
- **Two-Way Trust**: Both domains mutually trust each other, allowing resource access in both directions.

Trust relationships don't automatically grant access; they provide the option to authorize cross-domain access as needed.

### Conclusion

In this Active Directory lab, I explored the core components and functionality of Active Directory (AD) in a structured environment, gaining hands-on experience in various AD management tasks. From understanding the basic concepts of **Domains**, **Organizational Units (OUs)**, and **Domain Controllers (DCs)** to configuring advanced features like **Delegation**, **Group Policies**, and **Trust Relationships**, the lab provided a comprehensive introduction to the power and flexibility of AD in managing network resources and users.

I learned how AD enables centralized control over users and devices, ensuring efficient management and security across large networks. The ability to delegate tasks such as password resets, coupled with the control offered by **Group Policy Objects (GPOs)**, allows for fine-grained management tailored to an organization's specific needs.

Furthermore, understanding **Trees, Forests, and Trusts** provided insight into how organizations scale their AD infrastructure across multiple domains and locations while maintaining seamless cross-domain access and resource sharing.

Overall, this lab underscored the importance of AD in modern IT environments, where centralized management and security are critical for operational efficiency and compliance.

Created by Azrul

---

## References
- [TryHackMe Active Directory Basics Room](https://tryhackme.com)
- [PowerView Documentation](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
- [Microsoft AD Documentation](https://docs.microsoft.com/en-us/windows-server/identity/active-directory-domain-services)

---
