
<h2 align="center"># GCP Architect Learnings <br/> - Storage </h2>

<h2 align="center">1. Creating buckets</h2>

Native Google Cloud storage addresses start with gs:// <br/>
Just like web addresses start with https://

Used for:  
`Uploading & managing objects` \ 
`(Public and Private)`
   
<details>
 <summary>Storage Bucket types</summary>
 ➟ Multi-Regional<br>
  ・These buckets are designed for storing data that requires high availability and low latency access from multiple regions.
  <br>
 ➟ Regional<br>
  ・for redundancy. Regional buckets are ideal for applications that have regional data requirements, such as analytics, backups, and content distribution within a specific geographic area.
  <br>
 ➟ Nearline<br>
  ・for storing infrequently accessed data that may need to be accessed within seconds or minutes. with a slightly higher access latency. This type of bucket is suitable for data that needs to be available for quick retrieval but is not accessed frequently, such as backup archives, disaster recovery data, and long-term storage.
  <br>
➟ Coldline<br>
  ・designed for long-term archival storage of data that is rarely accessed. accessed no more than once per year, such as regulatory or compliance archives, historical records, and large-scale data archiving.
  <br>
</details>
   
---
<h2 align="center">2. Commands for storage</h2>

<pre>
- Gsutil connects to google cloud storage
- Double star lists out files even from folders 
   - gsutil ls gs://bucket-name/**
- Make Bucket
   - gsutil mb ( -l location-name) (gs://bucket-name)
- gsutil label (get gs://)[to view] 
   - (get gs://bucket > bucketlabels.json)[to export] 
   - (set bucketlabels.json gs://)[to set from file] 
   - (ch -l "key":"key" gs://bucket)[shortcut to add to existing labels]
- Check file versioning. Off by default. NOT possible on Console, only on CLI
   - gsutil versioning get gs://
   - Enable = gsutil versioning set on gs://
- View file Version with the generation-number even after its deleted
   -  gsutil ls -a gs:// 
- Make object public
   - gsutil acl ch -u AllUsers:R gs://bucket/img.jpg
     - acl = access control, -u =user, R = read 
</pre>

<h2 align="center">3. Commands for shell</h2>

<pre>
-- gcloud config get-value project
-- gcloud services list--enabled
-- gcloud services list--available | grep compute
-- gcloud compute instances create vm-name
-- screenshots  
-- gcloud compute instances list [see existing vms]
-- gcloud compute machine-types list
-- gcloud topic filters [we'll use it instead of grep so to remove paid f1 zones like north Virginia].
-- 
-- CREATING VM ⬇
-- gcloud compute machine-types list--filter="NAME: f1-micro AND ZONE~us-west" [free tier]
-- gcloud config set compute/zone us-west2-b
-- gcloud config set compute/region us-west2
-- ⮕ gcloud compute instances create-machine-type=f1-micro vm-name
-- ping-c 3 $external_ip_of_vm [name or internal ip didn't work]
-- 
-- SSH INTO VM ⬇
-- ssh $external_ip [this doesn't work]
-- ⮕ gcloud compute ssh $vm_name [this works]
-- {A public and private ssh key is now created. Public key can be viewed from Compute Engine ↠ Metadata ↠ SSH keys. That key is now registered. So any future instances they get created will automatically know to accept this for log in.}
-- 
-- CHECK SESSION IP ⬇
-- ⮕ curl api.ipify. org 
-- 
-- Where our Public KEY is saved ⬇
-- {After our first time ssh, two files: google_compute_engine (encrypted PRIVATE KEY) & google_compute_engine.pub (PUBLIC KEY) files are created into .ssh directory. Both are needed to ssh into the machine}
-- whoami ➟ owner details
-- hostname ➟ current user
-- 
-- HOW DOES VM KNOWS OUR PUBLIC KEY: ⬇
-- (run inside ssh)
-- cd .ssh/ ➟ cat authorized_keys [Contains our PUBLIC KEY]
-- curl-H "Metadata-Flavor:Google" metadata. google.internal/computeMetadata/v1/$
-- $/instance/name [our vm name]
-- $/project/attributes/ssh-keys [this contains the key that identifies us, OUR PRIVATE KEY]
-- $/project/project-id [this tells our project name]
-- $/instance/service-accounts/default/ [see info about the service account for this VM instance, like aliases, email, identity, scopes, token]
-- gcloud config list [see metadata for our project]
-- 
-- DELETE INSTANCE: ⬇
-- gcloud compute instances delete myhappyvm
</pre>

<h2 align="center">4. GCP Console - Compute Engine</h2>

<pre>
When creating a Compute Instance, <br/>
⮕ In the Identity and API access section <br/>
We have our Service Account selected to give VM access to Google services. <br/>but an interesting and important thing to understand is that even when we've given access to a particular service account <br/>these access scopes restrict what you can do with it. <br/>
If you look at the "Access scopes" under Service Account,  <br/>⮕ By default you only get read access to Google cloud storage and you don't actually get access to Google compute engine <br/>which is why we were unable to terminate the instance that we were using when we shelled into it.<br/>
If you want you can "[x]Set access for each API" to send access on a service-by-service basis.<br/>
Or [x] Allow full access to all Cloud APIs. Which we will choose for this exploration lab.<br/>
⮕ Here are all the Default Scope settings and the Best Scope Practices: https://cloud.google.com/compute/docs/access/service-accounts#default_scopes
⮕ Now if you try to delete VM after ssh-ing, you can see in the activity logs <br/>That the last two activities to delete the VM were performed by the service account being used by compute engine instance, <br/>and not our user account.
example: 339365196943-compute@developer.gserviceaccount.com deleted myvmname.

⮕ There is an option to Enable deletion protection, which is not always the best practice. <br/>
Also, there is another checkbox, <br/>"[x]Delete boot disk when instance is deleted" which is ticked, <br/>because the default behavior is not to be preserving these instances, <br/>you start them up -> They do some stuff -> they go away. <br/>Going away should be fine because the doing some stuff part should always be writing everything that matters somewhere else. <br/>
Now to store things, we can add persistent disks, but that doesn't go well with auto scaling, <br/>so you often want to use a separate service to store the data instead.<br/>

⮕ Now in the advanced settings section of compute settings:<br/>
we also have a "Security" tab and here's where we could override the project level settings about SSH keys <br/>and block them from being used and add custom ones just for this instance.<br/>
You might want to use this if you have special machines
that shouldn't be available to most people in the project <br/>or some machines where you want to allow specific access to those machines and not the others.

Now for the disks, remember that the default behaviour is to delete the boot disk whenever the instance goes away <br/>though you could either add a new one or attach an existing persistent disk that you had created. <br/>
For Key's encryption, It's generally easiest to let [x]Google manage the keys<br/>
And if possible though, you should try to avoid [x]Customer-supplied key, i.e, having to manage the keys yourself outside of Google.

⮕ For Security features, If we go to the ➟ menu ➟ to Compute ➟ look at Metadata ➟ and then switch to the SSH keys tab. <br/>
We'll see new Public SSH KEYs added every time when we ssh from the CONSOLE, as a part of connection from the console to the instance.<br/>
After running "gcloud compute ssh" the public key is put in this project metadata spot <br/>and it stored the private key on the file system that was running gcloud <br/>which happened to be cloud shell in our this case.<br/> 
But when ssh=ing from console, look at the new keys added...<br/>
They have a new "expireOn:" tag at the end, so Google will automatically remove these keys in a short while. <br/>They last long enough for us to make the connection to the instance <br/>but not long enough for them to become an issue of someone stealing these credentials and using them to mount an attack.

</pre>
→ Service Accounts: https://cloud.google.com/compute/docs/access/service-accounts <br/>
→ VM instance lifecycle: https://cloud.google.com/compute/docs/instances/instance-life-cycle <br/>
→ About VM metadata: https://cloud.google.com/compute/docs/metadata/overview#waitforchange <br/>
・Note: We cannot pull logs from instance, The instances are set up to push their logs to Google Cloud operations (formerly Stack-driver) through the agent. <br/>
Note: Metadata service uses the http service, not https <br/>
・Tip: If a startup script for a newly created GCE instance did not run at all, to help investigate this problem, we can<br/>a)Press the SSH button on the instance Details page in the console. <br/>& b) Check the machine's syslog. <br/>
Note: GCE console's monitoring tab has information that it can get from the hypervisor so we can see things like CPU usage <br/>but it cannot see error messages logged by the startup script.

<img src="./images/milestone1.png" width="500"/>

The real world is not about following instructions that someone else gives you, <br/>The real world is about figuring out what needs to be done and doing it.<br/>

<h2 align="center">5. Scaling / Auto-Scaling</h2>

<pre>
We can use INSTANCE GROUPS to manage the scaling of our infrastructure.
➟ Un-Managed Instance Group type <br/> requires us to manually monitor the VM load, then CREATE New VMs manually and add them to the group accordingly. <br/>
・We have to switch to "[x]Single zone" to be able to have an unmanaged instance group.<br/>
・We have to manually stop the instances when they are not being used. <br/>
・When deleting Un-Managed Instance Group, instances are not automatically deleted and need to be further deleted manually.<br/>

➟ Managed instance Groups are created by using INSTANCE TEMPLATES only.<br/>
・We have to switch to "[x]Multiple zones" to be able to have a managed instance group.<br/>
・It automatically scales and stops the instances as per the requirements and the set specifications. <br/>
・Deleting a managed instance group will delete all of its instances because it owns them.<br/>
・In normal circumstances, we want the cooldown period to be set longer than the total amount of<br/> time it takes to start up the instance and finish running its startup script.<br/>
・Health Check: This is about ensuring that the instances are healthy enough to do the work they need to do. <br/>  If you imagine that you're running a program to pick up work packages, but something went wrong with it, and it crashed...<br/>  That won't necessarily shut down the machine, but if you have a health check that is continually asking: Are you still alright? What about now? Everything Good? <br/>  Then, if it doesn't get a positive response, it can say all right, there's something wrong with you, You need to be refreshed. <br/>  It'll delete the problematic machine and create a new one from template.
   
<img src="./images/milestone2.png" width="500"/>

😃 Autoscaling is much better than manually managing instances <br/>😃 Everything depends on reliable automation<br/>😌 Load balancing is separate from auto-scaling. (See Section 7)
</pre>

<h2 align="center">6. Security - (Need to Re-Study to improve notes)</h2>

<pre>
Security means Ensuring proper data flow.
Not too much; not too little.
<br/>
What is "proper" data flow? (CIA Triad)
→ You cannot view data you shouldn't 
(making sure that only the right people can see certain data is Confidentiality)
→ You cannot change data you shouldn't 
(Being able to trust that the data hasn't been changed inappropriately is Integrity)
→ You can access data you should 
(Being able to get at the data and use the system when you do need to is Availability)
<br/>
How do we control data flow? (AAA)
・ Authentication: Who are you?
・ Authorization: What are you allowed to do?
・ Accounting: What did you?
<br/>
Key Security Products/Features:
➟ AuthN (authentication)
・ Identity - Humans in G Suite, Cloud Identity
            - Applications & services use Service Accounts
・ Identity hierarchy: Google Groups
・ Can use Google Cloud Directory Sync (GCDS) to pull from LDAP (no push)
<br/>
➟ AuthZ (Authorization)
・ Identity hierarchy (Google Groups)
・ Resource hierarchy (Organization, Folders, Projects)
・ Identity and Access Management (IAM): Permissions, Roles, Bindings
・ GCS ACLs
・ Billing management
・ Networking structure & restrictions
<br/>
➟ AccT (Accounting)
・ Audit Activity Logs (provided by Stackdriver)
・ Billing export - To BigQuery
                  - To file (in GCS bucket): Can be JSON or CSV
・ GCS Object Lifecycle Management
------------------------------------------------<br/>
⮕ IAM Resource Hierarchy [AuthZ]
Organization (Tied to G Suite or Cloud Identity domain)
 ↆ 
Folder (Contains any number of Projects and Subfolders)
 ↆ
Project (Container for a set of related resources)
 ↆ
Resource (Something you create in GCP)

⮕ Permissions: allows you to perform a certain action<br/>In the IAM world,
-- Each permission follows the form ➟ Service.Resource.Verb
-- for example: pubsub.subscriptions.consume.

⮕ Role: is a collection of Permissions to use or manage GCP resources.<br/>
-- Primitive Roles: Project-level and often too broad
 ・ Viewer: read-only
 ・ Editor: view and change things
 ・ Owner: can also control access & billing
-- Predefined Roles Give granular access to specific GCP resources
 ・ E.g.: roles/bigquery.dataEditor, roles/pubsub.subscriber

⮕ Roles Overview: https://cloud.google.com/iam/docs/roles-overview <br/>⮕ Complete List of Roles & Permissions: https://cloud.google.com/iam/docs/understanding-roles#basic

⮕Member: A Member is some Google-known identity identified by a unique email address

-- user: Specific Google account (can be G Suite, Cloud Identity, Gmail, or any validated email)
-- serviceAccount: used by apps/services (not by humans)
-- group: is a named collection of Google accounts and service accounts. (giving developer access) 
-- domain: for when you want to grant access to some resource to every single account in a particular domain.  That domain needs to be managed by G Suite or Cloud Identity
-- allAuthenticatedUsers: Resource is Public but needs some Google account or service account. (The only difference is that you can tie actions back to some user account which is what makes this different)
-- allUsers: Anyone on the Internet, no authentication needed, completely anonymous access is fine. (Public) <br/>
- Group should be the default choice always
- Use them for everything!
- You can even use a group to be the owner of projects when you have an organization.
- And to be able to manage the groups effectively in an organization, you can nest them. <br/>So for example you might want to have one group for each department and then have another group called all staff. This is good because every employee will be in some department so they'll always be included in the all staff group. And it's a lot less work to manage adding and removing employees this way than going and finding each and every one of the groups that they should be a part of.

⮕ Policies
・Each resource can have exactly one policy attached to it.
・By default Resources generally have an empty policy attached to them.
・Now in that one policy per resource you can have a maximum of 1500 member bindings. It should never get to that number. Always use group.
・Policies are Always additive ("Allow only") and never subtractive (no "Deny").<br/>
<img src="./images/policies-example.png" width="700"/> Source: https://cloud.google.com/iam/docs/overview

Note: Manual work Not Preferred = Can use get-iam-policy, edit the JSON/YAML, and set-iam-policy back.
[Prefer "add-iam-policy-binding" & "remove-iam-policy-binding"]
ADDING A POLICY ⬇
⮕ gcloud [GROUP] add-iam-policy-binding [RESOURCE-NAME]
 --role [ROLE-ID-TO-GRANT] --member user: [USER-EMAIL]
REMOVING A POLICY ⬇
⮕ gcloud [GROUP] remove-iam-policy-binding [RESOURCE-NAME]
 --role [ROLE-ID-TO-REVOKE] --member user: [USER-EMAIL]

➟ example 1: 
 gcloud compute instances add-iam-policy-binding my-instance
 --role roles/compute.instanceAdmin
 --member user: me@example.com <br/>
➟ example 2: 
 gcloud compute instances add-iam-policy-binding my-instance --zone=ZONE --member='user:test-user@gmail.com' --role='roles/compute.securityAdmin'

Best Practices: Using IAM Securely= https://cloud.google.com/iam/docs/using-iam-securely

BILLING IAM ROLES ⬇
・Billing Account Creator = Create new self-serve billing accounts (Scope= User at Org level)
・Billing Account Administrator = Manage billing accounts (but not create them). (Scope= User at Billing Account level)
・Billing Account User = Link projects to billing accounts. (Scope= User at Billing Account level)
・Billing Account Viewer = View billing account cost information and transactions. (This role could let you audit what's going on but not change anything.)
・Project Billing Manager = Link/Unlink projects to a billing account. (Scope=Project level)

</pre>

<h2 align="center">7. Networking = Routing</h2>

<pre>
Don't think in terms of connections and networks.
Instead, think about how each piece of data makes its journey from place to place.
・Routing is about deciding where data should go next.
 ⮕ READ: 7 layers of OSI https://www.webopedia.com/definitions/7-layers-of-osi-model/
  ⮕ READ: Hot vs Cold Potato Routing https://en.wikipedia.org/wiki/Hot-potato_and_cold-potato_routing<br/>
<img src="./images/google uses cold networking & not hot.png" width="700"/>
 ⮕ READ: Networking tears https://cloud.google.com/blog/products/networking/introducing-network-service-tiers-your-cloud-network-your-way <br/>
Let's take a look at how data makes its way from the front door of Google's network to the correct resource in our system.

Reasons to direct data flow to different resources:
-- Latency reduction = Use servers physically close to clients. (You want your system to respond quickly to users. So it's good if your servers are physically close to them.)
 ➟ Use Cross-Region Load Balancing (with Global Anycast IPs)<br/>
-- Load balancing = Let's say you have too many incoming requests to be handled by just one machine. So you add a second machine to help out, but there's no point in doing that if all of the incoming requests still go to the overloaded machine, right? That's why we need load balancing to divide the requests amongst the machines that can handle them. 
 ➟ Use Cloud Load Balancer (all types; internal and external)
 ・ (So Load balancing is Separate from auto-scaling to handle fluctuating workloads. Yes, they work well together, but it would be best to keep a mental model that has them related, not entirely overlapped.)
 ⮕ READ: https://cloud.google.com/load-balancing/docs/load-balancing-overview<br/>
-- System design = You might have different servers in your system handling different parts of that system. And this is especially true when your service is composed of micro services rather than all jammed together (in a monolith).
 ➟ Use HTTP(S) Load Balancer (with URL Map)
 ・ (If you imagine something like an online store, you might have some systems that handle the item catalog, maybe other systems that know about the current inventory levels, and yet different systems that handle user data like past orders that they ve placed or maybe even what's in their cart right now.)
 ⮕ READ: Routing schemes https://en.wikipedia.org/wiki/Routing
-- 